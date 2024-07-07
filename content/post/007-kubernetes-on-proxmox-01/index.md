---
title: kubernetesをproxmox上に立ててみた（1）
date: 2024-07-03
slug: kubernetes-on-proxmox-01
categories:
    - Kubernetes
    - Proxmox
---

## 
k8sを勉強してみようと思い、デプロイ等の操作は書籍で触っていたら構築もしてみたくなったので、Proxmox上に仮想マシンを作成してk8sを構築してみた

## 開発環境
- Proxmox 8.2.4
- Ubuntu Server 24.04 LTS

## 構成
Proxmox上に以下6つの仮想マシンを立てました
- haproxy-01
- control-plane-node-01
- control-plane-node-02
- control-plane-node-03
- worker-node-01
- worker-node-02

## Control-Plane（Master-Node）とWorker-Node両方で実行する
### setup
パッケージの更新をする
```
sudo apt update
sudo apt upgrade
```

必要なソフトウェアをインストールする
```
sudo apt install nano
```

公式の手順にそって実行する
- https://kubernetes.io/ja/docs/setup/production-environment/

### Swapをオフにする
swapを止めます
```
sudo swapoff -a
```

設定ファイルを書き換えて永続的にswapをオフにする
```
sudo nano /etc/fstab
```
編集内容
```
- /swap.img      none    swap    sw      0       0
+ #/swap.img      none    swap    sw      0       0
```

swapがオフになっているか確認する
```
free -h
```
実行結果
```
mao@k8s-control-plane-01:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       510Mi       7.2Gi       704Ki       248Mi       7.3Gi
Swap:             0B          0B          0B
```

### IPアドレスを固定IPアドレスにする
ネットワークのデバイスを確認します
```
ip address
```
実行結果
```
mao@k8s-control-plane-01:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether bc:24:11:7f:08:e4 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    inet 192.168.10.10/24 metric 100 brd 192.168.10.255 scope global dynamic ens18
       valid_lft 85953sec preferred_lft 85953sec
    inet6 fe80::be24:11ff:fe7f:8e4/64 scope link 
       valid_lft forever preferred_lft forever
mao@k8s-control-plane-01:~$ 
```

netplanファイルを作成します
- ファイル名：99-config.yaml
```
network:
    version: 2
    renderer: networkd
    ethernets:
      ens18:
        dhcp4: false
        addresses:
          - 192.168.10.41/24
        routes:
          - to: default
            via: 192.168.10.1
        nameservers:
            search: []
            addresses: [192.168.10.1]
```

netplanファイルをコピーして適用します
- ssh等で接続している場合は、IPアドレスが変わるので接続が切れます、\
再度固定にしたIPアドレスに変更すれば接続できます
```
sudo cp 99-config.yaml /etc/netplan/
sudo netplan apply
sudo chmod 600 /etc/netplan/99-config.yaml
```

### containerdをインストールする
公式手順に従ってインストール
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md
- Option 1: From the official binaries
- containerd 1.7.18

順にコマンドを実行する
```
wget https://github.com/containerd/containerd/releases/download/v1.7.18/containerd-1.7.18-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.18-linux-amd64.tar.gz
```
```
sudo wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -O /etc/systemd/system/containerd.service
```

systemctlをリロードし、containerdを有効にする
```
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

### runCをインストールする
```
sudo wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
```
```
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

### CNI(Container Network Interface) pluginをインストールする
```
sudo wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
```
```
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.1.tgz
```

### IPv4フォワーディングの設定をする
以下のコマンドを順に実行する

#### 1
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
```

実行結果
```
mao@k8s-control-plane-01:~$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
> overlay
> br_netfilter
> EOF
overlay
br_netfilter
mao@k8s-control-plane-01:~$ 
```
```
sudo modprobe overlay
sudo modprobe br_netfilter
```

#### 2
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
```
実行結果
```
mao@k8s-control-plane-01:~$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-iptables  = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> net.ipv4.ip_forward                 = 1
> EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
mao@k8s-control-plane-01:~$ 
```
```
sudo sysctl --system
```

