---
title: NFS-serverを構築してPV(PersistentVolume)を作成する
date: 2024-08-14
slug: kubernetes-pv-pvc
categories:
    - Kubernetes
---

## 環境
- Kubernetes 1.30.3

## 参考URL
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/storage/storage-classes/
- https://changineer.info/vmware/hypervisor/vmware_ubuntu_nfs.html
- https://blog.denet.co.jp/building-an-nfs-server-and-using-it-as-storage-from-kubernetes/
- https://thinkit.co.jp/article/14195?page=0%2C1

## NFSサーバーの構築
"nfs-server"を構築するサーバーで実行するコマンド\
インストールをする
```
sudo apt install nfs-kernel-server
```

NFSで公開するディレクトリの作成をする
```
mkdir /home/mao/nfs
```

設定ファイルを編集する
```
sudo nano /etc/exports
```

設定ファイルに下記を追加
```
/home/mao/nfs 192.168.10.0/24(rw,sync,no_root_squash)
```

再起動と自動起動の有効化
```
sudo systemctl restart nfs-server.service
sudo systemctl enable nfs-server.service
```

設定が読み込まれているか確認する
```
sudo exportfs
```

## nfs-commonをインストールする
Kubernetesの全てのnode(Control-Plane,Woker-Node)に"nfs-common"をインストールする
```
sudo apt install nfs-common
```
```
mao@k8s-control-plane-01:~$ sudo apt install nfs-common
[sudo] password for mao: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libevent-core-2.1-7t64 libnfsidmap1 rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libevent-core-2.1-7t64 libnfsidmap1 nfs-common rpcbind
0 upgraded, 5 newly installed, 0 to remove and 30 not upgraded.
Need to get 491 kB of archives.
After this operation, 1680 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 libevent-core-2.1-7t64 amd64 2.1.12-stable-9ubuntu2 [91.3 kB]
Get:2 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 libnfsidmap1 amd64 1:2.6.4-3ubuntu5 [48.2 kB]
Get:3 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 rpcbind amd64 1.2.6-7ubuntu2 [46.5 kB]
Get:4 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 keyutils amd64 1.6.3-3build1 [56.8 kB]
Get:5 http://jp.archive.ubuntu.com/ubuntu noble/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5 [248 kB]
Fetched 491 kB in 2s (295 kB/s)   
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libevent-core-2.1-7t64:amd64.
(Reading database ... 110850 files and directories currently installed.)
Preparing to unpack .../libevent-core-2.1-7t64_2.1.12-stable-9ubuntu2_amd64.deb ...
Unpacking libevent-core-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Selecting previously unselected package libnfsidmap1:amd64.
Preparing to unpack .../libnfsidmap1_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.4-3ubuntu5) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-7ubuntu2_amd64.deb ...
Unpacking rpcbind (1.2.6-7ubuntu2) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.3-3build1_amd64.deb ...
Unpacking keyutils (1.6.3-3build1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5) ...
Setting up libnfsidmap1:amd64 (1:2.6.4-3ubuntu5) ...
Setting up rpcbind (1.2.6-7ubuntu2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.3-3build1) ...
Setting up libevent-core-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Setting up nfs-common (1:2.6.4-3ubuntu5) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 79.)
debconf: falling back to frontend: Readline

Creating config file /etc/idmapd.conf with new version
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 79.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 79.)
debconf: falling back to frontend: Readline

Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...

info: Adding system user `statd' (UID 106) ...
info: Adding new user `statd' (UID 106) with group `nogroup' ...
info: Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
Processing triggers for libc-bin (2.39-0ubuntu8.2) ...
Scanning processes...                                                            
Scanning linux images...                                                         

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
mao@k8s-control-plane-01:~$ 
```

## NFS-server上にPV(PersistentVolume)用のディレクトリを作成する
nfs-serverで下記のコマンドを実行してPV用のディレクトリを作成する
- "pv0001"
```
mkdir /home/mao/nfs/pv0001
```

## PV(PersistentVolume)のマニフェストファイルをデプロイする
マニフェストファイルを作成する
- create-persistent-volume.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
  annotations:
    volume.beta.kubernetes.io/storage-class: "slow"
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.10.17
    path: /home/mao/nfs/pv0001
```

