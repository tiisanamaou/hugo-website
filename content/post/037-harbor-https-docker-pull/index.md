---
title: Harborにhttpsでアクセスしてdocker pullできるようにする
date: 2025-03-14
slug: harbor-https-docker-pull
categories:
    - Harbor
    - Docker
---

## 環境
- Harbor Version v2.11.1-6b7ecba1
- Let's Encrypt
- Unbound 1.19.2
- Docker Engine Community 28.0.1

## 証明書の発行
{{< link-corss-ref "post/030-certbot-let's-encrypt/index.md" >}} を参照

## Harborの設定
証明書ファイルをサーバーにアップロードする

設定ファイルを編集する
- harbor.yml
```
hostname: harbor.yourdomain.dev

http:
  port: 80

https:
  port: 443
  certificate: /home/mao/yourdomain.dev/cert1.pem
  private_key: /home/mao/yourdomain.dev/privkey1.pem
```
- 証明書はフルパスで指定する

下記を実行して設定ファイルを再生成する
```
sudo ./install.sh
```

起動する
```
sudo docker compose up -d
```

## DNS設定
DNS（Unbound）に追加する
キャッシュされるまで時間がかかる？
- 900s?
```
sudo systemctl restart unbound
sudo unbound-control reload
```

DNSに登録されているか確認する
- traceroute
- dig
```
mao@mao (x86_64) : /home/mao 
> traceroute harbor.yourdomain.dev
traceroute to harbor.yourdomain.dev (192.168.10.24), 30 hops max, 60 byte packets
 1  192.168.10.24 (192.168.10.24)  0.403 ms  0.349 ms  0.332 ms
mao@mao (x86_64) : /home/mao 
> dig harbor.yourdomain.dev       

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> harbor.yourdomain.dev
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62907
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;harbor.yourdomain.dev.	IN	A

;; ANSWER SECTION:
harbor.yourdomain.dev. 3123 IN	A	192.168.10.24

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Mar 08 17:00:06 JST 2025
;; MSG SIZE  rcvd: 72
```

## Harborからpullできるか確認する
pullを実行するとエラーになった
```
mao@mao (x86_64) : /home/mao 
> sudo docker pull harbor.yourdomain.dev/homelab/homelab-traefik:3.1.6 
Error response from daemon: Get "https://harbor.yourdomain.dev/v2/": tls: failed to verify certificate: x509: certificate signed by unknown authority
```

### エラーの対処法
証明書（公開鍵）と中間証明書を結合したものが必要なよう
- https://goharbor.io/docs/2.0.0/install-config/troubleshoot-installation/#https

一度止める
```
sudo docker compose down -v
```

設定ファイルを再度編集する
- harbor.yml
```
hostname: harbor.yourdomain.dev

http:
  port: 80

https:
  port: 443
- certificate: /home/mao/yourdomain.dev/cert1.pem
+ certificate: /home/mao/yourdomain.dev/fullchain1.pem
  private_key: /home/mao/yourdomain.dev/privkey1.pem
```
- 証明書と中間証明書が結合された"fullchain1.pem"を使用するように編集する
    - "certificate: /home/mao/yourdomain.dev/cert1.pem"ではなく
    - "certificate: /home/mao/yourdomain.dev/fullchain1.pem"とする

下記を実行して設定ファイルを再生成する
```
sudo ./install.sh
```

起動する
```
sudo docker compose up -d
```

### 改めてpullしてみる
下記コマンドでpullをする
```
sudo docker pull harbor.yourdomain.dev/homelab/homelab-traefik:3.1.6
```

ちゃんとpullされたことが確認できた
```
mao@mao (x86_64) : /home/mao 
> sudo docker pull harbor.yourdomain.dev/homelab/homelab-traefik:3.1.6
3.1.6: Pulling from homelab/homelab-traefik
43c4264eed91: Pull complete 
f60fb4c0fbec: Pull complete 
9a6d31097c4f: Pull complete 
e5f06ee63d76: Pull complete 
Digest: sha256:22aec04848987fe5b3999a4099d766de614b04da52a936fc5ac214ffec04dbac
Status: Downloaded newer image for harbor.yourdomain.dev/homelab/homelab-traefik:3.1.6
harbor.yourdomain.dev/homelab/homelab-traefik:3.1.6
```

これでHorborからhttpsでpullができたので、daemon.jsonファイルを編集しなくても使えるようになった

## 参考URL
- OSSのコンテナレジストリ「Harbor」を自己署名証明書でHTTPS通信させる
    - https://tech-mmmm.blogspot.com/2023/05/ossharborhttps.html
- HTTPS接続のトラブルシューティング
    - https://goharbor.io/docs/2.0.0/install-config/troubleshoot-installation/#https
    - https://github.com/goharbor/harbor/issues/6774
- Let's Encrypt(無料SSL証明書)についてまとめ
    - https://qiita.com/morrr/items/4d96d7a52c7a54e22bf3
