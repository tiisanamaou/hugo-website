---
title: 仮想マシン上のDiskの拡張手順
date: 2024-11-09
lastmod: 2025-02-24
slug: ubuntu-server-disk-expansion
categories:
    - Ubuntu
---

## 背景
Proxmox上に作成した仮想マシンの容量を10GBで作成したが、足りなくなり、20GBへ容量アップした際の手順です

## 環境
- Ubuntu Server 24.04 LTS
- Parted 3.6
- ディスク容量：10GB→20GB

## 参考URL
- VMware上のUbuntu 22.04 ファイルシステム拡張
    - https://qiita.com/mcyang/items/a32b914db073f308a3cb

## 仮想マシンのDiskの拡張手順
- 下記の順番でコマンドを実行していけば、割り当てられたストレージの容量いっぱいに拡張される

### パーティションを拡張する
```
sudo apt install lvm2
```
```
df -h
sudo pvdisplay
sudo apt install parted
sudo parted /dev/sda
```

```
print free
```
```
(parted) print free                                                       
Warning: Not all of the space available to /dev/sda appears to be used, you can fix the GPT to use all of the
space (an extra 20971520 blocks) or continue with the current setting? 
Fix/Ignore? Fix
```
```
resizepart 3 100% 
print free
q
```

- ※UbuntuDesktopの場合は上記コマンド実行後、下記コマンドを実行するだけで容量が増える
```
sudo resize2fs /dev/sda2
```

### 物理ボリュームを拡張する
```
sudo pvdisplay
sudo pvresize /dev/sda3
sudo pvdisplay
```

### ロジカルボリュームとファイルシステムを拡張する
```
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
df -h
```

## 実際の実行結果
- 10GBから20GBへ増やす
```
mao@mao-cri-o-worker-node:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              795M  1.5M  793M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  8.1G  5.7G  2.0G  75% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G   94M  1.5G   6% /boot
tmpfs                              795M   12K  795M   1% /run/user/1000
mao@mao-cri-o-worker-node:~$ sudo pvdisplay
[sudo] password for mao: 
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <8.25 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2111
  Free PE               0
  Allocated PE          2111
  PV UUID               CGNZQy-1SZf-8wIh-yeOq-BIF7-f2VJ-9cWRR2
   
mao@mao-cri-o-worker-node:~$ sudo apt install parted
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  dmidecode libparted2t64
Suggested packages:
  libparted-dev libparted-i18n parted-doc
The following NEW packages will be installed:
  dmidecode libparted2t64 parted
0 upgraded, 3 newly installed, 0 to remove and 44 not upgraded.
Need to get 267 kB of archives.
After this operation, 758 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 dmidecode amd64 3.5-3build1 [72.1 kB]
Get:2 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 libparted2t64 amd64 3.6-4build1 [152 kB]
Get:3 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 parted amd64 3.6-4build1 [43.3 kB]
Fetched 267 kB in 2s (131 kB/s) 
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package dmidecode.
(Reading database ... 72892 files and directories currently installed.)
Preparing to unpack .../dmidecode_3.5-3build1_amd64.deb ...
Unpacking dmidecode (3.5-3build1) ...
Selecting previously unselected package libparted2t64:amd64.
Preparing to unpack .../libparted2t64_3.6-4build1_amd64.deb ...
Adding 'diversion of /lib/x86_64-linux-gnu/libparted.so.2 to /lib/x86_64-linux-gnu/libparted.so.2.usr-is-merged by libparted2t64'
Adding 'diversion of /lib/x86_64-linux-gnu/libparted.so.2.0.5 to /lib/x86_64-linux-gnu/libparted.so.2.0.5.usr-is-merged by libparted2t64'
Unpacking libparted2t64:amd64 (3.6-4build1) ...
Selecting previously unselected package parted.
Preparing to unpack .../parted_3.6-4build1_amd64.deb ...
Unpacking parted (3.6-4build1) ...
Setting up dmidecode (3.5-3build1) ...
Setting up libparted2t64:amd64 (3.6-4build1) ...
Removing 'diversion of /lib/x86_64-linux-gnu/libparted.so.2 to /lib/x86_64-linux-gnu/libparted.so.2.usr-is-merged by libparted2t64'
Removing 'diversion of /lib/x86_64-linux-gnu/libparted.so.2.0.5 to /lib/x86_64-linux-gnu/libparted.so.2.0.5.usr-is-merged by libparted2t64'
Setting up parted (3.6-4build1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.3) ...
Scanning processes...                                                                                          
Scanning linux images...                                                                                       

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
mao@mao-cri-o-worker-node:~$ sudo parted /dev/sda
GNU Parted 3.6
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print free                                                       
Warning: Not all of the space available to /dev/sda appears to be used, you can fix the GPT to use all of the
space (an extra 20971520 blocks) or continue with the current setting? 
Fix/Ignore? Fix                                                           
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  1881MB  1879MB  ext4
 3      1881MB  10.7GB  8855MB
        10.7GB  21.5GB  10.7GB  Free Space

(parted) resizepart 3 100%
(parted) print free
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  1881MB  1879MB  ext4
 3      1881MB  21.5GB  19.6GB

(parted) q                                                                
Information: You may need to update /etc/fstab.

mao@mao-cri-o-worker-node:~$ sudo pvdisplay                               
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <8.25 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2111
  Free PE               0
  Allocated PE          2111
  PV UUID               CGNZQy-1SZf-8wIh-yeOq-BIF7-f2VJ-9cWRR2
   
mao@mao-cri-o-worker-node:~$ sudo pvresize /dev/sda3
  Physical volume "/dev/sda3" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
mao@mao-cri-o-worker-node:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <18.25 GiB / not usable 16.50 KiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              4671
  Free PE               2560
  Allocated PE          2111
  PV UUID               CGNZQy-1SZf-8wIh-yeOq-BIF7-f2VJ-9cWRR2
   
mao@mao-cri-o-worker-node:~$ sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <8.25 GiB (2111 extents) to <18.25 GiB (4671 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
mao@mao-cri-o-worker-node:~$ sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 4783104 (4k) blocks long.

mao@mao-cri-o-worker-node:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              795M  3.4M  791M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   18G  5.7G   12G  34% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G   94M  1.5G   6% /boot
tmpfs                              795M   12K  795M   1% /run/user/1000
mao@mao-cri-o-worker-node:~$ 
```
