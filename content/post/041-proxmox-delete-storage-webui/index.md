---
title: ProxmoxのWebUI上で削除したストレージがはてなマークのまま残ってしまった場合の対処方法
date: 2025-03-25
slug: proxmox-delete-storage-webui
categories:
    - Proxmox
---

## 対処方法
Proxmoxの設定ファイルから、残ってしまったストレージを削除する
```
nano /etc/pve/storage.cfg
```

下記のようなになっているので必要のない部分を削除する
```
dir: local
        path /var/lib/vz
        content backup,iso,vztmpl

lvmthin: local-lvm
        thinpool data
        vgname pve
        content images,rootdir

lvmthin: add-data
        thinpool add-data
        vgname add-data
        content rootdir,images
        nodes pve

lvm: hdd-storage
        vgname hdd-storage
        content rootdir,images
        nodes pve
        shared 0

dir: vm-backup
        path /mnt/pve/vm-backup
        content snippets,iso,backup,images,vztmpl,rootdir
        is_mountpoint 1
        nodes pve
```

## 参考URL
- https://pve.proxmox.com/wiki/Storage
- https://zaki-lknr.github.io/Misc/proxmox/
