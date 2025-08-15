---
title: Proxmoxを8から9へバージョンアップする
date: 2025-08-15
slug: proxmox-upgrade-8to9
categories:
    - Proxmox
---

## 参考URL
基本は公式の手順に従ってアップグレードする

- Upgrade from 8 to 9
    - https://pve.proxmox.com/wiki/Upgrade_from_8_to_9
- Proxmox VE 8 を9にアップグレードしたときのメモ
    - https://zenn.dev/omohikane/articles/upgrade-proxmoxve8to9
- PVE 9.0アップデート手順のリポジトリ設定がわからなかった話（aptの.listと.source）
    - https://qiita.com/oishi-d/items/2618fbb3a87520737b94

## 現在の状態
"neofetch"で確認する
```
root@pve:~# neofetch
         .://:`              `://:.            root@pve 
       `hMMMMMMd/          /dMMMMMMh`          -------- 
        `sMMMMMMMd:      :mMMMMMMMs`           OS: Proxmox VE 8.4.11 x86_64 
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`   Kernel: 6.8.12-8-pve 
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`   Uptime: 159 days, 4 hours, 14 mins 
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`     Packages: 873 (dpkg) 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.       Shell: bash 5.2.15 
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.         Terminal: /dev/pts/0 
        -+ooooooo/.`sMMs`./ooooooo+-           CPU: AMD Ryzen 7 5700G with Radeon Graphics (16) @ 4.673GHz 
          :oooooooo/`..`/oooooooo:             GPU: AMD ATI Radeon Vega Series / Radeon Vega Mobile Series 
          :oooooooo/`..`/oooooooo:             Memory: 2082MiB / 60133MiB 
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

## アップグレードの事前確認をする
SSHでProxmoxのマシンに接続します

