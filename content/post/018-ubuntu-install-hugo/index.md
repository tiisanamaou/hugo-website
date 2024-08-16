---
title: UbuntuにHugoをインストールする
date: 2024-08-15
slug: ubuntu-install-hugo
categories:
    - Hugo
    - Ubuntu
---

##
WindowsノートPCを売却するに当たり、データや開発環境をUbuntuへと移行しました\
その中でこのサイトの作成に使用しているHugoのインストール方法をメモしました

## 環境
- Ubuntu 24.04 LTS
- Intel Core i5-13500 (amd64)

## 参考URL
- https://gohugo.io/installation/linux/#prebuilt-binaries
- https://github.com/gohugoio/hugo/releases

## ダウンロード
バイナリをダウンロードします
```
wget https://github.com/gohugoio/hugo/releases/download/v0.132.1/hugo_extended_0.132.1_linux-amd64.tar.gz
```

## 展開・解凍
フォルダを作成して展開する
```
mkdir hugo_extended_0.132.1_linux-amd64
tar -zxvf hugo_extended_0.132.1_linux-amd64.tar.gz -C ./hugo_extended_0.132.1_linux-amd64
```

## パスの通っている場所へ移動し、バージョン確認
バイナリファイルを"/usr/local/bin"へ移動します\
その後バージョンの確認をします
```
sudo mv hugo_extended_0.132.1_linux-amd64/hugo /usr/local/bin/hugo
hugo version
```
```
mao@mao:~$ hugo version
hugo v0.132.1-1bde700dfc0770bb11eb8445aff1ab5abdccb46e+extended linux/amd64 BuildDate=2024-08-13T10:10:10Z VendorInfo=gohugoio
mao@mao:~$ 
```

## アンインストール
アンインストールする際は下記のコマンドを実行し、バイナリの場所を確認して削除します
```
mao@mao:~$ which hugo
/usr/local/bin/hugo
mao@mao:~$ sudo rm /usr/local/bin/hugo
```
