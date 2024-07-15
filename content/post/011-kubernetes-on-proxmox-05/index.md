---
title: kubernetesをproxmox上に立ててみた（5）/HA構成
date: 2024-07-15
slug: kubernetes-on-proxmox-05
categories:
    - Kubernetes
    - Proxmox
---

## 開発環境
- Proxmox 8.2.4
- Ubuntu Server 24.04 LTS
- Kubernetes v1.30.2

## HAProxyをセットアップする
```
sudo apt update
sudo apt upgrade
```
### IPアドレスを固定する
- 99-config.yaml
```
network:
    version: 2
    renderer: networkd
    ethernets:
      ens18:
        dhcp4: false
        addresses:
          - 192.168.10.45/24
        routes:
          - to: default
            via: 192.168.10.1
        nameservers:
            search: []
            addresses: [192.168.10.1]
```

ファイルを反映する
```
sudo cp 99-config.yaml /etc/netplan/
sudo netplan apply
```
```
sudo chmod 600 /etc/netplan/99-config.yaml
```

### HAProxyをインストールする
```
sudo apt install haproxy
```
```
sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.default
sudo nano /etc/haproxy/haproxy.cfg
```

下記の通りに編集する
```
defaults
    timeout connect    10s
    timeout client     30s
    timeout server     30s

frontend k8s
    bind *:6443
    mode               tcp
    #option tcplog
    default_backend    k8s_backend

backend k8s_backend
    balance            roundrobin
    server             k8s-control-plane-01 192.168.10.41:6443 check
    server             k8s-control-plane-02 192.168.10.44:6443 check
    #server             k8s-control-plane-01 <control node2のip>:6443 check
```

反映する
```
sudo systemctl enable --now haproxy
```

## HA構成のクラスタを構築する
### Control-Plane-01で下記コマンドを実行する
```
sudo kubeadm init --control-plane-endpoint=192.168.10.45:6443 --pod-network-cidr=10.128.0.0/16 --upload-certs
```
- "--control-plane-endpoint=<IPアドレス>:6443":Control-PlaneのIPアドレスとAPIサーバーのポートを指定する
- "--pod-network-cidr=10.128.0.0/16":Pod間ネットワークの指定する
- "--upload-certs":Control-Plane、Worker-Node間で共有する証明書を暗号化する

実行結果
```
mao@k8s-control-plane-01:~$ sudo kubeadm init --control-plane-endpoint=192.168.10.45:6443 --pod-network-cidr=10.128.0.0/16 --upload-certs
[init] Using Kubernetes version: v1.30.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-control-plane-01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.10.41 192.168.10.45]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-control-plane-01 localhost] and IPs [192.168.10.41 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-control-plane-01 localhost] and IPs [192.168.10.41 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.677267ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 5.012535655s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
46c4727ea4da0877fd6e152f0c5d4842837ea949909e9ccb83a5c2f7331f53a9
[mark-control-plane] Marking the node k8s-control-plane-01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-control-plane-01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: h3eh17.f1j6k27nzi34hd0w
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w \
        --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7 \
        --control-plane --certificate-key 46c4727ea4da0877fd6e152f0c5d4842837ea949909e9ccb83a5c2f7331f53a9

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w \
        --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7 
mao@k8s-control-plane-01:~$ 
```

### kubeadmコマンドを実行できるようにする
上記の実行結果に記載のある通り下記のコマンドを実行する
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### クラスとにJoinするために必要な情報をメモ（記録）しておく
上記の実行結果に記載のある通りクラスタにJoinする際に必要なコマンドをメモしておく\
Control-PlaneがクラスタにJoinするために必要な情報
```
kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w \
        --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7 \
        --control-plane --certificate-key 46c4727ea4da0877fd6e152f0c5d4842837ea949909e9ccb83a5c2f7331f53a9
```
Worker-NodeがクラスタにJoinするために必要な情報
```
kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w \
        --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7 
```

## 他のControl-PlaneをクラスタにJoinする
下記コマンドを実行する
```
sudo kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7 --control-plane --certificate-key 46c4727ea4da0877fd6e152f0c5d4842837ea949909e9ccb83a5c2f7331f53a9
```

## Woker-NodeをクラスタにJoinする
Worker-NodeをクラスタにJoinさせる、1つ目以外のWoker-Nodeも同じようにクラスタに参加させる
```
sudo kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7
```

実行結果
```
mao@k8s-worker-01:~$ sudo kubeadm join 192.168.10.45:6443 --token h3eh17.f1j6k27nzi34hd0w --discovery-token-ca-cert-hash sha256:6ee64840cf218d5f9ee05a5138e0f543bf6b2359f51ab3d13fde7405370ba7a7
[sudo] password for mao: 
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 500.998945ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

mao@k8s-worker-01:~$ 
```

## HA構成の確認のためにControl-Planeを1つダウンさせてみる
Control-Planeを1つシャットダウンさせて擬似的にダウンさせてみる
```
mao@k8s-control-plane-03:~$ kubectl get nodes
NAME                   STATUS     ROLES           AGE    VERSION
k8s-control-plane-01   Ready      control-plane   146m   v1.30.2
k8s-control-plane-02   NotReady   control-plane   140m   v1.30.2
k8s-control-plane-03   Ready      control-plane   10m    v1.30.2
k8s-worker-01          Ready      <none>          136m   v1.30.2
k8s-worker-02          Ready      <none>          127m   v1.30.2
mao@k8s-control-plane-03:~$ 
```
確認用のコマンドが問題なく実行され、Control-Planeが1つ"NotReady"になっているが、問題なくクラスタが稼働している

## 参考
### IPアドレスの構成
- 192.168.10.45,HAProxy
- 192.168.10.41,Control-Plane-01
- 192.168.10.44,Control-Plane-02
- 192.168.10.46,Control-Plane-03
- 192.168.10.42,Worker-Node-01
- 192.168.10.43,Worker-Node-02

### 参考URL
- https://kubernetes.io/ja/docs/setup/#production-environment
- https://kubernetes.io/ja/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/
- https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## 感想
以上でHA構成のkubernetesの構築は完了しました\
公式ドキュメントを参考にしつつ進めてわからないところは検索したりして構築できました\
ネットワーク部分に関しては構築前よりも詳しくなったと思います\
発展として、他にも"keepalive"や"kube-vip"を使用した構成もあるみたいなので、いずれ試してみようと思います\
あとkubernetes上に仮想マシンを構成する"kubevirt"や"openstack on kubernetes"を構築してみようと思います