下記コマンドを実行してチェックをします
```
pve8to9
```
```
root@pve:~# pve8to9
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_ADDRESS = "ja_JP.UTF-8",
	LC_NAME = "ja_JP.UTF-8",
	LC_MONETARY = "ja_JP.UTF-8",
	LC_PAPER = "ja_JP.UTF-8",
	LC_IDENTIFICATION = "ja_JP.UTF-8",
	LC_TELEPHONE = "ja_JP.UTF-8",
	LC_MEASUREMENT = "ja_JP.UTF-8",
	LC_TIME = "ja_JP.UTF-8",
	LC_NUMERIC = "ja_JP.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
= CHECKING VERSION INFORMATION FOR PVE PACKAGES =

Checking for package updates..
PASS: all packages up-to-date

Checking proxmox-ve package version..
PASS: proxmox-ve package has version >= 8.4-0

Checking running kernel version..
PASS: running kernel '6.8.12-8-pve' is considered suitable for upgrade.

= CHECKING CLUSTER HEALTH/SETTINGS =

SKIP: standalone node.

= CHECKING HYPER-CONVERGED CEPH STATUS =

SKIP: no hyper-converged ceph setup detected!

= CHECKING CONFIGURED STORAGES =

PASS: storage 'add-data' enabled and active.
PASS: storage 'local' enabled and active.
PASS: storage 'local-lvm' enabled and active.
PASS: storage 'vm-backup' enabled and active.
INFO: Checking storage content type configuration..
PASS: no storage content problems found
PASS: no storage re-uses a directory for multiple content types.
INFO: Check for usage of native GlusterFS storage plugin...
PASS: No GlusterFS storage found.
INFO: Checking whether all external RBD storages have the 'keyring' option configured
SKIP: No RBD storage configured.

= VIRTUAL GUEST CHECKS =

INFO: Checking for running guests..
PASS: no running guest detected.
INFO: Checking if LXCFS is running with FUSE3 library, if already upgraded..
SKIP: not yet upgraded, no need to check the FUSE library version LXCFS uses
INFO: Checking for VirtIO devices that would change their MTU...
PASS: All guest config descriptions fit in the new limit of 8 KiB
INFO: Checking container configs for deprecated lxc.cgroup entries
PASS: No legacy 'lxc.cgroup' keys found.
INFO: Checking VM configurations for outdated machine versions
PASS: All VM machine versions are recent enough

= MISCELLANEOUS CHECKS =

INFO: Checking common daemon services..
PASS: systemd unit 'pveproxy.service' is in state 'active'
PASS: systemd unit 'pvedaemon.service' is in state 'active'
PASS: systemd unit 'pvescheduler.service' is in state 'active'
PASS: systemd unit 'pvestatd.service' is in state 'active'
INFO: Checking for supported & active NTP service..
PASS: Detected active time synchronisation unit 'chrony.service'
INFO: Checking if the local node's hostname 'pve' is resolvable..
INFO: Checking if resolved IP is configured on local node..
PASS: Resolved node IP '192.168.10.111' configured and active on single interface.
INFO: Check node certificate's RSA key size
PASS: Certificate 'pve-root-ca.pem' passed Debian Busters (and newer) security level for TLS connections (4096 >= 2048)
PASS: Certificate 'pve-ssl.pem' passed Debian Busters (and newer) security level for TLS connections (2048 >= 2048)
INFO: Checking backup retention settings..
PASS: no backup retention problems found.
INFO: checking CIFS credential location..
PASS: no CIFS credentials at outdated location found.
INFO: Checking permission system changes..
INFO: Checking custom role IDs
PASS: no custom roles defined
INFO: Checking node and guest description/note length..
PASS: All node config descriptions fit in the new limit of 64 KiB
INFO: Checking if the suite for the Debian security repository is correct..
PASS: found no suite mismatch
INFO: Checking for existence of NVIDIA vGPU Manager..
PASS: No NVIDIA vGPU Service found.
INFO: Checking bootloader configuration...
PASS: bootloader packages installed correctly
INFO: Check for dkms modules...
SKIP: could not get dkms status
INFO: Check for legacy 'filter' or 'group' sections in /etc/pve/notifications.cfg...
INFO: Check for legacy 'notification-policy' or 'notification-target' options in /etc/pve/jobs.cfg...
PASS: No legacy 'notification-policy' or 'notification-target' options found!
INFO: Check for LVM autoactivation settings on LVM and LVM-thin storages...
NOTICE: storage 'add-data' has guest volumes with autoactivation enabled
NOTICE: storage 'local-lvm' has guest volumes with autoactivation enabled
NOTICE: Starting with PVE 9, autoactivation will be disabled for new LVM/LVM-thin guest volumes. This system has some volumes that still have autoactivation enabled. All volumes with autoactivations reside on local storage, where this normally does not cause any issues.
You can run the following command to disable autoactivation for existing LVM/LVM-thin guest volumes:

	/usr/share/pve-manager/migrations/pve-lvm-disable-autoactivation

INFO: Checking lvm config for thin_check_options...
PASS: Check for correct thin_check_options passed
INFO: Check space requirements for RRD migration...
PASS: Enough free disk space for increased RRD metric granularity requirements, which is roughly 74.66 MiB.
INFO: Checking for IPAM DB files that have not yet been migrated.
PASS: No legacy IPAM DB found.
PASS: No legacy MAC DB found.
INFO: Checking if the legacy sysctl file '/etc/sysctl.conf' needs to be migrated to new '/etc/sysctl.d/' path.
PASS: Legacy file '/etc/sysctl.conf' exists but does not contain any settings.
INFO: Checking if matching CPU microcode package is installed.
WARN: The matching CPU microcode package 'amd64-microcode' could not be found! Consider installing it to receive the latest security and bug fixes for your CPU.
	Ensure you enable the 'non-free-firmware' component in the apt sources and run:
	apt install amd64-microcode
SKIP: NOTE: Expensive checks, like CT cgroupv2 compat, not performed without '--full' parameter

= SUMMARY =

TOTAL:    45
PASSED:   35
SKIPPED:  6
WARNINGS: 1
FAILURES: 0

ATTENTION: Please check the output for detailed information!
root@pve:~# 
```

