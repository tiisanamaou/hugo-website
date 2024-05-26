---
title: Go言語をソースコードからビルドする
date: 2024-05-25
slug: golang-source-build
categories:
    - Go
---

# 環境
- Ubuntu 23.10
- Intel Core i5 13500
- メモリ64GB

# 1：setup
ソースからビルドするのに必要なソフトを確認する
```
make --version
gcc --version
```
インストールされていない場合は以下のコマンドでインストールする
```
sudo apt install make
sudo apt install gcc
```

ビルドするのに必要なバージョン
>The minimum version of Go required depends on the target version of Go:
>- Go <= 1.4: a C toolchain.
>- 1.5 <= Go <= 1.19: a Go 1.4 compiler.
>- 1.20 <= Go <= 1.21: a Go 1.17 compiler.
>- 1.22 <= Go <= 1.23: a Go 1.20 compiler.
>- Going forward, Go version 1.N will require a Go 1.M compiler, where M is N-2 rounded down to an even number. Example: Go 1.24 and 1.25 require Go 1.22.

# 2：build go1.4
go1.4-bootstrapをビルドする
```
wget https://dl.google.com/go/go1.4-bootstrap-20171003.tar.gz
mkdir go1.4-bootstrap && tar xzvf go1.4-bootstrap-20171003.tar.gz -C go1.4-bootstrap --strip-components 1
cd ./go1.4-bootstrap/src
```
```
CGO_ENABLED=0 bash ./make.bash
```
```
Installed Go for linux/amd64 in /home/mao/Desktop/go1.4-bootstrap
Installed commands in /home/mao/Desktop/go1.4-bootstrap/bin
```

# 3：build go1.17
go1.17をビルドする
```
wget https://dl.google.com/go/go1.17.src.tar.gz
mkdir go1.17 && tar xzvf go1.17.src.tar.gz -C go1.17 --strip-components 1
cd ./go1.17/src
```
```
GOROOT_BOOTSTRAP=${PWD}/go1.4-bootstrap bash ./all.bash
GOROOT_BOOTSTRAP=/home/mao/Desktop/go1.4-bootstrap bash ./all.bash
```
```
Go version is "go1.17", ignoring -next /home/mao/Desktop/go1.17/api/next.txt

ALL TESTS PASSED
---
Installed Go for linux/amd64 in /home/mao/Desktop/go1.17
Installed commands in /home/mao/Desktop/go1.17/bin
*** You need to add /home/mao/Desktop/go1.17/bin to your PATH.

```

# 4：build go1.20
go1.20をビルドする
```
wget https://dl.google.com/go/go1.20.src.tar.gz
mkdir go1.20 && tar xzvf go1.20.src.tar.gz -C go1.20 --strip-components 1
cd ./go1.20/src
```
```
/home/mao/Desktop/go1.17/bin
GOROOT_BOOTSTRAP=/home/mao/Desktop/go1.17 bash ./all.bash
```
```
ALL TESTS PASSED
---
Installed Go for linux/amd64 in /home/mao/Desktop/go1.20
Installed commands in /home/mao/Desktop/go1.20/bin
*** You need to add /home/mao/Desktop/go1.20/bin to your PATH.

```

# 5：build go1.22.2 latest
go1.22.2をビルドする
```
wget https://dl.google.com/go/go1.22.2.src.tar.gz
mkdir go1.22.2 && tar xzvf go1.22.2.src.tar.gz -C go1.22.2 --strip-components 1
cd ./go1.22.2/src
```
```
/home/mao/Desktop/go1.20/bin
GOROOT_BOOTSTRAP=/home/mao/Desktop/go1.20 bash ./all.bash
```
```
ALL TESTS PASSED
---
Installed Go for linux/amd64 in /home/mao/Desktop/go1.22.2
Installed commands in /home/mao/Desktop/go1.22.2/bin
*** You need to add /home/mao/Desktop/go1.22.2/bin to your PATH.
```

# 6：Pathを通す
パスを通してバージョンを確認する
```
sudo cp -rp ./go1.22.2 /usr/local/
```

.bashrc
```
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:/usr/local/go1.22.2/bin
```
```
source ~/.bashrc
```
```
go version
```
```
go version go1.22.2 linux/amd64
```

# 参考URL
- https://go.dev/doc/install/source
- https://qiita.com/myoshimi/items/5d1f6a2ee8a849bac7eb
- https://qiita.com/soarflat/items/d5015bec37f8a8254380

