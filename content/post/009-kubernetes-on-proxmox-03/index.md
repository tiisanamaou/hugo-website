---
title: kubernetesをproxmox上に立ててみた（3）/LoadBalancerの設定
date: 2024-07-07
lastmod: 2024-07-13
slug: kubernetes-on-proxmox-03
categories:
    - Kubernetes
    - Proxmox
---

## 開発環境
- Proxmox 8.2.4
- Ubuntu Server 24.04 LTS
- Kubernetes v1.30.2

## LoadBalancer（MetalLB）の設定をする
- kubernetesのクラスターの外部からIPアドレスでアクセスするための設定をする
- ロードバランサー（MetalLB）を使用して外部からアクセスできるようにする
- マニフェストファイルの"type"に"LoadBalancer"を指定できるようになる

### MetalLB を実行する
今回はMetalLBを使用する
- https://metallb.universe.tf/installation/

### ARPの設定をする
- 1
```
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system
```
- 2
```
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```
- 実行結果
```
mao@k8s-control-plane-01:~$ kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system
diff -u -N /tmp/LIVE-1007559117/v1.ConfigMap.kube-system.kube-proxy /tmp/MERGED-3311346621/v1.ConfigMap.kube-system.kube-proxy
--- /tmp/LIVE-1007559117/v1.ConfigMap.kube-system.kube-proxy    2024-06-26 12:33:11.717730136 +0000
+++ /tmp/MERGED-3311346621/v1.ConfigMap.kube-system.kube-proxy  2024-06-26 12:33:11.718730159 +0000
@@ -37,7 +37,7 @@
       excludeCIDRs: null
       minSyncPeriod: 0s
       scheduler: ""
-      strictARP: false
+      strictARP: true
       syncPeriod: 0s
       tcpFinTimeout: 0s
       tcpTimeout: 0s
mao@k8s-control-plane-01:~$ kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
Warning: resource configmaps/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/kube-proxy configured
mao@k8s-control-plane-01:~$ 
```

### MetalLBの実行をする
参考URL
- https://blog.ntnx.jp/entry/2024/02/14/025309

```
wget https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
kubectl apply -f metallb-native.yaml
```

確認をする
```
kubectl get pod -n metallb-system
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl get pod -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-86f5578878-dz5zm   1/1     Running   0          43s
speaker-mbvpk                 1/1     Running   0          43s
speaker-rx75f                 1/1     Running   0          43s
speaker-xwb66                 1/1     Running   0          43s
mao@k8s-control-plane-01:~$ 
```

払い出せるIPアドレスの範囲を設定する
- 下記の部分で範囲を設定する
```
spec:
  addresses:
  - 192.168.10.55-192.168.10.60
```

- 1
```
cat << EOF > ippool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.55-192.168.10.60
EOF
```
- 2
```
cat << EOF > l2adv.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
EOF
```

設定を実行する
```
kubectl apply -f ippool.yaml -f l2adv.yaml
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl apply -f ippool.yaml -f l2adv.yaml
ipaddresspool.metallb.io/default created
l2advertisement.metallb.io/default created
```

## マニフェストファイルを実行後、外部IPアドレスが設定されているか確認する
マニフェストファイル
- nginx-lb.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-deployment-lb
spec:
  type: LoadBalancer
  #type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```
- "type: LoadBalancer"にするとLoadBalancerからIPアドレスが払い出されてクラスタ外からアクセスできるようになる

マニフェストファイルを実行してデプロイする
```
kubectl apply -f nginx-lb.yaml
```


確認するためのコマンド
```
kubectl get svc
or
kubectl get service
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl get svc
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes            ClusterIP      10.96.0.1       <none>          443/TCP        25h
nginx-deployment-lb   LoadBalancer   10.98.208.100   192.168.10.45   80:32762/TCP   58s
mao@k8s-control-plane-01:~$ 
```

"nginx-deployment-lb"の"TYPE"に"LoadBalancer"が指定されており"EXTERNAL-IP"（外部IPアドレス）が割り当てられている

このIPアドレスにアクセスすると実際のコンテナにアクセスできる
