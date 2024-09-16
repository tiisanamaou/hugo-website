---
title: KubernetesのCNIにCiliumを使用して構築する
date: 2024-09-16
slug: kubernetes-cni-cilium
categories:
    - Kubernetes
    - Cilium
---

## 環境
- Kubernetes 1.31.0
  - Control-Plane：1台
  - Woker-Node：3台+1台
- cri-o v1.30.5
- Helm v3.15.4
- Cilium v1.16.1

## Kubernetesの準備
### cri-oインストール
```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/deb/Release.key |
    sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
```
```
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/deb/ /" |
    sudo tee /etc/apt/sources.list.d/cri-o.list
```
```
sudo apt update &&
sudo apt install cri-o &&
sudo systemctl daemon-reload &&
systemctl enable --now crio
```
```
crio version
```
```
mao@cilium-woker-node-01:~$ crio version
INFO[2024-09-04 11:27:18.691313588Z] Starting CRI-O, version: 1.30.5, git: df27b8f8eb49a13c522aca56ee4ec27bc7482fad(clean) 
Version:        1.30.5
GitCommit:      df27b8f8eb49a13c522aca56ee4ec27bc7482fad
GitCommitDate:  2024-09-02T07:15:35Z
GitTreeState:   clean
BuildDate:      1970-01-01T00:00:00Z
GoVersion:      go1.22.0
Compiler:       gc
Platform:       linux/amd64
Linkmode:       static
BuildTags:      
  static
  netgo
  osusergo
  exclude_graphdriver_btrfs
  exclude_graphdriver_devicemapper
  seccomp
  apparmor
  selinux
LDFlags:          unknown
SeccompEnabled:   true
AppArmorEnabled:  false

mao@cilium-woker-node-01:~$
```

### スワップをOFFにする
```
sudo swapoff -a
sudo nano /etc/fstab
free -h
```

### カーネルパラメータの設定をする
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
```
sudo modprobe overlay &&
sudo modprobe br_netfilter
```

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
```
sudo sysctl --system
```

### crioの設定をする
```
sudo crio config default | sudo tee /etc/crio/crio.conf
sudo nano /etc/crio/crio.conf
```

```
[crio.runtime]
conmon_cgroup = "pod"
cgroup_manager = "cgroupfs"

default_runtime = "runc"
```
```
[crio.image]
pause_image = "registry.k8s.io/pause:3.9"
```

```
sudo systemctl restart cri-o
```

### runCのインストール
```
sudo wget https://github.com/opencontainers/runc/releases/download/v1.1.14/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

### kubelet,kubeadm,kubectlのインストール
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
sudo apt update &&
sudo apt install kubelet kubeadm kubectl &&
sudo apt-mark hold kubelet kubeadm kubectl
```

### Control-Planeでの作業
```
sudo kubeadm init --apiserver-advertise-address=192.168.10.55 --pod-network-cidr=10.128.0.0/16
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
sudo kubeadm token create --print-join-command
```

### Woker-Nodeでの作業
```
sudo kubeadm join 192.168.10.55:6443 --token 2lbtwj.gnpknhy7yow5jqkg \
        --discovery-token-ca-cert-hash sha256:e5be5c6d9564bed4319dbbd872b105c401e0aed482bc131b6a4759ab5a279bcf
```

### クラスタの確認
```
kubectl get nodes -o wide
```
```
mao@cilium-control-plane-01:~$ kubectl get nodes -o wide
NAME                      STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
cilium-control-plane-01   Ready    control-plane   2m33s   v1.31.0   192.168.10.55   <none>        Ubuntu 24.04.1 LTS   6.8.0-41-generic   cri-o://1.30.5
cilium-woker-node-01      Ready    <none>          68s     v1.31.0   192.168.10.56   <none>        Ubuntu 24.04.1 LTS   6.8.0-41-generic   cri-o://1.30.5
mao@cilium-control-plane-01:~$ 
```

