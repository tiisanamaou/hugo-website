---
title: Kubernetesのクラスタをv1.30.2からv1.30.3へアップグレードをする
date: 2024-08-13
slug: kubernetes-upgrade-v1-30-3
categories:
    - Kubernetes
---

## 環境
- Kubernetes 1.30.2 → 1.30.3
- Control-Plane 3台
- Woker-Node 2台

## 現状の構成
v1.30.2を使用している
```
mao@k8s-control-plane-01:~$ kubectl get nodes -o wide
NAME                   STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-control-plane-01   Ready    control-plane   42d   v1.30.2   192.168.10.41   <none>        Ubuntu 24.04 LTS   6.8.0-36-generic   containerd://1.7.18
k8s-control-plane-02   Ready    control-plane   42d   v1.30.2   192.168.10.44   <none>        Ubuntu 24.04 LTS   6.8.0-36-generic   containerd://1.7.18
k8s-control-plane-03   Ready    control-plane   42d   v1.30.2   192.168.10.46   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.18
k8s-worker-01          Ready    <none>          42d   v1.30.2   192.168.10.42   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.18
k8s-worker-02          Ready    <none>          42d   v1.30.2   192.168.10.43   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.18
mao@k8s-control-plane-01:~$ 
```

## 手順
公式サイトの手順でアップグレードしていく
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

以下の順番でアップグレードしていく
- Control-Planeをアップグレード
    - Control-Plane-01をアップグレード
    - Control-Plane-02をアップグレード
    - Control-Plane-03をアップグレード
- Woker-Nodeをアップグレード
    - Woker-Node-01をアップグレード
    - Woker-Node-02をアップグレード

## Control-Planeのアップグレード
### node上あるpodを別にnodeへ移動していく
node名を確認する
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.2
k8s-control-plane-02   Ready    control-plane   42d   v1.30.2
k8s-control-plane-03   Ready    control-plane   42d   v1.30.2
k8s-worker-01          Ready    <none>          42d   v1.30.2
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

最初に"k8s-control-plane-01"をアップグレードする
```
kubectl drain --ignore-daemonsets <node name>
kubectl drain --ignore-daemonsets k8s-control-plane-01
```
```
mao@k8s-control-plane-01:~$ kubectl drain --ignore-daemonsets k8s-control-plane-01
node/k8s-control-plane-01 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-dc87d, calico-system/csi-node-driver-8b6ts, kube-system/kube-proxy-9ng8c, metallb-system/speaker-zj5x9
evicting pod kube-system/coredns-7db6d8ff4d-w9f4s
evicting pod calico-apiserver/calico-apiserver-5f78767767-hl7s7
evicting pod kube-system/coredns-7db6d8ff4d-vdlkw
pod/calico-apiserver-5f78767767-hl7s7 evicted
pod/coredns-7db6d8ff4d-vdlkw evicted
pod/coredns-7db6d8ff4d-w9f4s evicted
node/k8s-control-plane-01 drained
mao@k8s-control-plane-01:~$ 
```

"k8s-control-plane-01"の"STATUS"が"SchedulingDisabled"になっているか確認する
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready,SchedulingDisabled   control-plane   42d   v1.30.2
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.2
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.2
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### kubeadm kubelet kubectl をアップデートする
アップデートする対象のバージョンがあるか確認する
```
sudo apt update
sudo apt-cache madison kubeadm
```
"1.30.3"があることを確認
```
mao@k8s-control-plane-01:~$ sudo apt-cache madison kubeadm
   kubeadm | 1.30.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
mao@k8s-control-plane-01:~$ 
```

