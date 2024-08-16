---
title: Kubernetesのクラスタをv1.30.3からv1.31.0へアップグレードをする
date: 2024-08-16
slug: kubernetes-upgrade-v1-31-0
categories:
    - Kubernetes
---

## 環境
- Kubernetes 1.30.3 → 1.30.4 → 1.31.0
- Control-Plane 3台
- Woker-Node 2台

## 参考URL
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## 変更点の確認
- https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.30.md#downloads-for-v1304
- https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.31.md

問題なさそうなのでアップグレードします

## 現状の構成
v1.30.3を使用している
```
mao@k8s-control-plane-01:~$ kubectl get nodes -o wide
NAME                   STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-control-plane-01   Ready    control-plane   46d   v1.30.3   192.168.10.41   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-control-plane-02   Ready    control-plane   46d   v1.30.3   192.168.10.44   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-control-plane-03   Ready    control-plane   46d   v1.30.3   192.168.10.46   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-worker-01          Ready    <none>          46d   v1.30.3   192.168.10.42   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-worker-02          Ready    <none>          46d   v1.30.3   192.168.10.43   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
mao@k8s-control-plane-01:~$ 
```
## Control-Planeのアップグレード
### Control-Plane-01
アップグレードの際のコマンドを記載していきます
```
kubectl drain --ignore-daemonsets k8s-control-plane-01
kubectl get nodes
sudo apt update
sudo apt-cache madison kubeadm
```

#### 一旦v1.30.4にする
```
mao@k8s-control-plane-01:~$ sudo apt-cache madison kubeadm
   kubeadm | 1.30.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
mao@k8s-control-plane-01:~$ 
```
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.4-*' kubectl='1.30.4-*' && \
sudo apt-mark hold kubelet kubectl

kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.30.4
kubectl get nodes
```

#### このままv1.31.0にアップグレードする
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt-cache madison kubeadm
```
```
mao@k8s-control-plane-01:~$ sudo apt-cache madison kubeadm
   kubeadm | 1.31.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
mao@k8s-control-plane-01:~$ 
```
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-*' kubectl='1.31.0-*' && \
sudo apt-mark hold kubelet kubectl

kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.31.0
kubectl get nodes
kubectl uncordon k8s-control-plane-01
kubectl get nodes
```

### Control-Plane-02
#### 一旦v1.30.4にする
```
kubectl drain --ignore-daemonsets k8s-control-plane-02
kubectl get nodes
sudo apt update
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.4-*' kubectl='1.30.4-*' && \
sudo apt-mark hold kubelet kubectl

sudo kubeadm upgrade node
kubectl get nodes
```

#### このままv1.31.0にアップグレードする
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-*' kubectl='1.31.0-*' && \
sudo apt-mark hold kubelet kubectl

sudo kubeadm upgrade node
kubectl get nodes
kubectl uncordon k8s-control-plane-02
kubectl get nodes
```

### Control-Plane-03
#### 一旦v1.30.4にする
```
kubectl drain --ignore-daemonsets k8s-control-plane-03
kubectl get nodes
sudo apt update
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.4-*' kubectl='1.30.4-*' && \
sudo apt-mark hold kubelet kubectl

sudo kubeadm upgrade node
kubectl get nodes
```

#### このままv1.31.0にアップグレードする
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-*' && \
sudo apt-mark hold kubeadm

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-*' kubectl='1.31.0-*' && \
sudo apt-mark hold kubelet kubectl