デプロイします
```
kubectl apply -f create-persistent-volume.yaml
```
```
mao@k8s-control-plane-01:~$ kubectl apply -f create-persistent-volume.yaml
Warning: metadata.annotations[volume.beta.kubernetes.io/storage-class]: deprecated since v1.8; use "storageClassName" attribute instead
Warning: spec.persistentVolumeReclaimPolicy: The Recycle reclaim policy is deprecated. Instead, the recommended approach is to use dynamic provisioning.
persistentvolume/pv0001 created
mao@k8s-control-plane-01:~$ kubectl delete -f create-persistent-volume.yaml
persistentvolume "pv0001" deleted
mao@k8s-control-plane-01:~$ 
```

警告が出たのでマニフェストファイルのストレージクラスとポリシーを修正します、合わせてアクセスモードも修正します
- "volume.beta.kubernetes.io/storage-class"を"spec.storageClassName"へ修正
- "persistentVolumeReclaimPolicy"の"Recycle"を"Delete"へ修正
- "spec.accessModes"を"ReadWriteOnce"を"ReadWriteMany"へ修正
- "spec.nfs.server"は"nfs-server"のIPアドレスを指定する
- "spec.nfs.path"はPV用ディレクトリを指定します
- create-persistent-volume.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    #- ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  storageClassName: slow
  nfs:
    server: 192.168.10.17
    path: /home/mao/nfs/pv0001
```

デプロイします
```
mao@k8s-control-plane-01:~$ kubectl apply -f create-persistent-volume.yaml
persistentvolume/pv0001 created
mao@k8s-control-plane-01:~$
```

作成されているか確認をします
```
kubectl get pv
```
```
mao@k8s-control-plane-01:~$ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv0001   5Gi        RWX            Delete           Available           slow           <unset>                          2m5s
mao@k8s-control-plane-01:~$ 
```
- "STATUS"が"Available"になっていればOK

無事作成されました

## PVC(PersistentVolumeClaim)をデプロイする
マニフェストファイルを作成します
- create-persistent-volume-claim.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    #- ReadWriteOnce
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: slow
```

デプロイします
```
kubectl apply -f create-persistent-volume-claim.yaml
```
```
mao@k8s-control-plane-01:~$ kubectl apply -f create-persistent-volume-claim.yaml
persistentvolumeclaim/myclaim created
mao@k8s-control-plane-01:~$ 
```

作成されているか確認します
```
kubectl get pvc
```
```
mao@k8s-control-plane-01:~$ kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
myclaim   Bound    pv0001   5Gi        RWX            slow           <unset>                 31s
mao@k8s-control-plane-01:~$ 
```
- "STATUS"が"Bound"になっていればOK

無事作成されました

これでPV,PVCが必要なアプリケーションをデプロイできるようになりました

## PVC、PVの削除
- 削除していきます
```
kubectl delete -f create-persistent-volume-claim.yaml
kubectl delete -f create-persistent-volume.yaml
```
```
mao@k8s-control-plane-01:~$ kubectl delete -f create-persistent-volume-claim.yaml
persistentvolumeclaim "myclaim" deleted
mao@k8s-control-plane-01:~$ kubectl delete -f create-persistent-volume.yaml
persistentvolume "pv0001" deleted
mao@k8s-control-plane-01:~$ kubectl get pvc
No resources found in default namespace.
mao@k8s-control-plane-01:~$ kubectl get pv
No resources found
mao@k8s-control-plane-01:~$ 
```
問題なく削除されたことを確認

## 完了
