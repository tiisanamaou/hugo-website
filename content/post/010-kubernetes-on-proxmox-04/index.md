---
title: kubernetesをproxmox上に立ててみた（4）/クラスターを壊す
date: 2024-07-10
slug: kubernetes-on-proxmox-04
categories:
    - Kubernetes
    - Proxmox
---

## 開発環境
- Proxmox 8.2.4
- Ubuntu Server 24.04 LTS

## Worker-Nodeをクラスターから外す
### 先にControl-Planeですること
先にControl-Plane上でクラスターから外す準備をする
```
kubectl drain k8s-worker-01 --ignore-daemonsets --delete-emptydir-data --force
kubectl get node
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl drain k8s-worker-01 --ignore-daemonsets --delete-emptydir-data --force
node/k8s-worker-01 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-h75vn, calico-system/csi-node-driver-5b5k4, kube-system/kube-proxy-wzvtf, metallb-system/speaker-xwb66
evicting pod metallb-system/controller-86f5578878-dz5zm
pod/controller-86f5578878-dz5zm evicted
node/k8s-worker-01 drained
mao@k8s-control-plane-01:~$ kubectl get node
NAME                   STATUS                     ROLES           AGE     VERSION
k8s-control-plane-01   Ready                      control-plane   3d23h   v1.30.2
k8s-worker-01          Ready,SchedulingDisabled   <none>          3d22h   v1.30.2
k8s-worker-02          Ready                      <none>          3d14h   v1.30.2
mao@k8s-control-plane-01:~$ 
```

STATUSが"SchedulingDisabled"になったらOK

### Worker-Nodeでの作業
Control-Planeでの作業ができたら、次はWorker-Nodeで作業をする
```
sudo kubeadm reset
```
実行結果
```
mao@k8s-worker-01:~$ sudo kubeadm reset
W0630 01:12:34.493241   63962 preflight.go:56] [reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0630 01:12:36.338665   63962 removeetcdmember.go:106] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] Deleted contents of the etcd data directory: /var/lib/etcd
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/super-admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```

### Worker-Node上のcalicoを削除する
削除するネットワークインターフェースを確認する
```
ip link
```
実行結果
```
mao@k8s-worker-01:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether bc:24:11:fe:35:a0 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
10: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:bb:bb:d8:11:bf brd ff:ff:ff:ff:ff:ff
mao@k8s-worker-01:~$
```

"vxlan.calico"を削除する
```
sudo ip link delete vxlan.calico
```

### 最後にControl-PlaneからWorker-Nodeを削除する
Control-Plane上で下記コマンドを実行する
```
kubectl delete node k8s-worker-01
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl delete node k8s-worker-01
node "k8s-worker-01" deleted
mao@k8s-control-plane-01:~$ 
```

削除されたことを確認する
```
kubectl get node
```
実行結果
```
mao@k8s-control-plane-01:~$ kubectl get node
NAME                   STATUS                     ROLES           AGE     VERSION
k8s-control-plane-01   Ready                      control-plane   3d23h   v1.30.2
k8s-worker-02          Ready                      <none>          3d14h   v1.30.2
mao@k8s-control-plane-01:~$ 
```


## Control-Planeをクラスターから外す、クラスターを削除する
Control-Plane上で、下記のコマンドを順に実行してクラスタをリセットする
```
sudo kubeadm reset
sudo rm -rf $HOME/.kube
sudo systemctl daemon-reload && systemctl restart kubelet
sudo systemctl restart containerd
sudo ip link
sudo ip link delete vxlan.calico
sudo ip link
```

