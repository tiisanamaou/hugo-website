---
title: apt updateをした際にバージョンが上がっている旨のメッセージがでてきた場合の対処
date: 2025-07-07
slug: apt-update-inRelease-version-up
categories:
    - Debian
    - Proxmox
---

## "apt update"をしたらバージョンが上がっている旨のメッセージがでてきたので対処する
Proxmoxで"apt update"をしたらバージョンが上がっている旨のメッセージがでてきたので対処する
```
N: Repository 'http://ftp.jp.debian.org/debian bookworm InRelease' changed its 'Version' value from '12.7' to '12.8'
```
```
root@pve:~# apt update
Get:1 http://security.debian.org bookworm-security InRelease [48.0 kB]
Get:2 http://security.debian.org bookworm-security/main amd64 Packages [190 kB]
Get:3 http://ftp.jp.debian.org/debian bookworm InRelease [151 kB]
Get:4 http://ftp.jp.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:5 http://ftp.jp.debian.org/debian bookworm/main amd64 Packages [8,789 kB]
Get:6 http://ftp.jp.debian.org/debian bookworm/main Translation-en [6,109 kB]
Hit:7 http://download.proxmox.com/debian/pve bookworm InRelease
Fetched 15.3 MB in 2s (9,564 kB/s)             
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Repository 'http://ftp.jp.debian.org/debian bookworm InRelease' changed its 'Version' value from '12.7' to '12.8'
root@pve:~# 
```

## 対応
下記コマンドで対応する
```
apt --allow-releaseinfo-change update
```
```
root@pve:~# apt --allow-releaseinfo-change update
Hit:1 http://security.debian.org bookworm-security InRelease
Hit:2 http://ftp.jp.debian.org/debian bookworm InRelease
Hit:3 http://ftp.jp.debian.org/debian bookworm-updates InRelease
Hit:4 http://download.proxmox.com/debian/pve bookworm InRelease
Reading package lists... Done          
Building dependency tree... Done
Reading state information... Done
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@pve:~# 
```

メッセージは表示されなくなった
```
root@pve:~# apt update
Hit:1 http://security.debian.org bookworm-security InRelease
Hit:2 http://ftp.jp.debian.org/debian bookworm InRelease   
Hit:3 http://ftp.jp.debian.org/debian bookworm-updates InRelease
Hit:4 http://download.proxmox.com/debian/pve bookworm InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@pve:~# 
```
