---
title: KubernetesでTraefikを使用してIngressを使えるようにする
date: 2024-08-21
slug: kubernetes-ingress-traefik
categories:
    - Kubernetes
    - Traefik
    - Helm
---

## 環境
- Kubernetes 1.31.0
- Helm v3.15.3
- Traefik v3.1.2

## HelmでTraefikのリポジトリを追加する
リポジトリを追加してupdateする
```
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm repo list
```
```
mao@k8s-control-plane-01:~$ helm repo add traefik https://traefik.github.io/charts
"traefik" has been added to your repositories
mao@k8s-control-plane-01:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kubernetes-dashboard" chart repository
...Successfully got an update from the "traefik" chart repository
Update Complete. ⎈Happy Helming!⎈
mao@k8s-control-plane-01:~$ helm repo list
NAME                    URL                                    
kubernetes-dashboard    https://kubernetes.github.io/dashboard/
traefik                 https://traefik.github.io/charts       
mao@k8s-control-plane-01:~$
```

## Ingress-Controllerをインストールする
Ingress-Controllerを"ns-traefik"という"namespace"にインストールする\
作成されているか確認する
```
helm install traefik traefik/traefik --create-namespace --namespace ns-traefik
kubectl get services -A
```
```
mao@k8s-control-plane-01:~$ helm install traefik traefik/traefik --create-namespace --namespace ns-traefik
NAME: traefik
LAST DEPLOYED: Tue Aug 20 11:51:28 2024
NAMESPACE: ns-traefik
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
traefik with docker.io/traefik:v3.1.2 has been deployed successfully on ns-traefik namespace !
mao@k8s-control-plane-01:~$ 
mao@k8s-control-plane-01:~$ kubectl get services -A
NAMESPACE          NAME                              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
calico-apiserver   calico-api                        ClusterIP      10.100.15.33    <none>          443/TCP                      51d
calico-system      calico-kube-controllers-metrics   ClusterIP      None            <none>          9094/TCP                     51d
calico-system      calico-typha                      ClusterIP      10.111.65.20    <none>          5473/TCP                     51d
default            kubernetes                        ClusterIP      10.96.0.1       <none>          443/TCP                      51d
kube-system        kube-dns                          ClusterIP      10.96.0.10      <none>          53/UDP,53/TCP,9153/TCP       51d
metallb-system     metallb-webhook-service           ClusterIP      10.98.148.198   <none>          443/TCP                      51d
ns-traefik         traefik                           LoadBalancer   10.98.137.158   192.168.10.55   80:31371/TCP,443:31450/TCP   2m32s
mao@k8s-control-plane-01:~$ 
```

podが作成されていることを確認する
```
kubectl get --namespace ns-traefik pod
```
```
mao@k8s-control-plane-01:~$ kubectl get --namespace ns-traefik pod
NAME                       READY   STATUS    RESTARTS   AGE
traefik-6996c86bfd-kb284   1/1     Running   0          4m45s
mao@k8s-control-plane-01:~$ 
```

## Ingress,Serviceをデプロイする
マニフェストファイルを作成する
- traefik-ingress.yaml
- ingressとnginxのservice,podを一緒にデプロイする
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-traefik-test
  # ingressのnamespaceとserviceのnamespaceは同じにする
  #namespace: ns-traefik

spec:
  rules:
    # hostを設定しないとIPアドレスでアクセスできる
    #- host: ingress1.internal
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-deployment-lb
                port:
                  number: 83
    - host: ingress1.internal
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-deployment-lb
                port:
                  number: 83
    - host: ingress2.internal
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-deployment-lb
                port:
                  number: 83
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  #namespace: ns-traefik
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
  #namespace: ns-traefik
spec:
  type: NodePort
  ports:
  - port: 83
    targetPort: 80
  selector:
    app: nginx
```

デプロイする
```
kubectl apply -f traefik-ingress.yaml
kubectl get ingress -A
```

## アクセスできるか確認する
IPアドレスとバーチャルホストでアクセスできるか確認する\
結果が帰ってくればOK\
アクセスできない場合はportがあっているかnamespaceが同じか等確認をする
```
curl http://192.168.10.55
curl -H 'Host:ingress1.internal' http://192.168.10.55
curl -H 'Host:ingress2.internal' http://192.168.10.55
```

## 削除する
```
kubectl delete -f traefik-ingress.yaml
helm uninstall traefik -n ns-traefik
```
```
kubectl get services -A
helm ls -A
```

## 参考URL
- インストール（Helmを使用する）
    - https://doc.traefik.io/traefik/getting-started/install-traefik/
- Ingressのマニフェストファイル
    - https://doc.traefik.io/traefik/providers/kubernetes-ingress/
    - https://doc.traefik.io/traefik/routing/providers/kubernetes-ingress/
- Kubernetes で Traefik proxy を使う
    - https://zenn.dev/zenogawa/articles/k8s_traefik_ingress
- 自宅の kubernetes に ingress-nginx を入れてみる
    - https://konchangakita.hatenablog.com/entry/2020/07/13/220000
- https://y-ohgi.com/introduction-kubernetes/3_objects/ingress/
- https://qiita.com/dingtianhongjie/items/73980a3e9fbc8c7bc3cd
