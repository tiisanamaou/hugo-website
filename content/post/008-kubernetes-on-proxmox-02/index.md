---
title: kubernetesをproxmox上に立ててみた（2）/Control-Plane・Worker-Nodeの設定
date: 2024-07-06
slug: kubernetes-on-proxmox-02
categories:
    - Kubernetes
    - Proxmox
---

## 開発環境
- Proxmox 8.2.4
- Ubuntu Server 24.04 LTS

## Control-Plane（Master-Node）の設定をする
### kubeadm init を実行する
Control-Planeにするマシン上で実行する
```
sudo kubeadm init --apiserver-advertise-address=Control-PlaneのIPアドレス --pod-network-cidr=10.128.0.0/16
sudo kubeadm init --apiserver-advertise-address=192.168.10.41 --pod-network-cidr=10.128.0.0/16
```
実行結果
```
mao@k8s-control-plane-01:~$ sudo kubeadm init --apiserver-advertise-address=192.168.10.41 --pod-network-cidr=10.128.0.0/16
[init] Using Kubernetes version: v1.30.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-control-plane-01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.10.41]
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
[kubelet-check] The kubelet is healthy after 500.625027ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 3.50086825s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-control-plane-01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-control-plane-01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: evbpii.dp8y5hcfqkv9jn4n
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.41:6443 --token evbpii.dp8y5hcfqkv9jn4n \
        --discovery-token-ca-cert-hash sha256:dd7a24f7fcd7aeea509476025652a8a1aee32e9e8d5f54ec48de16345eb1a425 
mao@k8s-control-plane-01:~$ 
```

実行結果にも表示されている通り、下記のコマンドを実行する
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Calicoを実行する（Pod間ネットワーク）
参考URL
- https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart

tigera-operator.yaml
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
kubectl create -f tigera-operator.yaml
```
custom-resources.yaml
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml
```

ダウンロードしたcustom-resources.yamlを編集する
- kubeadm init で指定した引数--pod-network-cidrと同じものへ変更する
```
- cidr: 192.168.0.0/16
+ cidr: 10.128.0.0/16
```
実行する
```
kubectl apply -f custom-resources.yaml
```

### Nodeを確認する
```
kubectl get nodes -o wide
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl get nodes -o wide
NAME                   STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-control-plane-01   Ready    control-plane   32m   v1.30.2   192.168.10.41   <none>        Ubuntu 24.04 LTS   6.8.0-35-generic   containerd://1.7.18
mao@k8s-control-plane-01:~$ 
```

## Worker-Nodeの設定をする
### Worker-NodeをJoinする
Control-Planeでkubeadm initを実行した際に表示されたコマンドを実行する
```
sudo kubeadm join 192.168.10.41:6443 --token evbpii.dp8y5hcfqkv9jn4n --discovery-token-ca-cert-hash sha256:dd7a24f7fcd7aeea509476025652a8a1aee32e9e8d5f54ec48de16345eb1a425
```
実行結果
```
mao@k8s-worker-01:~$ sudo kubeadm join 192.168.10.41:6443 --token evbpii.dp8y5hcfqkv9jn4n --discovery-to
ken-ca-cert-hash sha256:dd7a24f7fcd7aeea509476025652a8a1aee32e9e8d5f54ec48de16345eb1a425
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.168161ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

mao@k8s-worker-01:~$ 
```

Worker-NodeがJoinされているか、Control-Planeで確認する
```
kubectl get node
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl get node
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   63m   v1.30.2
k8s-worker-01          Ready    <none>          36s   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### 複数のWorker-NodeをJoinする
基本手順は1つ目のWorker-Nodeと同じように実行するとJoinできる
```
sudo kubeadm join 192.168.10.41:6443 --token evbpii.dp8y5hcfqkv9jn4n --discovery-token-ca-cert-hash sha256:dd7a24f7fcd7aeea509476025652a8a1aee32e9e8d5f54ec48de16345eb1a425
```

実行したらControl-PlanでNodeを確認してみる
```
mao@k8s-control-plane-01:~$ kubectl get node
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   9h    v1.30.2
k8s-worker-01          Ready    <none>          8h    v1.30.2
k8s-worker-02          Ready    <none>          83s   v1.30.2
mao@k8s-control-plane-01:~$ 
```

無事にWorker-NodeがJoinされている