フルのオプションを付けて確認する
```
pve8to9 --full
```
```
root@pve:~# pve8to9 --full
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_ADDRESS = "ja_JP.UTF-8",
	LC_NAME = "ja_JP.UTF-8",
	LC_MONETARY = "ja_JP.UTF-8",
	LC_PAPER = "ja_JP.UTF-8",
	LC_IDENTIFICATION = "ja_JP.UTF-8",
	LC_TELEPHONE = "ja_JP.UTF-8",
	LC_MEASUREMENT = "ja_JP.UTF-8",
	LC_TIME = "ja_JP.UTF-8",
	LC_NUMERIC = "ja_JP.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
= CHECKING VERSION INFORMATION FOR PVE PACKAGES =

Checking for package updates..
PASS: all packages up-to-date

Checking proxmox-ve package version..
PASS: proxmox-ve package has version >= 8.4-0

Checking running kernel version..
PASS: running kernel '6.8.12-8-pve' is considered suitable for upgrade.

= CHECKING CLUSTER HEALTH/SETTINGS =

SKIP: standalone node.

= CHECKING HYPER-CONVERGED CEPH STATUS =

SKIP: no hyper-converged ceph setup detected!

= CHECKING CONFIGURED STORAGES =

PASS: storage 'add-data' enabled and active.
PASS: storage 'local' enabled and active.
PASS: storage 'local-lvm' enabled and active.
PASS: storage 'vm-backup' enabled and active.
INFO: Checking storage content type configuration..
PASS: no storage content problems found
PASS: no storage re-uses a directory for multiple content types.
INFO: Check for usage of native GlusterFS storage plugin...
PASS: No GlusterFS storage found.
INFO: Checking whether all external RBD storages have the 'keyring' option configured
SKIP: No RBD storage configured.

= VIRTUAL GUEST CHECKS =

INFO: Checking for running guests..
PASS: no running guest detected.
INFO: Checking if LXCFS is running with FUSE3 library, if already upgraded..
SKIP: not yet upgraded, no need to check the FUSE library version LXCFS uses
INFO: Checking for VirtIO devices that would change their MTU...
PASS: All guest config descriptions fit in the new limit of 8 KiB
INFO: Checking container configs for deprecated lxc.cgroup entries
PASS: No legacy 'lxc.cgroup' keys found.
INFO: Checking VM configurations for outdated machine versions
PASS: All VM machine versions are recent enough

= MISCELLANEOUS CHECKS =

INFO: Checking common daemon services..
PASS: systemd unit 'pveproxy.service' is in state 'active'
PASS: systemd unit 'pvedaemon.service' is in state 'active'
PASS: systemd unit 'pvescheduler.service' is in state 'active'
PASS: systemd unit 'pvestatd.service' is in state 'active'
INFO: Checking for supported & active NTP service..
PASS: Detected active time synchronisation unit 'chrony.service'
INFO: Checking if the local node's hostname 'pve' is resolvable..
INFO: Checking if resolved IP is configured on local node..
PASS: Resolved node IP '192.168.10.111' configured and active on single interface.
INFO: Check node certificate's RSA key size
PASS: Certificate 'pve-root-ca.pem' passed Debian Busters (and newer) security level for TLS connections (4096 >= 2048)
PASS: Certificate 'pve-ssl.pem' passed Debian Busters (and newer) security level for TLS connections (2048 >= 2048)
INFO: Checking backup retention settings..
PASS: no backup retention problems found.
INFO: checking CIFS credential location..
PASS: no CIFS credentials at outdated location found.
INFO: Checking permission system changes..
INFO: Checking custom role IDs
PASS: no custom roles defined
INFO: Checking node and guest description/note length..
PASS: All node config descriptions fit in the new limit of 64 KiB
INFO: Checking if the suite for the Debian security repository is correct..
PASS: found no suite mismatch
INFO: Checking for existence of NVIDIA vGPU Manager..
PASS: No NVIDIA vGPU Service found.
INFO: Checking bootloader configuration...
PASS: bootloader packages installed correctly
INFO: Check for dkms modules...
SKIP: could not get dkms status
INFO: Check for legacy 'filter' or 'group' sections in /etc/pve/notifications.cfg...
INFO: Check for legacy 'notification-policy' or 'notification-target' options in /etc/pve/jobs.cfg...
PASS: No legacy 'notification-policy' or 'notification-target' options found!
INFO: Check for LVM autoactivation settings on LVM and LVM-thin storages...
NOTICE: storage 'add-data' has guest volumes with autoactivation enabled
NOTICE: storage 'local-lvm' has guest volumes with autoactivation enabled
NOTICE: Starting with PVE 9, autoactivation will be disabled for new LVM/LVM-thin guest volumes. This system has some volumes that still have autoactivation enabled. All volumes with autoactivations reside on local storage, where this normally does not cause any issues.
You can run the following command to disable autoactivation for existing LVM/LVM-thin guest volumes:

	/usr/share/pve-manager/migrations/pve-lvm-disable-autoactivation

INFO: Checking lvm config for thin_check_options...
PASS: Check for correct thin_check_options passed
INFO: Check space requirements for RRD migration...
PASS: Enough free disk space for increased RRD metric granularity requirements, which is roughly 74.66 MiB.
INFO: Checking for IPAM DB files that have not yet been migrated.
PASS: No legacy IPAM DB found.
PASS: No legacy MAC DB found.
INFO: Checking if the legacy sysctl file '/etc/sysctl.conf' needs to be migrated to new '/etc/sysctl.d/' path.
PASS: Legacy file '/etc/sysctl.conf' exists but does not contain any settings.
INFO: Checking if matching CPU microcode package is installed.
WARN: The matching CPU microcode package 'amd64-microcode' could not be found! Consider installing it to receive the latest security and bug fixes for your CPU.
	Ensure you enable the 'non-free-firmware' component in the apt sources and run:
	apt install amd64-microcode
SKIP: No containers on node detected.

= SUMMARY =

TOTAL:    45
PASSED:   35
SKIPPED:  6
WARNINGS: 1
FAILURES: 0

ATTENTION: Please check the output for detailed information!
root@pve:~#
```