#### 3
```
lsmod | grep br_netfilter
lsmod | grep overlay
```
実行結果
```
mao@k8s-control-plane-01:~$ lsmod | grep br_netfilter
br_netfilter           32768  0
bridge                421888  1 br_netfilter
mao@k8s-control-plane-01:~$ lsmod | grep overlay
overlay               212992  0
mao@k8s-control-plane-01:~$ 
```

#### 4
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```
実行結果
```
mao@k8s-control-plane-01:~$ sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
mao@k8s-control-plane-01:~$ 
```

### systemd cgroup の設定をする
参考URL
- https://kubernetes.io/ja/docs/concepts/architecture/cgroups/
- https://kubernetes.io/ja/docs/concepts/architecture/cgroups/#check-cgroup-version
- https://sogo.dev/posts/2022/12/kubernetes-ubuntu22.04-cgroup-systemd

```
stat -fc %T /sys/fs/cgroup/
```
- cgroup v2では、"cgroup2fs"と出力されます。
- cgroup v1では、"tmpfs"と出力されます。


ディレクトリを作成する
```
sudo mkdir /etc/containerd
```

以下のコマンドで、デフォルトのコンフィグを作成できます。
```
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
実行結果
```
mao@k8s-control-plane-01:~$ sudo containerd config default | sudo tee /etc/containerd/config.toml
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    drain_exec_sync_io_timeout = "0s"
    enable_cdi = false
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_deprecation_warnings = []
    ignore_image_defined_volumes = false
    image_pull_progress_timeout = "5m0s"
    image_pull_with_sync_fs = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.8"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
      setup_serially = false

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_blockio_not_enabled_errors = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          privileged_without_host_devices_all_devices_allowed = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          snapshotter = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = false

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.nri.v1.nri"]
    disable = true
    disable_connections = false
    plugin_config_path = "/etc/nri/conf.d"
    plugin_path = "/opt/nri/plugins"
    plugin_registration_timeout = "5s"
    plugin_request_timeout = "2s"
    socket_path = "/var/run/nri/nri.sock"

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    blockio_config_file = ""
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.blockfile"]
    fs_type = ""
    mount_options = []
    root_path = ""
    scratch_file = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]

  [plugins."io.containerd.transfer.v1.local"]
    config_path = ""
    max_concurrent_downloads = 3
    max_concurrent_uploaded_layers = 3

    [[plugins."io.containerd.transfer.v1.local".unpack_config]]
      differ = ""
      platform = "linux/amd64"
      snapshotter = "overlayfs"

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
mao@k8s-control-plane-01:~$ 
```

設定ファイルを編集する
```
sudo nano /etc/containerd/config.toml
```
以下の2箇所を編集する
```
  [plugins."io.containerd.grpc.v1.cri"]
-   sandbox_image = "registry.k8s.io/pause:3.6"
+   sandbox_image = "registry.k8s.io/pause:3.9"
...
           [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
-            SystemdCgroup = false
+            SystemdCgroup = true
```

containerdを再起動する
```
sudo systemctl restart containerd
```

### kubeadm/kubelet/kubectl をインストールする
参考URL
- https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

以下のコマンドを順番に実行する
```
sudo apt install apt-transport-https ca-certificates curl gpg
```
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

一度アップデートした後にインストールする
```
sudo apt update
sudo apt install kubelet kubeadm kubectl
```
バージョンを固定する
```
sudo apt-mark hold kubelet kubeadm kubectl
```
実行結果
```
mao@k8s-control-plane-01:~$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
mao@k8s-control-plane-01:~$ 
```

バージョン固定の解除コマンド
```
sudo apt-mark showhold
sudo apt-mark unhold <パッケージ名>
```