実行結果
```
mao@k8s-control-plane-01:~$ sudo kubeadm reset
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0630 03:05:22.877860  124743 preflight.go:56] [reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
[reset] Deleted contents of the etcd data directory: /var/lib/etcd
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
W0630 03:05:30.442255  124743 cleanupnode.go:106] [reset] Failed to remove containers: [failed to stop running pod 67f96751dedca872f742388cb86d92f0388de2eef490d813d380e706cb8c7424: output: E0630 03:05:29.277302  126592 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"67f96751dedca872f742388cb86d92f0388de2eef490d813d380e706cb8c7424\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="67f96751dedca872f742388cb86d92f0388de2eef490d813d380e706cb8c7424"
time="2024-06-30T03:05:29Z" level=fatal msg="stopping the pod sandbox \"67f96751dedca872f742388cb86d92f0388de2eef490d813d380e706cb8c7424\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"67f96751dedca872f742388cb86d92f0388de2eef490d813d380e706cb8c7424\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1, failed to stop running pod 9b4a0f41c7cafc48e51a68e89a0e9ad265fee646ef148a7ade366f90be956ee4: output: E0630 03:05:29.422959  126734 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"9b4a0f41c7cafc48e51a68e89a0e9ad265fee646ef148a7ade366f90be956ee4\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="9b4a0f41c7cafc48e51a68e89a0e9ad265fee646ef148a7ade366f90be956ee4"
time="2024-06-30T03:05:29Z" level=fatal msg="stopping the pod sandbox \"9b4a0f41c7cafc48e51a68e89a0e9ad265fee646ef148a7ade366f90be956ee4\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"9b4a0f41c7cafc48e51a68e89a0e9ad265fee646ef148a7ade366f90be956ee4\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1, failed to stop running pod 976d40faccfc6491f13fc990ae3c94ee0348faa04ab74968d05b5c5904c75e12: output: E0630 03:05:29.566837  126875 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"976d40faccfc6491f13fc990ae3c94ee0348faa04ab74968d05b5c5904c75e12\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="976d40faccfc6491f13fc990ae3c94ee0348faa04ab74968d05b5c5904c75e12"
time="2024-06-30T03:05:29Z" level=fatal msg="stopping the pod sandbox \"976d40faccfc6491f13fc990ae3c94ee0348faa04ab74968d05b5c5904c75e12\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"976d40faccfc6491f13fc990ae3c94ee0348faa04ab74968d05b5c5904c75e12\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1, failed to stop running pod 28a5274b17af2a1a78e80d18c48a8732826f7a020fef31901eec76fcaec91fc5: output: E0630 03:05:29.713590  127015 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"28a5274b17af2a1a78e80d18c48a8732826f7a020fef31901eec76fcaec91fc5\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="28a5274b17af2a1a78e80d18c48a8732826f7a020fef31901eec76fcaec91fc5"
time="2024-06-30T03:05:29Z" level=fatal msg="stopping the pod sandbox \"28a5274b17af2a1a78e80d18c48a8732826f7a020fef31901eec76fcaec91fc5\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"28a5274b17af2a1a78e80d18c48a8732826f7a020fef31901eec76fcaec91fc5\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1, failed to stop running pod 2a4ddac4e1d2d11184fb82e8283250c0756a73c23fe053d12edf9a4f037eb976: output: E0630 03:05:29.858624  127156 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"2a4ddac4e1d2d11184fb82e8283250c0756a73c23fe053d12edf9a4f037eb976\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="2a4ddac4e1d2d11184fb82e8283250c0756a73c23fe053d12edf9a4f037eb976"
time="2024-06-30T03:05:29Z" level=fatal msg="stopping the pod sandbox \"2a4ddac4e1d2d11184fb82e8283250c0756a73c23fe053d12edf9a4f037eb976\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"2a4ddac4e1d2d11184fb82e8283250c0756a73c23fe053d12edf9a4f037eb976\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1, failed to stop running pod 70174a13942604fd21e17d0425064f463e35c347067301b9051206c088123112: output: E0630 03:05:30.010265  127297 remote_runtime.go:222] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"70174a13942604fd21e17d0425064f463e35c347067301b9051206c088123112\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused" podSandboxID="70174a13942604fd21e17d0425064f463e35c347067301b9051206c088123112"
time="2024-06-30T03:05:30Z" level=fatal msg="stopping the pod sandbox \"70174a13942604fd21e17d0425064f463e35c347067301b9051206c088123112\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"70174a13942604fd21e17d0425064f463e35c347067301b9051206c088123112\": plugin type=\"calico\" failed (delete): error getting ClusterInformation: Get \"https://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default\": dial tcp 10.96.0.1:443: connect: connection refused"
: exit status 1]
[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/super-admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
mao@k8s-control-plane-01:~$ 
```
```
mao@k8s-control-plane-01:~$ sudo systemctl daemon-reload && systemctl restart kubelet
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to restart 'kubelet.service'.
Authenticating as: mao
Password: 
==== AUTHENTICATION COMPLETE ====
mao@k8s-control-plane-01:~$ 
```
```
mao@k8s-control-plane-01:~$ sudo ip link 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether bc:24:11:7f:08:e4 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
9: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:ca:40:c1:cc:6f brd ff:ff:ff:ff:ff:ff
mao@k8s-control-plane-01:~$ 
```
```
mao@k8s-control-plane-01:~$ sudo ip link 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether bc:24:11:7f:08:e4 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
mao@k8s-control-plane-01:~$ 
```

これでクラスターをリセットができました
