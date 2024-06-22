---
title: ProxmoxがインストールされているDeskminix300のメモリを増設する
date: 2024-06-22
slug: deskminix300-memory-upgrade
categories:
    - proxmox
---

## 環境
- Proxmox VE 8.2.4 x86_64 

## アップグレード前
- メモリ16GB
```
root@pve:~# neofetch
         .://:`              `://:.            root@pve 
       `hMMMMMMd/          /dMMMMMMh`          -------- 
        `sMMMMMMMd:      :mMMMMMMMs`           OS: Proxmox VE 8.2.4 x86_64 
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`   Kernel: 6.8.4-2-pve 
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`   Uptime: 33 days, 20 hours, 14 mins 
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`     Packages: 852 (dpkg) 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.       Shell: bash 5.2.15 
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.         Terminal: /dev/pts/0 
        -+ooooooo/.`sMMs`./ooooooo+-           CPU: AMD Ryzen 7 5700G with Radeon Graphi 
          :oooooooo/`..`/oooooooo:             GPU: AMD ATI Radeon Vega Series / Radeon  
          :oooooooo/`..`/oooooooo:             Memory: 1858MiB / 13837MiB 
        -+ooooooo/.`sMMs`./ooooooo+-
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.                                 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.                               
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`
        `sMMMMMMMm:      :dMMMMMMMs`
       `hMMMMMMd/          /dMMMMMMh`
         `://:`              `://:`

root@pve:~#
```

## アップグレード後
- メモリ64GB
```
root@pve:~# neofetch
         .://:`              `://:.            root@pve 
       `hMMMMMMd/          /dMMMMMMh`          -------- 
        `sMMMMMMMd:      :mMMMMMMMs`           OS: Proxmox VE 8.2.4 x86_64 
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`   Kernel: 6.8.8-1-pve 
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`   Uptime: 1 min 
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`     Packages: 852 (dpkg) 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.       Shell: bash 5.2.15 
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.         Terminal: /dev/pts/0 
        -+ooooooo/.`sMMs`./ooooooo+-           CPU: AMD Ryzen 7 5700G with Radeon Graphics (16) @ 4.673GHz 
          :oooooooo/`..`/oooooooo:             GPU: AMD ATI Radeon Vega Series / Radeon Vega Mobile Series 
          :oooooooo/`..`/oooooooo:             Memory: 1508MiB / 60133MiB 
        -+ooooooo/.`sMMs`./ooooooo+-
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.                                 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.                               
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`
        `sMMMMMMMm:      :dMMMMMMMs`
       `hMMMMMMd/          /dMMMMMMh`
         `://:`              `://:`

root@pve:~#
```
