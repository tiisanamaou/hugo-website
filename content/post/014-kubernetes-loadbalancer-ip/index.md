---
title: KubernetesのLoadBalancerでIPアドレスを指定する方法
date: 2024-08-11
slug: kubernetes-loadbalancer-ip
categories:
    - MetalLB
    - Kubernetes
---

## 環境
- Kubernetes 1.30.2
- containerd 1.7.18
- Calico 3.28.0
- MetalLB 0.14.5
  - IPアドレスのプール:192.168.10.55-192.168.10.58

## 背景
MetalLBを使用してtype:loadBalancerが使用できるようになったが、IPアドレスの指定方法がわからなかったので、調べて作業してみました。\
"このサービスにはこのIPアドレスを使用する"のように自分でIPアドレスを指定したい。

## 元のマニフェストファイル
- nginx-test.yaml
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
  annotations:
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx

```

## 現状
普通にデプロイするとIPプールの中から空いているIPアドレスが付与される

- 実行結果
```
mao@k8s-control-plane-01:~$ kubectl apply -f nginx-test.yaml
deployment.apps/nginx-deployment created
service/nginx-deployment-lb created
mao@k8s-control-plane-01:~$ kubectl get services
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes            ClusterIP      10.96.0.1       <none>          443/TCP        38d
nginx-deployment-lb   LoadBalancer   10.96.182.105   192.168.10.55   80:30974/TCP   3s
mao@k8s-control-plane-01:~$ 
```

- 削除します
```
mao@k8s-control-plane-01:~$ kubectl delete -f nginx-test.yaml
deployment.apps "nginx-deployment" deleted
service "nginx-deployment-lb" deleted
```

## LoadBalancerでIPアドレスを指定する
MetalLBを使用していると2つの方法があるようなので両方試してみます\
MetalLBのページでは"metadata"を使用した方法が推奨されている（spec.LoadBalancerIPはk8s apisで非推奨となる予定だからのようです）

### spec.loadBalancerIPに指定する
"spec"に"loadBalancerIP:192.168.10.57"を追加してデプロイしてみます
- マニフェストファイル
```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-deployment-lb
spec:
  type: LoadBalancer
+  loadBalancerIP: 192.168.10.57
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```
- 実行結果
```
mao@k8s-control-plane-01:~$ kubectl apply -f nginx-test.yaml
deployment.apps/nginx-deployment created
service/nginx-deployment-lb created
mao@k8s-control-plane-01:~$ kubectl get services
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
kubernetes            ClusterIP      10.96.0.1        <none>          443/TCP        38d
nginx-deployment-lb   LoadBalancer   10.111.232.189   192.168.10.57   80:31150/TCP   4s
mao@k8s-control-plane-01:~$ 
```
しっかりと指定したIPアドレスがが付与されています

- 削除します
```
mao@k8s-control-plane-01:~$ kubectl delete -f nginx-test.yaml
deployment.apps "nginx-deployment" deleted
service "nginx-deployment-lb" deleted
```

### metadata.annotationsに指定する
マニフェストファイルを修正します
- "metadata"に"annotations:"と"metallb.universe.tf/loadBalancerIPs: 192.168.10.56"を追加する
- "loadBalancerIP: 192.168.10.57"を削除する
```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-deployment-lb
+  annotations:
+    metallb.universe.tf/loadBalancerIPs: 192.168.10.56
spec:
  type: LoadBalancer
-  loadBalancerIP: 192.168.10.57
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```
- 実行結果
```
mao@k8s-control-plane-01:~$ kubectl apply -f nginx-test.yaml
deployment.apps/nginx-deployment created
service/nginx-deployment-lb created
mao@k8s-control-plane-01:~$ kubectl get services
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes            ClusterIP      10.96.0.1       <none>          443/TCP        38d
nginx-deployment-lb   LoadBalancer   10.110.178.62   192.168.10.56   80:32442/TCP   3s
mao@k8s-control-plane-01:~$ 
```
マニフェストファイルで指定したIPアドレスが付与されている

終わったら削除する
```
mao@k8s-control-plane-01:~$ kubectl delete -f nginx-test.yaml
deployment.apps "nginx-deployment" deleted
service "nginx-deployment-lb" deleted
mao@k8s-control-plane-01:~$
```

## 参考URL
- https://metallb.universe.tf/usage/
- https://cstoku.dev/posts/2018/k8sdojo-09/
- https://qiita.com/suzuyui/items/8f53a80edf2b32d45be2
- https://blog.framinal.life/entry/2020/04/16/022042
