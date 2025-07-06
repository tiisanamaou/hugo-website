---
title: Cursorのインストール方法のメモ
date: 2025-07-06
slug: cursor-install
categories:
    - cursor
---

## HPからAppImageをダウンロードする
- https://www.cursor.com/ja

下記のようなファイルがダウンロードされる
- Cursor-0.48.7-x86_64.AppImage

ダウンロードしたファイルを右クリックして"プロパティ"を選択する
"Executable as Program"をオンにする

ダブルクリックしたが起動できなかったので、ターミナルから起動しようとしたらエラーが表示された
```
mao@mao (x86_64) : /home/mao/Downloads 
> ./Cursor-0.48.7-x86_64.AppImage
dlopen(): error loading libfuse.so.2

AppImages require FUSE to run. 
You might still be able to extract the contents of this AppImage 
if you run it with the --appimage-extract option. 
See https://github.com/AppImage/AppImageKit/wiki/FUSE 
for more information
```
FUSEが必要で、インストールされていないから実行できないよう
なので、インストールをする

### 下記の方法でインストールするとシステムが破損するので注意
```
sudo apt install fuse
```
```
mao@mao (x86_64) : /home/mao/Downloads 
> sudo apt install fuse
[sudo] password for mao: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  gir1.2-gnomeautoar-0.1 gir1.2-gnomedesktop-3.0 libcue2 libdecor-0-0
  libdecor-0-plugin-1-gtk libdee-1.0-4 libei1 libexiv2-27 libfreerdp-server3-3
  libfreerdp3-3 libgexiv2-2 libgsf-1-114 libgsf-1-common libntfs-3g89t64
  libportal-gtk4-1 libportal1 libtotem-plparser-common libtotem-plparser18
  libtracker-sparql-3.0-0 libtss2-tcti-libtpms0t64 libtss2-tcti-spi-helper0t64
  libtss2-tctildr0t64 libunity-protocol-private0
  libunity-scopes-json-def-desktop libunity9 libwinpr3-3 nautilus-data tracker
  tracker-extract tracker-miner-fs xwayland
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libfuse2t64
The following packages will be REMOVED:
  fuse3 gnome-remote-desktop gnome-shell-extension-desktop-icons-ng gvfs-fuse
  nautilus ntfs-3g ubuntu-desktop-minimal ubuntu-session xdg-desktop-portal
  xdg-desktop-portal-gnome xdg-desktop-portal-gtk
The following NEW packages will be installed:
  fuse libfuse2t64
0 upgraded, 2 newly installed, 11 to remove and 0 not upgraded.
Need to get 117 kB of archives.
After this operation, 6,632 kB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu noble/universe amd64 libfuse2t64 amd64 2.9.9-8.1build1 [89.9 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/universe amd64 fuse amd64 2.9.9-8.1build1 [27.6 kB]
Fetched 117 kB in 1s (110 kB/s)
(Reading database ... 198533 files and directories currently installed.)
Removing gnome-remote-desktop (46.3-0ubuntu1) ...
Selecting previously unselected package libfuse2t64:amd64.
(Reading database ... 198506 files and directories currently installed.)
Preparing to unpack .../libfuse2t64_2.9.9-8.1build1_amd64.deb ...
Unpacking libfuse2t64:amd64 (2.9.9-8.1build1) ...
(Reading database ... 198519 files and directories currently installed.)
Removing ubuntu-desktop-minimal (1.539.2) ...
Removing ubuntu-session (46.0-1ubuntu4) ...
Removing xdg-desktop-portal-gnome (46.2-0ubuntu1) ...
dpkg: xdg-desktop-portal: dependency problems, but removing anyway as you reques
ted:
 xdg-desktop-portal-gtk depends on xdg-desktop-portal (>= 1.14.0).

Removing xdg-desktop-portal (1.18.4-1ubuntu2.24.04.1) ...
dpkg: fuse3: dependency problems, but removing anyway as you requested:
 snapd depends on fuse3 (>= 3.10.5-1) | fuse; however:
  Package fuse3 is to be removed.
  Package fuse is not installed.
  Package fuse3 which provides fuse is to be removed.
 ntfs-3g depends on fuse3.
 gvfs-fuse depends on fuse3.
 snapd depends on fuse3 (>= 3.10.5-1) | fuse; however:
  Package fuse3 is to be removed.
  Package fuse is not installed.
  Package fuse3 which provides fuse is to be removed.

Removing fuse3 (3.14.0-5build1) ...
update-initramfs: deferring update (trigger activated)
dpkg: nautilus: dependency problems, but removing anyway as you requested:
 gnome-shell-extension-desktop-icons-ng depends on nautilus (>= 3.38).

Removing nautilus (1:46.2-0ubuntu0.3) ...
Selecting previously unselected package fuse.
(Reading database ... 198439 files and directories currently installed.)
Preparing to unpack .../fuse_2.9.9-8.1build1_amd64.deb ...
Unpacking fuse (2.9.9-8.1build1) ...
(Reading database ... 198449 files and directories currently installed.)
Removing gnome-shell-extension-desktop-icons-ng (46+really47.0.9-1ubuntu1) ...
Removing gvfs-fuse (1.54.0-1ubuntu2) ...
Removing ntfs-3g (1:2022.10.3-1.2ubuntu3) ...
Removing xdg-desktop-portal-gtk (1.15.1-1build2) ...
Setting up libfuse2t64:amd64 (2.9.9-8.1build1) ...
Setting up fuse (2.9.9-8.1build1) ...
Installing new version of config file /etc/fuse.conf ...
update-initramfs: deferring update (trigger activated)
Processing triggers for initramfs-tools (0.142ubuntu25.5) ...
update-initramfs: Generating /boot/initrd.img-6.8.0-57-generic
Processing triggers for gnome-menus (3.36.0-1.1ubuntu3) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libglib2.0-0t64:amd64 (2.80.0-6ubuntu3.2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for mailcap (3.70+nmu1ubuntu1) ...
Processing triggers for desktop-file-utils (0.27-2build1) ...
mao@mao (x86_64) : /home/mao/Downloads 
> 
```

### 下記の方法でインストールすると大丈夫
- https://github.com/AppImage/AppImageKit/wiki/FUSE

```
sudo apt install libfuse2t64
```

## ダブルクリックして起動する
設定ファイルは下記の場所に生成される
```
/home/user/.cursor
/home/user/.config/Cursor
```