sudo kubeadm upgrade node
kubectl get nodes
kubectl uncordon k8s-control-plane-03
kubectl get nodes
```

## Woker-Node
### Woker-Node-01
#### 一旦v1.30.4にする
- Control-Planeでの作業
```
kubectl drain k8s-worker-01 --ignore-daemonsets
kubectl get nodes
```

- Woker-Nodeでの作業
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-*' && \
sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.4-*' kubectl='1.30.4-*' && \
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

- Control-Planeでの作業
```
kubectl get nodes
```

#### このままv1.31.0にアップグレードする
- Woker-Nodeでの作業
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-*' && \
sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-*' kubectl='1.31.0-*' && \
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

- Control-Planeでの作業
```
kubectl get nodes
kubectl uncordon k8s-worker-01
kubectl get nodes
```

### Woker-Node-02
#### 一旦v1.30.4にする
- Control-Planeでの作業
```
kubectl drain k8s-worker-02 --ignore-daemonsets
kubectl get nodes
```

- Woker-Nodeでの作業
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-*' && \
sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.4-*' kubectl='1.30.4-*' && \
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

- Control-Planeでの作業
```
kubectl get nodes
```

#### このままv1.31.0にアップグレードする
- Woker-Nodeでの作業
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.0-*' && \
sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.0-*' kubectl='1.31.0-*' && \
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

- Control-Planeでの作業
```
kubectl get nodes
kubectl uncordon k8s-worker-02
kubectl get nodes
```

## 最終確認
下記のコマンドで問題ないか確認する
```
kubectl get nodes -o wide
kubectl get pod -A -o wide
```
```
mao@k8s-control-plane-01:~$ kubectl get nodes -o wide
NAME                   STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-control-plane-01   Ready    control-plane   46d   v1.31.0   192.168.10.41   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-control-plane-02   Ready    control-plane   46d   v1.31.0   192.168.10.44   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-control-plane-03   Ready    control-plane   46d   v1.31.0   192.168.10.46   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-worker-01          Ready    <none>          46d   v1.31.0   192.168.10.42   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
k8s-worker-02          Ready    <none>          46d   v1.31.0   192.168.10.43   <none>        Ubuntu 24.04 LTS   6.8.0-40-generic   containerd://1.7.18
mao@k8s-control-plane-01:~$ kubectl get pod -A -o wide
NAMESPACE          NAME                                           READY   STATUS    RESTARTS         AGE     IP               NODE                   NOMINATED NODE   READINESS GATES
calico-apiserver   calico-apiserver-5f78767767-gfsdj              1/1     Running   0                2m44s   10.128.36.197    k8s-worker-01          <none>           <none>
calico-apiserver   calico-apiserver-5f78767767-stslw              1/1     Running   0                9m44s   10.128.251.151   k8s-control-plane-01   <none>           <none>
calico-system      calico-kube-controllers-5f5665469b-8rzw4       1/1     Running   0                2m44s   10.128.36.198    k8s-worker-01          <none>           <none>
calico-system      calico-node-26sbk                              1/1     Running   7 (26m ago)      46d     192.168.10.44    k8s-control-plane-02   <none>           <none>
calico-system      calico-node-28xrv                              1/1     Running   7 (55s ago)      46d     192.168.10.43    k8s-worker-02          <none>           <none>
calico-system      calico-node-dc87d                              1/1     Running   7 (47m ago)      46d     192.168.10.41    k8s-control-plane-01   <none>           <none>
calico-system      calico-node-g2ks8                              1/1     Running   8 (18m ago)      46d     192.168.10.46    k8s-control-plane-03   <none>           <none>
calico-system      calico-node-l2ll2                              1/1     Running   7 (4m46s ago)    46d     192.168.10.42    k8s-worker-01          <none>           <none>
calico-system      calico-typha-5579b889c8-9r6v9                  1/1     Running   0                4m30s   192.168.10.42    k8s-worker-01          <none>           <none>
calico-system      calico-typha-5579b889c8-gd8sc                  1/1     Running   0                17m     192.168.10.46    k8s-control-plane-03   <none>           <none>
calico-system      calico-typha-5579b889c8-n8nmp                  1/1     Running   0                40s     192.168.10.43    k8s-worker-02          <none>           <none>
calico-system      csi-node-driver-2pzn6                          2/2     Running   14 (4m46s ago)   46d     10.128.36.196    k8s-worker-01          <none>           <none>
calico-system      csi-node-driver-8b6ts                          2/2     Running   14 (47m ago)     46d     10.128.251.150   k8s-control-plane-01   <none>           <none>
calico-system      csi-node-driver-9w86p                          2/2     Running   14 (18m ago)     46d     10.128.204.146   k8s-control-plane-03   <none>           <none>
calico-system      csi-node-driver-cljz8                          2/2     Running   14 (26m ago)     46d     10.128.194.204   k8s-control-plane-02   <none>           <none>
calico-system      csi-node-driver-jddlc                          2/2     Running   14 (55s ago)     46d     10.128.118.120   k8s-worker-02          <none>           <none>
kube-system        coredns-7db6d8ff4d-7gv56                       1/1     Running   0                9m44s   10.128.194.205   k8s-control-plane-02   <none>           <none>
kube-system        coredns-7db6d8ff4d-znvj7                       1/1     Running   0                2m44s   10.128.36.202    k8s-worker-01          <none>           <none>
kube-system        etcd-k8s-control-plane-01                      1/1     Running   0                46m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        etcd-k8s-control-plane-02                      1/1     Running   0                25m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        etcd-k8s-control-plane-03                      1/1     Running   0                17m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-01            1/1     Running   0                46m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-02            1/1     Running   1 (26m ago)      26m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-03            1/1     Running   1 (18m ago)      18m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-01   1/1     Running   0                45m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-02   1/1     Running   1 (26m ago)      26m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-03   1/1     Running   1 (18m ago)      18m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-proxy-2h2wm                               1/1     Running   1 (56s ago)      19m     192.168.10.43    k8s-worker-02          <none>           <none>
kube-system        kube-proxy-746mc                               1/1     Running   0                19m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-proxy-k2h9l                               1/1     Running   1 (18m ago)      19m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-proxy-lvgb7                               1/1     Running   1 (4m47s ago)    19m     192.168.10.42    k8s-worker-01          <none>           <none>
kube-system        kube-proxy-xtm2m                               1/1     Running   0                19m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-01            1/1     Running   0                45m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-02            1/1     Running   1 (26m ago)      26m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-03            1/1     Running   1 (18m ago)      18m     192.168.10.46    k8s-control-plane-03   <none>           <none>
metallb-system     controller-86f5578878-4whk5                    1/1     Running   0                2m44s   10.128.36.200    k8s-worker-01          <none>           <none>
metallb-system     speaker-2hccg                                  1/1     Running   13 (4m46s ago)   46d     192.168.10.42    k8s-worker-01          <none>           <none>
metallb-system     speaker-cjz7j                                  1/1     Running   13 (26m ago)     46d     192.168.10.44    k8s-control-plane-02   <none>           <none>
metallb-system     speaker-dpfpl                                  1/1     Running   13 (55s ago)     46d     192.168.10.43    k8s-worker-02          <none>           <none>
metallb-system     speaker-gg452                                  1/1     Running   13 (18m ago)     46d     192.168.10.46    k8s-control-plane-03   <none>           <none>
metallb-system     speaker-zj5x9                                  1/1     Running   13 (47m ago)     46d     192.168.10.41    k8s-control-plane-01   <none>           <none>
tigera-operator    tigera-operator-76ff79f7fd-8dgv4               1/1     Running   0                2m44s   192.168.10.42    k8s-worker-01          <none>           <none>
mao@k8s-control-plane-01:~$ 
```

全てNodeもpodも問題なく稼働しているのを確認したので、無事アップグレード完了