## リポジトリを更新します
下記コマンドを実行します
```
apt update
apt dist-upgrade
pveversion
```
```
root@pve:~# pveversion
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_ADDRESS = "ja_JP.UTF-8",
	LC_NAME = "ja_JP.UTF-8",
	LC_MONETARY = "ja_JP.UTF-8",
	LC_PAPER = "ja_JP.UTF-8",
	LC_IDENTIFICATION = "ja_JP.UTF-8",
	LC_TELEPHONE = "ja_JP.UTF-8",
	LC_MEASUREMENT = "ja_JP.UTF-8",
	LC_TIME = "ja_JP.UTF-8",
	LC_NUMERIC = "ja_JP.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
pve-manager/8.4.11/14a32011146091ed (running kernel: 6.8.12-8-pve)
root@pve:~# 
```

## リポジトリを変更します
下記コマンド実行します
```
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list 
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-enterprise.list
```
```
cat /etc/apt/sources.list
cat /etc/apt/sources.list.d/pve-enterprise.list
```
```
root@pve:~# cat /etc/apt/sources.list
deb http://ftp.jp.debian.org/debian trixie main contrib

deb http://ftp.jp.debian.org/debian trixie-updates main contrib

# security updates
deb http://security.debian.org trixie-security main contrib

deb http://download.proxmox.com/debian/pve trixie pve-no-subscription

root@pve:~# cat /etc/apt/sources.list.d/pve-enterprise.list
# deb https://enterprise.proxmox.com/debian/pve trixie pve-enterprise

root@pve:~#
```