## Helmのインストール
下記のコマンドを実行してHelmをインストールします
- https://github.com/helm/helm/releases
```
wget https://get.helm.sh/helm-v3.15.4-linux-amd64.tar.gz
tar -zxvf helm-v3.15.4-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
helm version
```

インストール完了
```
mao@cilium-control-plane-01:~$ helm version
version.BuildInfo{Version:"v3.15.4", GitCommit:"fa9efb07d9d8debbb4306d72af76a383895aa8c4", GitTreeState:"clean", GoVersion:"go1.22.6"}
```

アンインストール方法
```
helm uninstall release_name -n release_namespace
helm uninstall kubernetes-dashboard -n kubernetes-dashboard
```

## Ciliumのデプロイ
### リポジトリの追加します
```
helm repo add cilium https://helm.cilium.io/
```

追加されたか確認
```
helm repo list
```
```
mao@cilium-control-plane-01:~$ helm repo list
NAME    URL                    
cilium  https://helm.cilium.io/
```

### Ciliumをインストールします
下記のファイルをダウンロードしてCIDRを書き換えます
- 1784行目くらいにある
- デフォルト"10.0.0.0/8"
- 変更後"10.128.0.0/16"
```
wget https://raw.githubusercontent.com/cilium/cilium/v1.16.1/install/kubernetes/cilium/values.yaml
```

- "vakues.yaml"を指定してインストールする
```
helm install cilium cilium/cilium --version 1.16.1 --namespace kube-system -f values.yaml
```
```
mao@cilium-control-plane-01:~$ helm install cilium cilium/cilium --version 1.16.1 --namespace kube-system -f values.yaml
NAME: cilium
LAST DEPLOYED: Wed Sep  4 11:53:51 2024
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
You have successfully installed Cilium with Hubble.

Your release version is 1.16.1.

For any further help, visit https://docs.cilium.io/en/v1.16/gettinghelp
mao@cilium-control-plane-01:~$ 
```

デプロイされているか確認をする
```
kubectl get pod -A -o wide
```
```
mao@cilium-control-plane-01:~$ kubectl -n kube-system get pods -o wide
NAME                                              READY   STATUS    RESTARTS   AGE     IP              NODE                      NOMINATED NODE   READINESS GATES
cilium-6swqr                                      1/1     Running   0          80s     192.168.10.56   cilium-woker-node-01      <none>           <none>
cilium-envoy-5rlzw                                1/1     Running   0          79s     192.168.10.56   cilium-woker-node-01      <none>           <none>
cilium-envoy-clh2j                                1/1     Running   0          80s     192.168.10.55   cilium-control-plane-01   <none>           <none>
cilium-operator-5c7867ccd5-ngjj8                  1/1     Running   0          79s     192.168.10.55   cilium-control-plane-01   <none>           <none>
cilium-operator-5c7867ccd5-qrsnp                  1/1     Running   0          79s     192.168.10.56   cilium-woker-node-01      <none>           <none>
cilium-z44kx                                      1/1     Running   0          79s     192.168.10.55   cilium-control-plane-01   <none>           <none>
coredns-6f6b679f8f-66qm2                          1/1     Running   0          64s     10.0.0.159      cilium-control-plane-01   <none>           <none>
coredns-6f6b679f8f-wkpjz                          1/1     Running   0          49s     10.0.0.177      cilium-control-plane-01   <none>           <none>
etcd-cilium-control-plane-01                      1/1     Running   0          5m17s   192.168.10.55   cilium-control-plane-01   <none>           <none>
kube-apiserver-cilium-control-plane-01            1/1     Running   0          5m17s   192.168.10.55   cilium-control-plane-01   <none>           <none>
kube-controller-manager-cilium-control-plane-01   1/1     Running   0          5m17s   192.168.10.55   cilium-control-plane-01   <none>           <none>
kube-proxy-86htv                                  1/1     Running   0          5m12s   192.168.10.55   cilium-control-plane-01   <none>           <none>
kube-proxy-msrd4                                  1/1     Running   0          3m55s   192.168.10.56   cilium-woker-node-01      <none>           <none>
kube-scheduler-cilium-control-plane-01            1/1     Running   0          5m17s   192.168.10.55   cilium-control-plane-01   <none>           <none>
mao@cilium-control-plane-01:~$ 
```