"kubeadm"を"1.30.3"にアップデートする
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
```
```
mao@k8s-control-plane-01:~$ sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                           
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease           
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubeadm'
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 32 not upgraded.
Need to get 10.4 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.3-1.1 [10.4 MB]
Fetched 10.4 MB in 0s (31.6 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110850 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.30.3-1.1_amd64.deb ...
Unpacking kubeadm (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubeadm (1.30.3-1.1) ...
Scanning processes...                                                                
Scanning candidates...                                                               
Scanning linux images...                                                             

Pending kernel upgrade!
Running kernel version:
  6.8.0-36-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically, so
you should consider rebooting.

Restarting services...

Service restarts being deferred:
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 mao @ session #1: sshd[4320]
 mao @ user manager service: systemd[4324]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
mao@k8s-control-plane-01:~$ 
```

"kubelet"と"kubectl"もアップデートする
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
```
```
mao@k8s-control-plane-01:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
kubelet was already not on hold.
kubectl was already not on hold.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease                    
Hit:3 http://jp.archive.ubuntu.com/ubuntu noble InRelease                    
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubelet'
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubectl'
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 30 not upgraded.
Need to get 28.9 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.3-1.1 [10.8 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.3-1.1 [18.1 MB]
Fetched 28.9 MB in 1s (57.2 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110850 files and directories currently installed.)
Preparing to unpack .../kubectl_1.30.3-1.1_amd64.deb ...
Unpacking kubectl (1.30.3-1.1) over (1.30.2-1.1) ...
Preparing to unpack .../kubelet_1.30.3-1.1_amd64.deb ...
Unpacking kubelet (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubectl (1.30.3-1.1) ...
Setting up kubelet (1.30.3-1.1) ...
Scanning processes...                                                                
Scanning candidates...                                                               
Scanning linux images...                                                             

Pending kernel upgrade!
Running kernel version:
  6.8.0-36-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically, so
you should consider rebooting.

Restarting services...
 systemctl restart kubelet.service

Service restarts being deferred:
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 mao @ session #1: sshd[4320]
 mao @ user manager service: systemd[4324]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
mao@k8s-control-plane-01:~$ 
```

"kubeadm"のバージョンを確認する
```
kubeadm version
```
```
mao@k8s-control-plane-01:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.3", GitCommit:"6fc0a69044f1ac4c13841ec4391224a2df241460", GitTreeState:"clean", BuildDate:"2024-07-16T23:53:15Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}
mao@k8s-control-plane-01:~$ 
```
"v1.30.3"となっていることを確認

アップグレードプランを確認する
```
sudo kubeadm upgrade plan
```
```
mao@k8s-control-plane-01:~$ sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.30.2
[upgrade/versions] kubeadm version: v1.30.3
[upgrade/versions] Target version: v1.30.3
[upgrade/versions] Latest version in the v1.30 series: v1.30.3

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE                   CURRENT   TARGET
kubelet     k8s-control-plane-02   v1.30.2   v1.30.3
kubelet     k8s-control-plane-03   v1.30.2   v1.30.3
kubelet     k8s-worker-01          v1.30.2   v1.30.3
kubelet     k8s-worker-02          v1.30.2   v1.30.3
kubelet     k8s-control-plane-01   v1.30.3   v1.30.3

Upgrade to the latest version in the v1.30 series:

COMPONENT                 NODE                   CURRENT    TARGET
kube-apiserver            k8s-control-plane-01   v1.30.2    v1.30.3
kube-apiserver            k8s-control-plane-02   v1.30.2    v1.30.3
kube-apiserver            k8s-control-plane-03   v1.30.2    v1.30.3
kube-controller-manager   k8s-control-plane-01   v1.30.2    v1.30.3
kube-controller-manager   k8s-control-plane-02   v1.30.2    v1.30.3
kube-controller-manager   k8s-control-plane-03   v1.30.2    v1.30.3
kube-scheduler            k8s-control-plane-01   v1.30.2    v1.30.3
kube-scheduler            k8s-control-plane-02   v1.30.2    v1.30.3
kube-scheduler            k8s-control-plane-03   v1.30.2    v1.30.3
kube-proxy                                       1.30.2     v1.30.3
CoreDNS                                          v1.11.1    v1.11.1
etcd                      k8s-control-plane-01   3.5.12-0   3.5.12-0
etcd                      k8s-control-plane-02   3.5.12-0   3.5.12-0
etcd                      k8s-control-plane-03   3.5.12-0   3.5.12-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.30.3

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

mao@k8s-control-plane-01:~$ 
```

実際にアップグレードする
```
sudo kubeadm upgrade apply v1.30.3
```
```
mao@k8s-control-plane-01:~$ sudo kubeadm upgrade apply v1.30.3
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.30.3"
[upgrade/versions] Cluster version: v1.30.2
[upgrade/versions] kubeadm version: v1.30.3
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.30.3" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Current and new manifests of etcd are equal, skipping upgrade
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests890193377"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-05-10/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-05-10/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-05-10/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config745490494/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[upgrade/addons] skip upgrade addons because control plane instances [k8s-control-plane-02 k8s-control-plane-03] have not been upgraded

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.30.3". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
mao@k8s-control-plane-01:~$ 
```

無事アップグレードされました
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready,SchedulingDisabled   control-plane   42d   v1.30.3
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.2
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.2
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### nodeからpodを退避させているのを解除する
"kubectl drain"を解除する
```
kubectl uncordon <node name>
kubectl uncordon k8s-control-plane-01
```
```
mao@k8s-control-plane-01:~$ kubectl uncordon k8s-control-plane-01
node/k8s-control-plane-01 uncordoned
mao@k8s-control-plane-01:~$ 
```

"STATUS"に"SchedulingDisabled"が表示されなくなったことを確認
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.2
k8s-control-plane-03   Ready    control-plane   42d   v1.30.2
k8s-worker-01          Ready    <none>          42d   v1.30.2
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

## 他のContol-Planeもアップグレードをする
- k8s-control-plane-02
```
mao@k8s-control-plane-02:~$ kubectl drain --ignore-daemonsets k8s-control-plane-0
2
node/k8s-control-plane-02 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-26sbk, calico-system/csi-node-driver-cljz8, kube-system/kube-proxy-xkvj7, metallb-system/speaker-cjz7j
evicting pod calico-system/calico-typha-5579b889c8-kbqk7
evicting pod calico-apiserver/calico-apiserver-5f78767767-89z5t
pod/calico-apiserver-5f78767767-89z5t evicted
pod/calico-typha-5579b889c8-kbqk7 evicted
node/k8s-control-plane-02 drained
mao@k8s-control-plane-02:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready,SchedulingDisabled   control-plane   42d   v1.30.2
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.2
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-02:~$ sudo apt update
[sudo] password for mao: 
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                       
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease             
Get:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 http://jp.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [344 kB]
Get:7 http://jp.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [321 kB]
Fetched 791 kB in 3s (298 kB/s)                          
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
31 packages can be upgraded. Run 'apt list --upgradable' to see them.
mao@k8s-control-plane-02:~$ sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                       
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubeadm'
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 30 not upgraded.
Need to get 10.4 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.3-1.1 [10.4 MB]
Fetched 10.4 MB in 0s (30.5 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110849 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.30.3-1.1_amd64.deb ...
Unpacking kubeadm (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubeadm (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning candidates...                                                           
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-36-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

Restarting services...

Service restarts being deferred:
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
mao@k8s-control-plane-02:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                       
Hit:3 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease            
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubelet'
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubectl'
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 28 not upgraded.
Need to get 28.9 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.3-1.1 [10.8 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.3-1.1 [18.1 MB]
Fetched 28.9 MB in 1s (55.7 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110849 files and directories currently installed.)
Preparing to unpack .../kubectl_1.30.3-1.1_amd64.deb ...
Unpacking kubectl (1.30.3-1.1) over (1.30.2-1.1) ...
Preparing to unpack .../kubelet_1.30.3-1.1_amd64.deb ...
Unpacking kubelet (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubectl (1.30.3-1.1) ...
Setting up kubelet (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning candidates...                                                           
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-36-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

Restarting services...
 systemctl restart kubelet.service

Service restarts being deferred:
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
mao@k8s-control-plane-02:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.3", GitCommit:"6fc0a69044f1ac4c13841ec4391224a2df241460", GitTreeState:"clean", BuildDate:"2024-07-16T23:53:15Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}
mao@k8s-control-plane-02:~$ sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
W0811 23:15:18.916628   16854 compute.go:93] Different API server versions in the cluster were discovered: v1.30.3 on nodes [k8s-control-plane-01], v1.30.2 on nodes [k8s-control-plane-02 k8s-control-plane-03]. Please upgrade your control plane nodes to the same version of Kubernetes
[upgrade/versions] Cluster version: 1.30.3
[upgrade/versions] kubeadm version: v1.30.3
[upgrade/versions] Target version: v1.30.3
[upgrade/versions] Latest version in the v1.30 series: v1.30.3

mao@k8s-control-plane-02:~$ sudo kubeadm upgrade apply v1.30.3
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.30.3"
[upgrade/versions] Cluster version: v1.30.2
[upgrade/versions] kubeadm version: v1.30.3
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.30.3" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-15-44/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests3449985093"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-15-44/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-15-44/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-15-44/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config534750710/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[upgrade/addons] skip upgrade addons because control plane instances [k8s-control-plane-03] have not been upgraded

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.30.3". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
mao@k8s-control-plane-02:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready,SchedulingDisabled   control-plane   42d   v1.30.3
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.2
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-02:~$ kubectl uncordon k8s-control-plane-02
node/k8s-control-plane-02 uncordoned
mao@k8s-control-plane-02:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.2
k8s-worker-01          Ready    <none>          42d   v1.30.2
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-02:~$ 
```

- k8s-control-plane-03
- "sudo kubeadm upgrade plan"と"sudo kubeadm upgrade apply v1.30.3"の変わりに"sudo kubeadm upgrade node"を実行する
```
mao@k8s-control-plane-03:~$ kubectl drain --ignore-daemonsets k8s-control-plane-03
node/k8s-control-plane-03 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-g2ks8, calico-system/csi-node-driver-9w86p, kube-system/kube-proxy-4l8w6, metallb-system/speaker-gg452
evicting pod tigera-operator/tigera-operator-76ff79f7fd-tc9t7
evicting pod calico-system/calico-kube-controllers-5f5665469b-nh6qm
pod/tigera-operator-76ff79f7fd-tc9t7 evicted
pod/calico-kube-controllers-5f5665469b-nh6qm evicted
node/k8s-control-plane-03 drained
mao@k8s-control-plane-03:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-03   Ready,SchedulingDisabled   control-plane   42d   v1.30.2
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-03:~$ sudo apt update
[sudo] password for mao: 
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease   
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Get:3 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB] 
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 http://jp.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [344 kB]
Get:7 http://jp.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [321 kB]
Fetched 791 kB in 2s (466 kB/s)                          
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
33 packages can be upgraded. Run 'apt list --upgradable' to see them.
mao@k8s-control-plane-03:~$ sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease                
Hit:3 http://jp.archive.ubuntu.com/ubuntu noble InRelease                    
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubeadm'
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 32 not upgraded.
Need to get 10.4 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.3-1.1 [10.4 MB]
Fetched 10.4 MB in 0s (31.7 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110850 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.30.3-1.1_amd64.deb ...
Unpacking kubeadm (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubeadm (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
mao@k8s-control-plane-03:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubelet'
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubectl'
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 30 not upgraded.
Need to get 28.9 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.3-1.1 [10.8 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.3-1.1 [18.1 MB]
Fetched 28.9 MB in 0s (58.1 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110850 files and directories currently installed.)
Preparing to unpack .../kubectl_1.30.3-1.1_amd64.deb ...
Unpacking kubectl (1.30.3-1.1) over (1.30.2-1.1) ...
Preparing to unpack .../kubelet_1.30.3-1.1_amd64.deb ...
Unpacking kubelet (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubectl (1.30.3-1.1) ...
Setting up kubelet (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning candidates...                                                           
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

Restarting services...
 systemctl restart kubelet.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
mao@k8s-control-plane-03:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.3", GitCommit:"6fc0a69044f1ac4c13841ec4391224a2df241460", GitTreeState:"clean", BuildDate:"2024-07-16T23:53:15Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}
mao@k8s-control-plane-03:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade] Upgrading your Static Pod-hosted control plane instance to version "v1.30.3"...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-21-58/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests3409919858"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-21-58/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-21-58/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2024-08-11-23-21-58/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 3 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy
[upgrade] The control plane instance for this node was successfully updated!
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config1587591676/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
mao@k8s-control-plane-03:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-03   Ready,SchedulingDisabled   control-plane   42d   v1.30.3
k8s-worker-01          Ready                      <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-03:~$ kubectl uncordon k8s-control-plane-03
node/k8s-control-plane-03 uncordoned
mao@k8s-control-plane-03:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.3
k8s-worker-01          Ready    <none>          42d   v1.30.2
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-03:~$ 
```

## これでControl-Planeのアップグレードは完了

## Worker-Nodeをアップグレードする
### kubeadmをアップデート
"kubeadm"をアップデートする
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
```
```
mao@k8s-worker-01:~$ sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
[sudo] password for mao: 
Canceled hold on kubeadm.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease                
Hit:3 http://jp.archive.ubuntu.com/ubuntu noble InRelease                       
Get:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 http://jp.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [344 kB]
Get:7 http://jp.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [321 kB]
Fetched 791 kB in 2s (365 kB/s)                          
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubeadm'
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 32 not upgraded.
Need to get 10.4 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.3-1.1 [10.4 MB]
Fetched 10.4 MB in 0s (30.4 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110854 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.30.3-1.1_amd64.deb ...
Unpacking kubeadm (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubeadm (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
mao@k8s-worker-01:~$
```

"kubeadm upgrade"を実行する
```
sudo kubeadm upgrade node
```
```
mao@k8s-worker-01:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config1611631804/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
mao@k8s-worker-01:~$ 
```

### Woker-Nodeをドレインする
Control-Planeで下記のコマンドを実行する
```
kubectl drain <node-to-drain> --ignore-daemonsets
kubectl drain k8s-worker-01 --ignore-daemonsets
```
```
mao@k8s-control-plane-01:~$ kubectl drain k8s-worker-01 --ignore-daemonsets
node/k8s-worker-01 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-l2ll2, calico-system/csi-node-driver-2pzn6, kube-system/kube-proxy-rmxrw, metallb-system/speaker-2hccg
evicting pod metallb-system/controller-86f5578878-xxpt6
evicting pod calico-system/calico-typha-5579b889c8-dc9zt
evicting pod calico-apiserver/calico-apiserver-5f78767767-cnx87
evicting pod kube-system/coredns-7db6d8ff4d-m29qd
pod/controller-86f5578878-xxpt6 evicted
pod/calico-apiserver-5f78767767-cnx87 evicted
pod/calico-typha-5579b889c8-dc9zt evicted
pod/coredns-7db6d8ff4d-m29qd evicted
node/k8s-worker-01 drained
mao@k8s-control-plane-01:~$ 
```
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.3
k8s-worker-01          Ready,SchedulingDisabled   <none>          42d   v1.30.2
k8s-worker-02          Ready                      <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### kubeletとkubectlをアップデートする
Worker-Nodeで実行する
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
```
```
mao@k8s-worker-01:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                       
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubelet'
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubectl'
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 30 not upgraded.
Need to get 28.9 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.3-1.1 [10.8 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.3-1.1 [18.1 MB]
Fetched 28.9 MB in 1s (53.4 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110854 files and directories currently installed.)
Preparing to unpack .../kubectl_1.30.3-1.1_amd64.deb ...
Unpacking kubectl (1.30.3-1.1) over (1.30.2-1.1) ...
Preparing to unpack .../kubelet_1.30.3-1.1_amd64.deb ...
Unpacking kubelet (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubectl (1.30.3-1.1) ...
Setting up kubelet (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning candidates...                                                           
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

Restarting services...
 systemctl restart kubelet.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
mao@k8s-worker-01:~$ 
```

リロードする
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### nodeからpodを退避させているのを解除する
"kubectl drain"を解除する\
Control-Planeで実行する
```
kubectl uncordon <node-to-uncordon>
kubectl uncordon k8s-worker-01
```
```
mao@k8s-control-plane-01:~$ kubectl uncordon k8s-worker-01
node/k8s-worker-01 uncordoned
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.3
k8s-worker-01          Ready    <none>          42d   v1.30.3
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### Woker-Nodeのバージョン確認
"kubectl get nodes"をControl-Planeで実行して"k8s-worker-01"のバージョンが"v1.30.3"にアップグレードされていることを確認
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.3
k8s-worker-01          Ready    <none>          42d   v1.30.3
k8s-worker-02          Ready    <none>          42d   v1.30.2
mao@k8s-control-plane-01:~$ 
```

### これでWoker-Nodeのアップグレードは完了

## 他のWoker-Nodeもアップグレードする
- Woker-Node-02
```
mao@k8s-control-plane-01:~$ kubectl drain k8s-worker-02 --ignore-daemonsets
node/k8s-worker-02 cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-28xrv, calico-system/csi-node-driver-jddlc, kube-system/kube-proxy-8hsdx, metallb-system/speaker-dpfpl
evicting pod tigera-operator/tigera-operator-76ff79f7fd-hm5bk
evicting pod calico-system/calico-typha-5579b889c8-pdlqz
evicting pod metallb-system/controller-86f5578878-chzwd
evicting pod kube-system/coredns-7db6d8ff4d-5kfxd
evicting pod calico-apiserver/calico-apiserver-5f78767767-gjh5t
evicting pod calico-system/calico-kube-controllers-5f5665469b-fbfh2
pod/controller-86f5578878-chzwd evicted
pod/tigera-operator-76ff79f7fd-hm5bk evicted
pod/calico-kube-controllers-5f5665469b-fbfh2 evicted
pod/calico-apiserver-5f78767767-gjh5t evicted
pod/coredns-7db6d8ff4d-5kfxd evicted
pod/calico-typha-5579b889c8-pdlqz evicted
node/k8s-worker-02 drained
mao@k8s-control-plane-01:~$ 
```
```
mao@k8s-worker-02:~$ sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.3-*' && \
sudo apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                    
Hit:3 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubeadm'
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 30 not upgraded.
Need to get 10.4 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.3-1.1 [10.4 MB]
Fetched 10.4 MB in 0s (31.1 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110852 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.30.3-1.1_amd64.deb ...
Unpacking kubeadm (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubeadm (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
mao@k8s-worker-02:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config2769406187/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
mao@k8s-worker-02:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.3-*' kubectl='1.30.3-*' && \
sudo apt-mark hold kubelet kubectl
[sudo] password for mao: 
Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Hit:2 http://jp.archive.ubuntu.com/ubuntu noble InRelease                    
Get:3 http://jp.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]   
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease             
Hit:5 http://jp.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 http://jp.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [344 kB]
Get:7 http://jp.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [321 kB]
Fetched 791 kB in 2s (469 kB/s)                          
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubelet'
Selected version '1.30.3-1.1' (isv:kubernetes:core:stable:v1.30:pkgs.k8s.io [amd64]) for 'kubectl'
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 31 not upgraded.
Need to get 28.9 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.3-1.1 [10.8 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.3-1.1 [18.1 MB]
Fetched 28.9 MB in 0s (57.8 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 110852 files and directories currently installed.)
Preparing to unpack .../kubectl_1.30.3-1.1_amd64.deb ...
Unpacking kubectl (1.30.3-1.1) over (1.30.2-1.1) ...
Preparing to unpack .../kubelet_1.30.3-1.1_amd64.deb ...
Unpacking kubelet (1.30.3-1.1) over (1.30.2-1.1) ...
Setting up kubectl (1.30.3-1.1) ...
Setting up kubelet (1.30.3-1.1) ...
Scanning processes...                                                            
Scanning candidates...                                                           
Scanning linux images...                                                         

Pending kernel upgrade!
Running kernel version:
  6.8.0-39-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version
6.8.0-40-generic.

Restarting the system to load the new kernel will not be handled automatically,
so you should consider rebooting.

Restarting services...
 systemctl restart kubelet.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.



mao@k8s-worker-02:~$ sudo systemctl daemon-reload
mao@k8s-worker-02:~$ sudo systemctl restart kubelet
mao@k8s-worker-02:~$ 
```
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS                     ROLES           AGE   VERSION
k8s-control-plane-01   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-02   Ready                      control-plane   42d   v1.30.3
k8s-control-plane-03   Ready                      control-plane   42d   v1.30.3
k8s-worker-01          Ready                      <none>          42d   v1.30.3
k8s-worker-02          Ready,SchedulingDisabled   <none>          42d   v1.30.3
mao@k8s-control-plane-01:~$ kubectl uncordon k8s-worker-02
node/k8s-worker-02 uncordoned
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.3
k8s-worker-01          Ready    <none>          42d   v1.30.3
k8s-worker-02          Ready    <none>          42d   v1.30.3
mao@k8s-control-plane-01:~$ 
```

## これでWoker-Nodeのアップグレードは完了

## クラスタのアップグレードが完了しました
"v1.30.2"が"v1.30.3"へ
```
mao@k8s-control-plane-01:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
k8s-control-plane-01   Ready    control-plane   42d   v1.30.3
k8s-control-plane-02   Ready    control-plane   42d   v1.30.3
k8s-control-plane-03   Ready    control-plane   42d   v1.30.3
k8s-worker-01          Ready    <none>          42d   v1.30.3
k8s-worker-02          Ready    <none>          42d   v1.30.3
mao@k8s-control-plane-01:~$ 
```

podも問題なく動作しているか確認
```
mao@k8s-control-plane-01:~$ kubectl get pod -A -o wide
NAMESPACE          NAME                                           READY   STATUS    RESTARTS       AGE     IP               NODE                   NOMINATED NODE   READINESS GATES
calico-apiserver   calico-apiserver-5f78767767-78qj9              1/1     Running   0              9m1s    10.128.36.245    k8s-worker-01          <none>           <none>
calico-apiserver   calico-apiserver-5f78767767-9q5bf              1/1     Running   0              16m     10.128.204.140   k8s-control-plane-03   <none>           <none>
calico-system      calico-kube-controllers-5f5665469b-qjbzw       1/1     Running   0              9m1s    10.128.36.244    k8s-worker-01          <none>           <none>
calico-system      calico-node-26sbk                              1/1     Running   4 (82m ago)    42d     192.168.10.44    k8s-control-plane-02   <none>           <none>
calico-system      calico-node-28xrv                              1/1     Running   4 (82m ago)    42d     192.168.10.43    k8s-worker-02          <none>           <none>
calico-system      calico-node-dc87d                              1/1     Running   4 (83m ago)    42d     192.168.10.41    k8s-control-plane-01   <none>           <none>
calico-system      calico-node-g2ks8                              1/1     Running   4 (82m ago)    42d     192.168.10.46    k8s-control-plane-03   <none>           <none>
calico-system      calico-node-l2ll2                              1/1     Running   4 (82m ago)    42d     192.168.10.42    k8s-worker-01          <none>           <none>
calico-system      calico-typha-5579b889c8-b4jzx                  1/1     Running   0              12m     192.168.10.42    k8s-worker-01          <none>           <none>
calico-system      calico-typha-5579b889c8-fxdx8                  1/1     Running   0              6m27s   192.168.10.43    k8s-worker-02          <none>           <none>
calico-system      calico-typha-5579b889c8-wrmhq                  1/1     Running   0              25m     192.168.10.41    k8s-control-plane-01   <none>           <none>
calico-system      csi-node-driver-2pzn6                          2/2     Running   8 (82m ago)    42d     10.128.36.239    k8s-worker-01          <none>           <none>
calico-system      csi-node-driver-8b6ts                          2/2     Running   8 (83m ago)    42d     10.128.251.148   k8s-control-plane-01   <none>           <none>
calico-system      csi-node-driver-9w86p                          2/2     Running   8 (82m ago)    42d     10.128.204.138   k8s-control-plane-03   <none>           <none>
calico-system      csi-node-driver-cljz8                          2/2     Running   8 (36h ago)    42d     10.128.194.202   k8s-control-plane-02   <none>           <none>
calico-system      csi-node-driver-jddlc                          2/2     Running   8 (82m ago)    42d     10.128.118.105   k8s-worker-02          <none>           <none>
kube-system        coredns-7db6d8ff4d-cbws9                       1/1     Running   0              9m1s    10.128.36.242    k8s-worker-01          <none>           <none>
kube-system        coredns-7db6d8ff4d-lx728                       1/1     Running   0              16m     10.128.204.139   k8s-control-plane-03   <none>           <none>
kube-system        etcd-k8s-control-plane-01                      1/1     Running   34 (83m ago)   42d     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        etcd-k8s-control-plane-02                      1/1     Running   0              34m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        etcd-k8s-control-plane-03                      1/1     Running   0              27m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-01            1/1     Running   0              44m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-02            1/1     Running   0              33m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-apiserver-k8s-control-plane-03            1/1     Running   0              27m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-01   1/1     Running   0              44m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-02   1/1     Running   0              33m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-controller-manager-k8s-control-plane-03   1/1     Running   0              26m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-proxy-8hsdx                               1/1     Running   0              26m     192.168.10.43    k8s-worker-02          <none>           <none>
kube-system        kube-proxy-ldpnm                               1/1     Running   0              26m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-proxy-qb2c4                               1/1     Running   0              26m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-proxy-rmxrw                               1/1     Running   0              26m     192.168.10.42    k8s-worker-01          <none>           <none>
kube-system        kube-proxy-v6k6r                               1/1     Running   0              26m     192.168.10.46    k8s-control-plane-03   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-01            1/1     Running   0              43m     192.168.10.41    k8s-control-plane-01   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-02            1/1     Running   0              32m     192.168.10.44    k8s-control-plane-02   <none>           <none>
kube-system        kube-scheduler-k8s-control-plane-03            1/1     Running   0              26m     192.168.10.46    k8s-control-plane-03   <none>           <none>
metallb-system     controller-86f5578878-9gqs5                    1/1     Running   0              9m1s    10.128.36.243    k8s-worker-01          <none>           <none>
metallb-system     speaker-2hccg                                  1/1     Running   8 (81m ago)    42d     192.168.10.42    k8s-worker-01          <none>           <none>
metallb-system     speaker-cjz7j                                  1/1     Running   8 (81m ago)    42d     192.168.10.44    k8s-control-plane-02   <none>           <none>
metallb-system     speaker-dpfpl                                  1/1     Running   8 (81m ago)    42d     192.168.10.43    k8s-worker-02          <none>           <none>
metallb-system     speaker-gg452                                  1/1     Running   8 (81m ago)    42d     192.168.10.46    k8s-control-plane-03   <none>           <none>
metallb-system     speaker-zj5x9                                  1/1     Running   8 (81m ago)    42d     192.168.10.41    k8s-control-plane-01   <none>           <none>
tigera-operator    tigera-operator-76ff79f7fd-rq76h               1/1     Running   0              9m1s    192.168.10.42    k8s-worker-01          <none>           <none>
mao@k8s-control-plane-01:~$ 
```

## 参考URL
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/
- https://goodbyegangster.hatenablog.com/entry/2021/01/19/205313

