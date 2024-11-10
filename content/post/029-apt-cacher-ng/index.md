---
title: aptのキャッシュをしてくれるapt-cacher-ngのインストール方法
date: 2024-11-10
slug: apt-cacher-ng
categories:
    - apt-cacher-ng
---

## 背景
複数のPCからaptを使用する際に毎回ダウンロードすると時間がかかるので、ローカルにキャッシュサーバーを立ててそこからダウンロードできるようにしたい\
キャッシュサーバーではなくミラーサーバーでも良かったが、ミラーサーバーだと容量が必要になるので、今回はキャッシュサーバーを使用することにした

## 環境
- Ubuntu Server 24.04 LTS
- Apt-Cacher-NG/3.7.4  

## apt-cacher-ngのインストールと設定ファイルの作成
```
sudo apt-get update; sudo apt-get install apt-cacher-ng ; echo 'Acquire::http::Proxy "http://127.0.0.1:3142";' | sudo tee /etc/apt/apt.conf.d/02proxy ;
```

作成した設定ファイルを編集する
```
sudo nano /etc/apt/apt.conf.d/02proxy
```
下記の内容を追記する
```
Acquire::http::Proxy "http://192.168.10.***:3142";
```
- "Acquire::http::Proxy "http://192.168.10.16:3142";"

httpsのものはパススルーをするように設定する
```
sudo nano /etc/apt-cacher-ng/acng.conf ;
```
```
PassThroughPattern: .*
```

設定を読み込むためにサービスを再起動する
```
sudo service apt-cacher-ng restart
```

## WebUIにアクセスする
"http://192.168.10.16:3142"にアクセスするとWebUIが表示される、どれくらいキャッシュされているか等が見れる

## 他のPCからaptのキャッシュを参照するように設定する
proxyの設定ファイルを編集する
```
sudo nano /etc/apt/apt.conf.d/02proxy
```

apt-cacher-ngをインストールしたサーバーのIPアドレスを指定する
```
Acquire::http::Proxy "http://192.168.10.***:3142";
```
- "Acquire::http::Proxy "http://192.168.10.16:3142";"

## キャッシュしたデータとログの保存場所
キャッシュの保存場所
```
/var/cache/apt-cacher-ng/
```

ログの保存場所
```
/var/log/apt-cacher-ng/apt-cacher.log
```

## キャッシュの削除
ストレージの容量不足でエラーになってしまったため一度キャッシュを削除する
```
ファイルへの書き込みでエラーが発生しました - write (28: デバイスに空き領域がありません) [IP: 192.168.10.16 3142]
```

キャッシュの削除手順
- 容量の確認
```
df -h
```

サービスを一旦止める
```
sudo systemctl stop apt-cacher-ng
```

キャッシュデータの削除
```
sudo rm -rf /var/lib/apt/* \
sudo rm -rf /var/cache/apt/* \
sudo rm -rf /var/cache/apt-cacher-ng/
```
```
sudo mkdir -p /var/cache/apt-cacher-ng/{headers,import,packages,private,temp}
sudo chown apt-cacher-ng:apt-cacher-ng -R /var/cache/apt-cacher-ng
```

サービスの再開
```
sudo systemctl start apt-cacher-ng
sudo systemctl status apt-cacher-ng
```

容量に空きができているか確認
```
df -h
```

### 参考URL
- https://qiita.com/mugimugi/items/edb743c6c32444159384
- http://bluearth.cocolog-nifty.com/blog/2020/04/post-182a10.html
- http://bluearth.cocolog-nifty.com/blog/2020/04/post-8db9b1.html
- https://qiita.com/mt08/items/c8b8187b1000382d77ae
- https://zenn.dev/toru3/scraps/07d08e9f86ae71
- https://blog.cybozu.io/entry/2016/07/19/103000

## 直接インストールではなくdockerを使うやり方もあるよう
dockerの方が楽そう
- apt-cacher-ng で apt をキャッシュする
    - https://zenn.dev/st_little/articles/cache-apt-with-apt-cacher-ng
- apt-cacher-ng サービスの Docker 化
    - https://docs.docker.jp/engine/examples/apt-cacher-ng.html
