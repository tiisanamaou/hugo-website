---
title: Traefikを使ってHarborをHTTPSで通信させる
date: 2026-03-08
slug: traefik-harbor-ssl
categories:
    - Traefik
    - Cloudflare
    - Harbor
---

## 環境
- Ubuntu 24.04.1 LTS
- Docker 29.1.3
- Traefik 3.6.5
- Harbor v2.11.1

上記構成の理由はSSL証明書を自分で管理するのではなく自動更新かつワイルドカードでの取得にしたかったため

## IPの固定
IPアドレスを固定します

```
ip a
nano 99-config.yaml
```

- 99-config.yaml
```
network:
    version: 2
    renderer: networkd
    ethernets:
      ens18:
        dhcp4: false
        addresses:
          - 192.168.10.70/24
        routes:
          - to: default
            via: 192.168.10.1
        nameservers:
            search: []
            addresses: [192.168.10.1]
```

設定を適用する
```
sudo cp 99-config.yaml /etc/netplan/
sudo netplan apply
sudo chmod 600 /etc/netplan/99-config.yaml
```

## 設定ファイルの書き換え1（Harbor）
Harborの設定ファイルを書き換える
```
nano harbor.yml
```

下記の設定を変更する
- "hostname","harbor.internal.example.dev"
- "http","port:8088"
  - ポートがTraefikの受付ポート等とかぶらないようにするため
- "http","relativeurls: true"
  - デフォルトではない項目なので項目を追加する
- "external_url: https://harbor.internal.example.dev"
  - "http"ではなく"http**s**"のURLを記載する

下記コマンドを実行し設定ファイル作成（再作成）する
```
sudo ./prepare
```

実行結果
```
mao@harbor-server:~/harbor$ sudo ./prepare
prepare base dir is set to /home/mao/harbor
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Clearing the configuration file: /config/portal/nginx.conf
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/jobservice/config.yml
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/log/rsyslog_docker.conf
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/registry/root.crt
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/registry/passwd
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /data/secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
mao@harbor-server:~/harbor$
```

## 設定ファイルの書き換え2（docker-compose）
上記で作成されたcomposeファイルを編集する
```
nano docker-compose.yml
```
下記項目について変更する
- nginx
- labels
- networks

### Traefikの設定例
Traefikでhttpsにするには下記のように"labels"と"networks"を設定する

- 例：compose.nginx.yaml
```
services:
  nginx:
    image: nginx:1.29.4
    container_name: nginx-sample
    restart: unless-stopped
    networks:
      - traefik-public
    labels:
      # Traefik で公開する
      - "traefik.enable=true"
      # ルーター設定（実際のドメインに変更してください）
      - "traefik.http.routers.nginx.rule=Host(`nginx.example.dev`)"
      - "traefik.http.routers.nginx.entrypoints=websecure"
      - "traefik.http.routers.nginx.tls.certresolver=letsencrypt"
      # ワイルドカード証明書の設定
      - "traefik.http.routers.nginx.tls.domains[0].main=example.dev"
      - "traefik.http.routers.nginx.tls.domains[0].sans=*.example.dev"
      # サービス設定
      - "traefik.http.services.nginx.loadbalancer.server.port=80"

networks:
  traefik-public:
    name: traefik-public
    external: true
```

### composeファイルを書き換える
- docker-compose.yml
- 上記ファイルの下の方にnginx(proxy)の設定があります
- 下記は抜粋です
```
  proxy:
    image: goharbor/nginx-photon:v2.11.1
    container_name: nginx
    restart: always
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - NET_BIND_SERVICE
    volumes:
      - ./common/config/nginx:/etc/nginx:z
      - /data/secret/cert:/etc/cert:z
      - type: bind
        source: ./common/config/shared/trust-certificates
        target: /harbor_cust_cert
    networks:
      - harbor
+    # Traefik用追加設定
+     - traefik-public
-    #ports:
-    #  - 8088:8080
+    # 追加（外部には公開しない）
+    expose:
+      - "8080"
    depends_on:
      - registry
      - core
      - portal
      - log
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://localhost:1514"
        tag: "proxy"
+   # Traefik用追加設定
+   labels:
+     # Traefik で公開する
+     - "traefik.enable=true"
+     # ルーター設定（実際のドメインに変更してください）
+     - "traefik.http.routers.harbor.rule=Host(`harbor.internal.example.dev`)"
+     - "traefik.http.routers.harbor.entrypoints=websecure"
+     - "traefik.http.routers.harbor.tls.certresolver=letsencrypt"
+     # ワイルドカード証明書の設定
+     - "traefik.http.routers.harbor.tls.domains[0].main=internal.example.dev"
+     - "traefik.http.routers.harbor.tls.domains[0].sans=*internal.example.dev"
+     # サービス設定
+     - "traefik.http.services.harbor.loadbalancer.server.port=8080"
networks:
  harbor:
    external: false
+ traefik-public:
+   name: traefik-public
+   external: true
```

### 起動
順番としてはTraefikが起動してからHarborを起動する

下記コマンドで起動する
```
sudo docker compose -up -d
```

しばらくすると起動するのでダッシュボードにアクセスできるようになる\
"gateway timeout"が表示される場合は一度落としてから再度起動する

これでhttpsでアクセス可能かつTraefikでSSL証明書を管理できるようになりました

## 参考URL
- https://github.com/goharbor/harbor/issues/12135
- https://medium.com/@joelkoussawo/installing-harbor-container-registry-behind-traefik-reverse-proxy-with-lets-encrypt-certificate-8ab1733daa1
- https://tech-mmmm.blogspot.com/2022/12/ossharbor.html
