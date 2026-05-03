---
title: Immichをdocker-composeで実行する
date: 2026-05-03
slug: immich-docker-compose
categories:
    - Docker
    - Immich
---

## 環境
- Ubuntu 24.04.1 LTS
- Docker 29.1.3
- Immich v2.7.5

## 手順
基本は公式サイトのドキュメント通りにする

最初にcomposeファイルとenvファイルをダウンロードする
```
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```
```
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

## 設定を追加する
.envに下記を追加する
```
TZ=Asia/Tokyo
```

## 実行する
```
sudo docker compose up -d
```

下記のURLにアクセスする
- "http://IPアドレス:2283"


終了する場合
```
sudo docker compose down -v
```

## 初期設定
![](01.png)
![](02.png)

![](03.png)
![](04.png)

![](05.png)
![](06.png)

![](07.png)
![](08.png)

![](09.png)
![](10.png)

![](11.png)
![](12.png)


## Traefikを使用してSSLにする場合
- https://docs.immich.app/administration/reverse-proxy/#traefik-proxy-example-config


## 参考URL
- https://docs.immich.app/install/docker-compose/
- https://docs.immich.app/install/post-install
- https://docs.immich.app/administration/reverse-proxy/#traefik-proxy-example-config