### Ciliumのアンインストール方法
```
helm uninstall cilium -n kube-system
```

### クラスタの再構築（手順を間違えた場合）
- Contrpl-Planeで実行
```
kubectl drain cilium-woker-node-01 --ignore-daemonsets --delete-emptydir-data --force
kubectl delete node cilium-woker-node-01
```

- Woker-Nodeで実行
```
sudo kubeadm reset
sudo ip link
sudo ip link delete cilium_vxlan
sudo ip link
```

- Contrpl-Planeで実行
```
sudo kubeadm reset
sudo rm -rf $HOME/.kube
sudo systemctl daemon-reload && systemctl restart kubelet
sudo systemctl restart cri-o
sudo ip link
sudo ip link delete cilium_vxlan
sudo ip link
```

## Hubble-UIにアクセスできるようにする
“vakues.yaml"を編集する
- 1307行目、"hubble.relay.enabled"
```
#enabled: false
enabled: true
```

- 1523行目、"hubble.ui.enabled"
```
#enabled: false
enabled: true
```

- 1683行目、"hubble.ui.service.type"
```
#type: ClusterIP
type: LoadBalancer
```

デプロイする（アップグレードする）
```
helm upgrade cilium cilium/cilium --version 1.16.1 --namespace kube-system -f values.yaml
```
```
mao@cilium-control-plane-01:~$ helm upgrade cilium cilium/cilium --version 1.16.1 --namespace kube-system -f values.yaml
Error: UPGRADE FAILED: execution error at (cilium/templates/validate.yaml:4:7): Hubble UI requires .Values.hubble.relay.enabled=true
mao@cilium-control-plane-01:~$ helm upgrade cilium cilium/cilium --version 1.16.1 --namespace kube-system -f values.yaml
Release "cilium" has been upgraded. Happy Helming!
NAME: cilium
LAST DEPLOYED: Fri Sep  6 12:27:38 2024
NAMESPACE: kube-system
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
You have successfully installed Cilium with Hubble Relay and Hubble UI.

Your release version is 1.16.1.

For any further help, visit https://docs.cilium.io/en/v1.16/gettinghelp
mao@cilium-control-plane-01:~$ 
```

UIのIPアドレスを確認する
```
kubectl -n kube-system get service
```
```
mao@cilium-control-plane-01:~$ kubectl -n kube-system get service
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                  AGE
hubble-peer    ClusterIP      10.100.129.52   <none>          443/TCP                  47h
hubble-relay   ClusterIP      10.101.171.5    <none>          80/TCP                   2m35s
hubble-ui      LoadBalancer   10.97.255.102   192.168.10.60   80:30533/TCP             2m35s
kube-dns       ClusterIP      10.96.0.10      <none>          53/UDP,53/TCP,9153/TCP   47h
mao@cilium-control-plane-01:~$ 
```

下記IPアドレスにアクセスする
- "192.168.10.60"
![](01.png)

## 参考URL
- HelmチャートでKubernetesにCiliumをインストール
    - https://qiita.com/showchan33/items/f336d46af383d4c746d4
- Cilium
    - https://github.com/cilium/cilium/tree/v1.16.1/install/kubernetes/cilium
    - https://docs.cilium.io/en/stable/network/kubernetes/concepts/
- Installation using Helm
    - https://docs.cilium.io/en/stable/installation/k8s-install-helm/
- AWSのEC2インスタンスでKubernetesを作ってみる
    - https://qiita.com/showchan33/items/02e4a5f02b08c08d7813
- kubeadm+containerd+ciliumを用いてk8s構築し、hubbleの動作確認するまで試した
    - https://qiita.com/fruscianteee/items/2b130eaa8418b183d515
- EKS上にCiliumサービスメッシュを稼動させてみた!
    - https://qiita.com/daitak/items/9749c3c6d9c489351ef6