パッケージリポジトリを追加する
```
cat > /etc/apt/sources.list.d/proxmox.sources << EOF
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

アップデートをすると重複エラーが表示される
```
apt update
```
```
W: Target Packages (pve-no-subscription/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
W: Target Packages (pve-no-subscription/binary-all/Packages) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
W: Target Translations (pve-no-subscription/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
W: Target Packages (pve-no-subscription/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
W: Target Packages (pve-no-subscription/binary-all/Packages) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
W: Target Translations (pve-no-subscription/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list:8 and /etc/apt/sources.list.d/proxmox.sources:1
```

### エラーを解消する
"apt modernize-sources"コマンド実行する
```
root@pve:~# apt modernize-sources
The following files need modernizing:
  - /etc/apt/sources.list
  - /etc/apt/sources.list.d/ceph.list
  - /etc/apt/sources.list.d/pve-enterprise.list

Modernizing will replace .list files with the new .sources format,
add Signed-By values where they can be determined automatically,
and save the old files into .list.bak files.

This command supports the 'signed-by' and 'trusted' options. If you
have specified other options inside [] brackets, please transfer them
manually to the output files; see sources.list(5) for a mapping.

For a simulation, respond N in the following prompt.
Rewrite 3 sources? [Y/n] Y
Modernizing /etc/apt/sources.list...
- Writing /etc/apt/sources.list.d/debian.sources
- Writing /etc/apt/sources.list.d/proxmox.sources

Modernizing /etc/apt/sources.list.d/ceph.list...

Modernizing /etc/apt/sources.list.d/pve-enterprise.list...

root@pve:~#
```

エラーがでなくなった
```
root@pve:~# apt update
Hit:1 http://security.debian.org trixie-security InRelease
Hit:2 http://ftp.jp.debian.org/debian trixie InRelease                      
Hit:3 http://ftp.jp.debian.org/debian trixie-updates InRelease              
Hit:4 http://download.proxmox.com/debian/pve trixie InRelease               
1 package can be upgraded. Run 'apt list --upgradable' to see it.
root@pve:~# nano /etc/apt/sources.list
root@pve:~# nano /etc/apt/sources.list.d/proxmox.sources
root@pve:~# apt update
Hit:1 http://ftp.jp.debian.org/debian trixie InRelease
Hit:2 http://ftp.jp.debian.org/debian trixie-updates InRelease                     
Hit:3 http://security.debian.org trixie-security InRelease                         
Hit:4 http://download.proxmox.com/debian/pve trixie InRelease
1 package can be upgraded. Run 'apt list --upgradable' to see it.
root@pve:~#
```

## Proxmox9へアップグレードする
下記コマンドを実行します
```
apt update
apt dist-upgrade
```

何回か選択を迫られるので下記の選択をした
```
q
N
N
```
アップグレードが終わったら再起動をする

## トラブル
起動しなくなった、BIOSの画面になってしまう\
BIOS画面から、直接起動ディスクを指定してもBIOSの画面になってしまう\
ProxmoxのインストールUSBを作成して、USBメモリを差しインストーラーを起動する際に"Boot Rescue"を選択し起動できた\
VMをバックアップする

### GRUBを再インストールする
"lsblk"でgrubが入っているディスクを探す
- "/boot/efi"が入っているパーティションを探す
```
root@pve:~# lsblk
NAME                             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                                8:0    0 465.8G  0 disk 
├─sda1                             8:1    0  1007K  0 part 
├─sda2                             8:2    0     1G  0 part /boot/efi
└─sda3                             8:3    0 464.8G  0 part 
  ├─pve-swap                     252:2    0     8G  0 lvm  [SWAP]
  ├─pve-root                     252:3    0    96G  0 lvm  /
  ├─pve-data_tmeta               252:4    0   3.4G  0 lvm  
  │ └─pve-data-tpool             252:6    0 337.9G  0 lvm  
  │   ├─pve-data                 252:7    0 337.9G  1 lvm  
  │   ├─pve-vm--101--disk--0     252:8    0    32G  0 lvm  
  │   ├─pve-vm--102--disk--0     252:10   0    32G  0 lvm  
  │   ├─pve-vm--102--disk--1     252:13   0    32G  0 lvm  
  │   ├─pve-vm--105--disk--0     252:15   0    32G  0 lvm  
  │   ├─pve-vm--105--disk--1     252:17   0    50G  0 lvm  
  │   ├─pve-vm--111--disk--0     252:19   0    32G  0 lvm  
  │   ├─pve-vm--112--disk--0     252:21   0    32G  0 lvm  
  │   ├─pve-vm--113--disk--0     252:23   0    32G  0 lvm  
  │   ├─pve-vm--114--disk--0     252:25   0    32G  0 lvm  
  │   ├─pve-vm--115--disk--0     252:27   0    32G  0 lvm  
  │   ├─pve-vm--116--disk--0     252:29   0    10G  0 lvm  
  │   ├─pve-vm--117--disk--0     252:32   0    32G  0 lvm  
  │   ├─pve-vm--120--disk--0     252:34   0    10G  0 lvm  
  │   ├─pve-vm--121--disk--0     252:36   0    10G  0 lvm  
  │   ├─pve-vm--122--disk--0     252:38   0    10G  0 lvm  
  │   ├─pve-vm--129--disk--0     252:40   0    10G  0 lvm  
  │   ├─pve-vm--128--disk--0     252:42   0    20G  0 lvm  
  │   ├─pve-vm--127--disk--0     252:44   0    20G  0 lvm  
  │   ├─pve-vm--126--disk--0     252:45   0    20G  0 lvm  
  │   ├─pve-vm--131--disk--0     252:46   0    20G  0 lvm  
  │   ├─pve-vm--132--disk--0     252:47   0    20G  0 lvm  
  │   ├─pve-vm--133--disk--0     252:48   0    20G  0 lvm  
  │   ├─pve-vm--134--disk--0     252:49   0    20G  0 lvm  
  │   └─pve-vm--135--disk--0     252:50   0    32G  0 lvm  
  └─pve-data_tdata               252:5    0 337.9G  0 lvm  
    └─pve-data-tpool             252:6    0 337.9G  0 lvm  
      ├─pve-data                 252:7    0 337.9G  1 lvm  
      ├─pve-vm--101--disk--0     252:8    0    32G  0 lvm  
      ├─pve-vm--102--disk--0     252:10   0    32G  0 lvm  
      ├─pve-vm--102--disk--1     252:13   0    32G  0 lvm  
      ├─pve-vm--105--disk--0     252:15   0    32G  0 lvm  
      ├─pve-vm--105--disk--1     252:17   0    50G  0 lvm  
      ├─pve-vm--111--disk--0     252:19   0    32G  0 lvm  
      ├─pve-vm--112--disk--0     252:21   0    32G  0 lvm  
      ├─pve-vm--113--disk--0     252:23   0    32G  0 lvm  
      ├─pve-vm--114--disk--0     252:25   0    32G  0 lvm  
      ├─pve-vm--115--disk--0     252:27   0    32G  0 lvm  
      ├─pve-vm--116--disk--0     252:29   0    10G  0 lvm  
      ├─pve-vm--117--disk--0     252:32   0    32G  0 lvm  
      ├─pve-vm--120--disk--0     252:34   0    10G  0 lvm  
      ├─pve-vm--121--disk--0     252:36   0    10G  0 lvm  
      ├─pve-vm--122--disk--0     252:38   0    10G  0 lvm  
      ├─pve-vm--129--disk--0     252:40   0    10G  0 lvm  
      ├─pve-vm--128--disk--0     252:42   0    20G  0 lvm  
      ├─pve-vm--127--disk--0     252:44   0    20G  0 lvm  
      ├─pve-vm--126--disk--0     252:45   0    20G  0 lvm  
      ├─pve-vm--131--disk--0     252:46   0    20G  0 lvm  
      ├─pve-vm--132--disk--0     252:47   0    20G  0 lvm  
      ├─pve-vm--133--disk--0     252:48   0    20G  0 lvm  
      ├─pve-vm--134--disk--0     252:49   0    20G  0 lvm  
      └─pve-vm--135--disk--0     252:50   0    32G  0 lvm  
sdb                                8:16   0   3.6T  0 disk 
└─sdb1                             8:17   0   3.6T  0 part /mnt/pve/vm-backup
sdc                                8:32   1  28.9G  0 disk 
├─sdc1                             8:33   1   282K  0 part 
├─sdc2                             8:34   1     8M  0 part 
├─sdc3                             8:35   1   1.5G  0 part 
└─sdc4                             8:36   1   300K  0 part 
nvme0n1                          259:0    0 931.5G  0 disk 
├─add--data-add--data_tmeta      252:0    0   9.3G  0 lvm  
│ └─add--data-add--data-tpool    252:9    0 912.8G  0 lvm  
│   ├─add--data-add--data        252:11   0 912.8G  1 lvm  
│   ├─add--data-vm--109--disk--0 252:12   0   200G  0 lvm  
│   ├─add--data-vm--124--disk--0 252:14   0    32G  0 lvm  
│   ├─add--data-vm--130--disk--0 252:16   0    32G  0 lvm  
│   ├─add--data-vm--134--disk--0 252:18   0    20G  0 lvm  
│   ├─add--data-vm--133--disk--0 252:20   0    20G  0 lvm  
│   ├─add--data-vm--132--disk--0 252:22   0    20G  0 lvm  
│   ├─add--data-vm--136--disk--0 252:24   0    50G  0 lvm  
│   ├─add--data-vm--100--disk--0 252:26   0    70G  0 lvm  
│   ├─add--data-vm--123--disk--0 252:28   0    10G  0 lvm  
│   ├─add--data-vm--138--disk--0 252:30   0    32G  0 lvm  
│   ├─add--data-vm--144--disk--0 252:31   0    50G  0 lvm  
│   ├─add--data-vm--145--disk--0 252:33   0    32G  0 lvm  
│   ├─add--data-vm--146--disk--0 252:35   0    10G  0 lvm  
│   ├─add--data-vm--151--disk--0 252:37   0    32G  0 lvm  
│   ├─add--data-vm--200--disk--0 252:39   0     4M  0 lvm  
│   ├─add--data-vm--200--disk--1 252:41   0   100G  0 lvm  
│   └─add--data-vm--200--disk--2 252:43   0     4M  0 lvm  
└─add--data-add--data_tdata      252:1    0 912.8G  0 lvm  
  └─add--data-add--data-tpool    252:9    0 912.8G  0 lvm  
    ├─add--data-add--data        252:11   0 912.8G  1 lvm  
    ├─add--data-vm--109--disk--0 252:12   0   200G  0 lvm  
    ├─add--data-vm--124--disk--0 252:14   0    32G  0 lvm  
    ├─add--data-vm--130--disk--0 252:16   0    32G  0 lvm  
    ├─add--data-vm--134--disk--0 252:18   0    20G  0 lvm  
    ├─add--data-vm--133--disk--0 252:20   0    20G  0 lvm  
    ├─add--data-vm--132--disk--0 252:22   0    20G  0 lvm  
    ├─add--data-vm--136--disk--0 252:24   0    50G  0 lvm  
    ├─add--data-vm--100--disk--0 252:26   0    70G  0 lvm  
    ├─add--data-vm--123--disk--0 252:28   0    10G  0 lvm  
    ├─add--data-vm--138--disk--0 252:30   0    32G  0 lvm  
    ├─add--data-vm--144--disk--0 252:31   0    50G  0 lvm  
    ├─add--data-vm--145--disk--0 252:33   0    32G  0 lvm  
    ├─add--data-vm--146--disk--0 252:35   0    10G  0 lvm  
    ├─add--data-vm--151--disk--0 252:37   0    32G  0 lvm  
    ├─add--data-vm--200--disk--0 252:39   0     4M  0 lvm  
    ├─add--data-vm--200--disk--1 252:41   0   100G  0 lvm  
    └─add--data-vm--200--disk--2 252:43   0     4M  0 lvm  
root@pve:~# 
```

bootのフォルダを確認する
```
root@pve:~# ls -al /dev/sda2
brw-rw---- 1 root disk 8, 2 Aug 14 15:41 /dev/sda2
root@pve:~# ls -al /boot/efi/EFI/
total 16
drwxr-xr-x 4 root root 4096 May 27  2023 .
drwxr-xr-x 3 root root 4096 Jan  1  1970 ..
drwxr-xr-x 2 root root 4096 May 27  2023 BOOT
drwxr-xr-x 2 root root 4096 May 27  2023 proxmox
root@pve:~# 
```

GRUBを再インストールする
```
update-grub
grub-install /dev/sda2
```

上記実行後、問題なく起動できることを確認する

## アップグレード完了
"neofetch"で確認する
```
root@pve:~# neofetch
         .://:`              `://:.            root@pve 
       `hMMMMMMd/          /dMMMMMMh`          -------- 
        `sMMMMMMMd:      :mMMMMMMMs`           OS: Proxmox VE 9.0.5 x86_64 
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`   Kernel: 6.14.8-2-pve 
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`   Uptime: 9 mins 
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`     Packages: 987 (dpkg) 
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.       Shell: bash 5.2.37 
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.         Resolution: 1920x1080 
        -+ooooooo/.`sMMs`./ooooooo+-           Terminal: /dev/pts/0 
          :oooooooo/`..`/oooooooo:             CPU: AMD Ryzen 7 5700G with Radeon Graphics (16) @ 4.673GHz 
          :oooooooo/`..`/oooooooo:             GPU: AMD ATI Radeon Vega Series / Radeon Vega Mobile Series 
        -+ooooooo/.`sMMs`./ooooooo+-           Memory: 1602MiB / 60137MiB 
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
