---
title: cri-oを使用してkubernetesを構築する
date: 2024-08-25
slug: kubernetes-cri-o
categories:
    - Kubernetes
---

## 環境
- Kubernetes 1.31.0
- cri-o v1.30.4

## 参考URL
- https://github.com/cri-o/packaging/blob/main/README.md
- https://github.com/cri-o/cri-o/releases

## cri-oのインストール
リポジトリを追加してのインストールと、バイナリをインストールの2種類がある

### リポジトリからインストールする
```
sudo apt install software-properties-common curl
```

リポジトリの追加
```
CRIO_VERSION=v1.30
```
```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```
```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/deb/Release.key |
    sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/v1.30/deb/ /" |
    sudo tee /etc/apt/sources.list.d/cri-o.list
```

cri-oをインストールする
```
sudo apt install cri-o
sudo systemctl daemon-reload
systemctl enable --now cri-o
systemctl start cri-o
systemctl status cri-o
crio version
```

アンインストール方法
```
which crio
```

### バイナリをインストール
上記のリポジトリを追加してインストールか、このバイナリをインストールする方法のどちらでも問題なく動作する\
バージョンアップはリポジトリの追加のほうがしやすい
```
wget https://storage.googleapis.com/cri-o/artifacts/cri-o.$ARCH.$REV.tar.gz
wget https://storage.googleapis.com/cri-o/artifacts/cri-o.amd64.v1.30.4.tar.gz
```
```
tar -zxvf cri-o.amd64.v1.30.4.tar.gz
cd cri-o
sudo bash ./install
```
```
mao@k8s-crio-woker-node:~/cri-o$ sudo bash ./install
+ DESTDIR=
+ PREFIX=/usr/local
+ ETCDIR=/etc
+ LIBEXECDIR=/usr/libexec
+ LIBEXEC_CRIO_DIR=/usr/libexec/crio
+ ETC_CRIO_DIR=/etc/crio
+ CONTAINERS_DIR=/etc/containers
+ CONTAINERS_REGISTRIES_CONFD_DIR=/etc/containers/registries.conf.d
+ CNIDIR=/etc/cni/net.d
+ BINDIR=/usr/local/bin
+ MANDIR=/usr/local/share/man
+ OCIDIR=/usr/local/share/oci-umount/oci-umount.d
+ BASHINSTALLDIR=/usr/local/share/bash-completion/completions
+ FISHINSTALLDIR=/usr/local/share/fish/completions
+ ZSHINSTALLDIR=/usr/local/share/zsh/site-functions
+ OPT_CNI_BIN_DIR=/opt/cni/bin
+ dpkg -l
+ SYSCONFIGDIR=/etc/default
+ sed -i 's;sysconfig/crio;default/crio;g' etc/crio
+ source /etc/os-release
++ PRETTY_NAME='Ubuntu 24.04 LTS'
++ NAME=Ubuntu
++ VERSION_ID=24.04
++ VERSION='24.04 LTS (Noble Numbat)'
++ VERSION_CODENAME=noble
++ ID=ubuntu
++ ID_LIKE=debian
++ HOME_URL=https://www.ubuntu.com/
++ SUPPORT_URL=https://help.ubuntu.com/
++ BUG_REPORT_URL=https://bugs.launchpad.net/ubuntu/
++ PRIVACY_POLICY_URL=https://www.ubuntu.com/legal/terms-and-policies/privacy-policy
++ UBUNTU_CODENAME=noble
++ LOGO=ubuntu-logo
+ [[ ubuntu == \f\e\d\o\r\a ]]
+ [[ ubuntu == \r\h\c\o\s ]]
+ SYSTEMDDIR=/usr/local/lib/systemd/system
+ SELINUX=
+ selinuxenabled
+ ARCH=amd64
+ install -d -m 755 /etc/cni/net.d
+ install -D -m 755 -t /opt/cni/bin cni-plugins/LICENSE cni-plugins/README.md cni-plugins/bandwidth cni-plugins/bridge cni-plugins/dhcp cni-plugins/dummy cni-plugins/firewall cni-plugins/host-device cni-plugins/host-local cni-plugins/ipvlan cni-plugins/loopback cni-plugins/macvlan cni-plugins/portmap cni-plugins/ptp cni-plugins/sbr cni-plugins/static cni-plugins/tap cni-plugins/tuning cni-plugins/vlan cni-plugins/vrf
+ install -D -m 644 -t /etc/cni/net.d contrib/11-crio-ipv4-bridge.conflist
+ install -d -m 755 /usr/libexec/crio
+ install -D -m 755 -t /usr/libexec/crio bin/conmon
+ install -D -m 755 -t /usr/libexec/crio bin/conmonrs
+ install -D -m 755 -t /usr/libexec/crio bin/crun
+ install -D -m 755 -t /usr/libexec/crio bin/runc
+ install -d -m 755 /usr/local/share/bash-completion/completions
+ install -d -m 755 /usr/local/share/fish/completions
+ install -d -m 755 /usr/local/share/zsh/site-functions
+ install -d -m 755 /etc/containers/registries.conf.d
+ install -D -m 755 -t /usr/local/bin bin/crio
+ install -D -m 755 -t /usr/local/bin bin/pinns
+ install -D -m 755 -t /usr/local/bin bin/crictl
+ install -D -m 644 -t /etc etc/crictl.yaml
+ install -D -m 644 -t /usr/local/share/oci-umount/oci-umount.d etc/crio-umount.conf
+ install -D -m 644 -t /etc/default etc/crio
+ install -D -m 644 -t /etc/crio contrib/policy.json
+ install -D -m 644 -t /etc/crio/crio.conf.d etc/10-crio.conf
+ install -D -m 644 -t /usr/local/share/man/man5 man/crio.conf.5
+ install -D -m 644 -t /usr/local/share/man/man5 man/crio.conf.d.5
+ install -D -m 644 -t /usr/local/share/man/man8 man/crio.8
+ install -D -m 644 -t /usr/local/share/bash-completion/completions completions/bash/crio
+ install -D -m 644 -t /usr/local/share/fish/completions completions/fish/crio.fish
+ install -D -m 644 -t /usr/local/share/zsh/site-functions completions/zsh/_crio
+ install -D -m 644 -t /usr/local/lib/systemd/system contrib/crio.service
+ install -D -m 644 -t /etc/containers/registries.conf.d contrib/registries.conf
+ sed -i 's;/usr/bin;/usr/local/bin;g' /etc/crio/crio.conf.d/10-crio.conf
+ sed -i 's;/usr/libexec;/usr/libexec;g' /etc/crio/crio.conf.d/10-crio.conf
+ sed -i 's;/etc/crio;/etc/crio;g' /etc/crio/crio.conf.d/10-crio.conf
+ '[' -n '' ']'
mao@k8s-crio-woker-node:~/cri-o$ 
```

```
sudo systemctl daemon-reload
sudo systemctl enable --now cri-o
sudo systemctl status cri-o
crio version
```
```
mao@k8s-crio-woker-node:~/cri-o$ sudo systemctl daemon-reload
mao@k8s-crio-woker-node:~/cri-o$ sudo systemctl enable --now crio
Created symlink /etc/systemd/system/cri-o.service → /usr/local/lib/systemd/system/crio.service.
Created symlink /etc/systemd/system/multi-user.target.wants/crio.service → /usr/local/lib/systemd/system/crio.service.
mao@k8s-crio-woker-node:~/cri-o$ crio version
INFO[2024-08-23 23:59:44.088388712Z] Starting CRI-O, version: 1.30.4, git: dbc00ffd41a487c847158032193b6dca9b49e821(clean) 
Version:        1.30.4
GitCommit:      dbc00ffd41a487c847158032193b6dca9b49e821
GitCommitDate:  2024-08-01T06:57:46Z
GitTreeState:   clean
BuildDate:      1970-01-01T00:00:00Z
GoVersion:      go1.22.0
Compiler:       gc
Platform:       linux/amd64
Linkmode:       static
BuildTags:      
  static
  netgo
  osusergo
  exclude_graphdriver_btrfs
  exclude_graphdriver_devicemapper
  seccomp
  apparmor
  selinux
LDFlags:          unknown
SeccompEnabled:   true
AppArmorEnabled:  false

mao@k8s-crio-woker-node:~/cri-o$
```

## nanoをインストール
```
sudo apt install nano
```

## スワップメモリとIPアドレスの設定
スワップをオフにする
```
sudo swapoff -a
sudo nano /etc/fstab
free -h
```

IPアドレスを固定する
```
ip link
ip address
sudo cp 99-config.yaml /etc/netplan/
sudo netplan apply try --timeout 10
sudo netplan apply
sudo chmod 600 /etc/netplan/99-config.yaml
```

## カーネルパラメーター設定
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# この構成に必要なカーネルパラメーター、再起動しても値は永続します
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 再起動せずにカーネルパラメーターを適用
sudo sysctl --system
```
```
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

## Cgroupfsの設定
```
stat -fc %T /sys/fs/cgroup/
```

```
sudo crio config default | sudo tee /etc/crio/crio.conf
```
```
mao@mao-cri-o:~$ sudo crio config default | sudo tee /etc/crio/crio.conf
INFO[2024-08-23 12:32:11.597692695Z] Starting CRI-O, version: 1.30.4, git: dbc00ffd41a487c847158032193b6dca9b49e821(clean) 
INFO Using default capabilities: CAP_CHOWN, CAP_DAC_OVERRIDE, CAP_FSETID, CAP_FOWNER, CAP_SETGID, CAP_SETUID, CAP_SETPCAP, CAP_NET_BIND_SERVICE, CAP_KILL 
# The CRI-O configuration file specifies all of the available configuration
# options and command-line flags for the crio(8) OCI Kubernetes Container Runtime
# daemon, but in a TOML format that can be more easily modified and versioned.
#
# Please refer to crio.conf(5) for details of all configuration options.

# CRI-O supports partial configuration reload during runtime, which can be
# done by sending SIGHUP to the running process. Currently supported options
# are explicitly mentioned with: 'This option supports live configuration
# reload'.

# CRI-O reads its storage defaults from the containers-storage.conf(5) file
# located at /etc/containers/storage.conf. Modify this storage configuration if
# you want to change the system's defaults. If you want to modify storage just
# for CRI-O, you can change the storage configuration options here.
[crio]

# Path to the "root directory". CRI-O stores all of its data, including
# containers images, in this directory.
# root = "/var/lib/containers/storage"

# Path to the "run directory". CRI-O stores all of its state in this directory.
# runroot = "/run/containers/storage"

# Path to the "imagestore". If CRI-O stores all of its images in this directory differently than Root.
# imagestore = ""

# Storage driver used to manage the storage of images and containers. Please
# refer to containers-storage.conf(5) to see all available storage drivers.
# storage_driver = ""

# List to pass options to the storage driver. Please refer to
# containers-storage.conf(5) to see all available storage options.
# storage_option = [
# ]

# The default log directory where all logs will go unless directly specified by
# the kubelet. The log directory specified must be an absolute directory.
# log_dir = "/var/log/crio/pods"

# Location for CRI-O to lay down the temporary version file.
# It is used to check if crio wipe should wipe containers, which should
# always happen on a node reboot
# version_file = "/var/run/crio/version"

# Location for CRI-O to lay down the persistent version file.
# It is used to check if crio wipe should wipe images, which should
# only happen when CRI-O has been upgraded
# version_file_persist = ""

# InternalWipe is whether CRI-O should wipe containers and images after a reboot when the server starts.
# If set to false, one must use the external command 'crio wipe' to wipe the containers and images in these situations.
# internal_wipe = true

# InternalRepair is whether CRI-O should check if the container and image storage was corrupted after a sudden restart.
# If it was, CRI-O also attempts to repair the storage.
# internal_repair = false

# Location for CRI-O to lay down the clean shutdown file.
# It is used to check whether crio had time to sync before shutting down.
# If not found, crio wipe will clear the storage directory.
# clean_shutdown_file = "/var/lib/crio/clean.shutdown"

# The crio.api table contains settings for the kubelet/gRPC interface.
[crio.api]

# Path to AF_LOCAL socket on which CRI-O will listen.
# listen = "/var/run/crio/crio.sock"

# IP address on which the stream server will listen.
# stream_address = "127.0.0.1"

# The port on which the stream server will listen. If the port is set to "0", then
# CRI-O will allocate a random free port number.
# stream_port = "0"

# Enable encrypted TLS transport of the stream server.
# stream_enable_tls = false

# Length of time until open streams terminate due to lack of activity
# stream_idle_timeout = ""

# Path to the x509 certificate file used to serve the encrypted stream. This
# file can change, and CRI-O will automatically pick up the changes within 5
# minutes.
# stream_tls_cert = ""

# Path to the key file used to serve the encrypted stream. This file can
# change and CRI-O will automatically pick up the changes within 5 minutes.
# stream_tls_key = ""

# Path to the x509 CA(s) file used to verify and authenticate client
# communication with the encrypted stream. This file can change and CRI-O will
# automatically pick up the changes within 5 minutes.
# stream_tls_ca = ""

# Maximum grpc send message size in bytes. If not set or <=0, then CRI-O will default to 80 * 1024 * 1024.
# grpc_max_send_msg_size = 83886080

# Maximum grpc receive message size. If not set or <= 0, then CRI-O will default to 80 * 1024 * 1024.
# grpc_max_recv_msg_size = 83886080

# The crio.runtime table contains settings pertaining to the OCI runtime used
# and options for how to set up and manage the OCI runtime.
[crio.runtime]

# A list of ulimits to be set in containers by default, specified as
# "<ulimit name>=<soft limit>:<hard limit>", for example:
# "nofile=1024:2048"
# If nothing is set here, settings will be inherited from the CRI-O daemon
# default_ulimits = [
# ]

# If true, the runtime will not use pivot_root, but instead use MS_MOVE.
# no_pivot = false

# decryption_keys_path is the path where the keys required for
# image decryption are stored. This option supports live configuration reload.
# decryption_keys_path = "/etc/crio/keys/"

# Path to the conmon binary, used for monitoring the OCI runtime.
# Will be searched for using $PATH if empty.
# This option is currently deprecated, and will be replaced with RuntimeHandler.MonitorEnv.
# conmon = ""

# Cgroup setting for conmon
# This option is currently deprecated, and will be replaced with RuntimeHandler.MonitorCgroup.
# conmon_cgroup = ""

# Environment variable list for the conmon process, used for passing necessary
# environment variables to conmon or the runtime.
# This option is currently deprecated, and will be replaced with RuntimeHandler.MonitorEnv.
# conmon_env = [
# ]

# Additional environment variables to set for all the
# containers. These are overridden if set in the
# container image spec or in the container runtime configuration.
# default_env = [
# ]

# If true, SELinux will be used for pod separation on the host.
# This option is deprecated, and be interpreted from whether SELinux is enabled on the host in the future.
# selinux = false

# Path to the seccomp.json profile which is used as the default seccomp profile
# for the runtime. If not specified, then the internal default seccomp profile
# will be used. This option supports live configuration reload.
# seccomp_profile = ""

# Used to change the name of the default AppArmor profile of CRI-O. The default
# profile name is "crio-default". This profile only takes effect if the user
# does not specify a profile via the Kubernetes Pod's metadata annotation. If
# the profile is set to "unconfined", then this equals to disabling AppArmor.
# This option supports live configuration reload.
# apparmor_profile = "crio-default"

# Path to the blockio class configuration file for configuring
# the cgroup blockio controller.
# blockio_config_file = ""

# Reload blockio-config-file and rescan blockio devices in the system before applying
# blockio parameters.
# blockio_reload = false

# Used to change irqbalance service config file path which is used for configuring
# irqbalance daemon.
# irqbalance_config_file = "/etc/sysconfig/irqbalance"

# irqbalance_config_restore_file allows to set a cpu mask CRI-O should
# restore as irqbalance config at startup. Set to empty string to disable this flow entirely.
# By default, CRI-O manages the irqbalance configuration to enable dynamic IRQ pinning.
# irqbalance_config_restore_file = "/etc/sysconfig/orig_irq_banned_cpus"

# Path to the RDT configuration file for configuring the resctrl pseudo-filesystem.
# This option supports live configuration reload.
# rdt_config_file = ""

# Cgroup management implementation used for the runtime.
# cgroup_manager = "systemd"

# Specify whether the image pull must be performed in a separate cgroup.
# separate_pull_cgroup = ""

# List of default capabilities for containers. If it is empty or commented out,
# only the capabilities defined in the containers json file by the user/kube
# will be added.
# default_capabilities = [
#       "CHOWN",
#       "DAC_OVERRIDE",
#       "FSETID",
#       "FOWNER",
#       "SETGID",
#       "SETUID",
#       "SETPCAP",
#       "NET_BIND_SERVICE",
#       "KILL",
# ]

# Add capabilities to the inheritable set, as well as the default group of permitted, bounding and effective.
# If capabilities are expected to work for non-root users, this option should be set.
# add_inheritable_capabilities = false

# List of default sysctls. If it is empty or commented out, only the sysctls
# defined in the container json file by the user/kube will be added.
# default_sysctls = [
# ]

# List of devices on the host that a
# user can specify with the "io.kubernetes.cri-o.Devices" allowed annotation.
# allowed_devices = [
#       "/dev/fuse",
# ]

# List of additional devices. specified as
# "<device-on-host>:<device-on-container>:<permissions>", for example: "--device=/dev/sdc:/dev/xvdc:rwm".
# If it is empty or commented out, only the devices
# defined in the container json file by the user/kube will be added.
# additional_devices = [
# ]

# List of directories to scan for CDI Spec files.
# cdi_spec_dirs = [
#       "/etc/cdi",
#       "/var/run/cdi",
# ]

# Change the default behavior of setting container devices uid/gid from CRI's
# SecurityContext (RunAsUser/RunAsGroup) instead of taking host's uid/gid.
# Defaults to false.
# device_ownership_from_security_context = false

# Path to OCI hooks directories for automatically executed hooks. If one of the
# directories does not exist, then CRI-O will automatically skip them.
# hooks_dir = [
#       "/usr/share/containers/oci/hooks.d",
# ]

# Path to the file specifying the defaults mounts for each container. The
# format of the config is /SRC:/DST, one mount per line. Notice that CRI-O reads
# its default mounts from the following two files:
#
#   1) /etc/containers/mounts.conf (i.e., default_mounts_file): This is the
#      override file, where users can either add in their own default mounts, or
#      override the default mounts shipped with the package.
#
#   2) /usr/share/containers/mounts.conf: This is the default file read for
#      mounts. If you want CRI-O to read from a different, specific mounts file,
#      you can change the default_mounts_file. Note, if this is done, CRI-O will
#      only add mounts it finds in this file.
#
# default_mounts_file = ""

# Maximum number of processes allowed in a container.
# This option is deprecated. The Kubelet flag '--pod-pids-limit' should be used instead.
# pids_limit = -1

# Maximum sized allowed for the container log file. Negative numbers indicate
# that no size limit is imposed. If it is positive, it must be >= 8192 to
# match/exceed conmon's read buffer. The file is truncated and re-opened so the
# limit is never exceeded. This option is deprecated. The Kubelet flag '--container-log-max-size' should be used instead.
# log_size_max = -1

# Whether container output should be logged to journald in addition to the kubernetes log file
# log_to_journald = false

# Path to directory in which container exit files are written to by conmon.
# container_exits_dir = "/var/run/crio/exits"

# Path to directory for container attach sockets.
# container_attach_socket_dir = "/var/run/crio"

# The prefix to use for the source of the bind mounts.
# bind_mount_prefix = ""

# If set to true, all containers will run in read-only mode.
# read_only = false

# Changes the verbosity of the logs based on the level it is set to. Options
# are fatal, panic, error, warn, info, debug and trace. This option supports
# live configuration reload.
# log_level = "info"

# Filter the log messages by the provided regular expression.
# This option supports live configuration reload.
# log_filter = ""

# The UID mappings for the user namespace of each container. A range is
# specified in the form containerUID:HostUID:Size. Multiple ranges must be
# separated by comma.
# This option is deprecated, and will be replaced with Kubernetes user namespace support (KEP-127) in the future.
# uid_mappings = ""

# The GID mappings for the user namespace of each container. A range is
# specified in the form containerGID:HostGID:Size. Multiple ranges must be
# separated by comma.
# This option is deprecated, and will be replaced with Kubernetes user namespace support (KEP-127) in the future.
# gid_mappings = ""

# If set, CRI-O will reject any attempt to map host UIDs below this value
# into user namespaces.  A negative value indicates that no minimum is set,
# so specifying mappings will only be allowed for pods that run as UID 0.
# This option is deprecated, and will be replaced with Kubernetes user namespace support (KEP-127) in the future.
# minimum_mappable_uid = -1

# If set, CRI-O will reject any attempt to map host GIDs below this value
# into user namespaces.  A negative value indicates that no minimum is set,
# so specifying mappings will only be allowed for pods that run as UID 0.
# This option is deprecated, and will be replaced with Kubernetes user namespace support (KEP-127) in the future.
# minimum_mappable_gid = -1

# The minimal amount of time in seconds to wait before issuing a timeout
# regarding the proper termination of the container. The lowest possible
# value is 30s, whereas lower values are not considered by CRI-O.
# ctr_stop_timeout = 30

# drop_infra_ctr determines whether CRI-O drops the infra container
# when a pod does not have a private PID namespace, and does not use
# a kernel separating runtime (like kata).
# It requires manage_ns_lifecycle to be true.
# drop_infra_ctr = true

# infra_ctr_cpuset determines what CPUs will be used to run infra containers.
# You can use linux CPU list format to specify desired CPUs.
# To get better isolation for guaranteed pods, set this parameter to be equal to kubelet reserved-cpus.
# infra_ctr_cpuset = ""

# shared_cpuset  determines the CPU set which is allowed to be shared between guaranteed containers,
# regardless of, and in addition to, the exclusiveness of their CPUs.
# This field is optional and would not be used if not specified.
# You can specify CPUs in the Linux CPU list format.
# shared_cpuset = ""

# The directory where the state of the managed namespaces gets tracked.
# Only used when manage_ns_lifecycle is true.
# namespaces_dir = "/var/run"

# pinns_path is the path to find the pinns binary, which is needed to manage namespace lifecycle
# pinns_path = ""

# Globally enable/disable CRIU support which is necessary to
# checkpoint and restore container or pods (even if CRIU is found in $PATH).
# enable_criu_support = true

# Enable/disable the generation of the container,
# sandbox lifecycle events to be sent to the Kubelet to optimize the PLEG
# enable_pod_events = false

# default_runtime is the _name_ of the OCI runtime to be used as the default.
# The name is matched against the runtimes map below.
default_runtime = "crun"

# A list of paths that, when absent from the host,
# will cause a container creation to fail (as opposed to the current behavior being created as a directory).
# This option is to protect from source locations whose existence as a directory could jeopardize the health of the node, and whose
# creation as a file is not desired either.
# An example is /etc/hostname, which will cause failures on reboot if it's created as a directory, but often doesn't exist because
# the hostname is being managed dynamically.
# absent_mount_sources_to_reject = [
# ]

# The "crio.runtime.runtimes" table defines a list of OCI compatible runtimes.
# The runtime to use is picked based on the runtime handler provided by the CRI.
# If no runtime handler is provided, the "default_runtime" will be used.
# Each entry in the table should follow the format:
#
# [crio.runtime.runtimes.runtime-handler]
# runtime_path = "/path/to/the/executable"
# runtime_type = "oci"
# runtime_root = "/path/to/the/root"
# monitor_path = "/path/to/container/monitor"
# monitor_cgroup = "/cgroup/path"
# monitor_exec_cgroup = "/cgroup/path"
# monitor_env = []
# privileged_without_host_devices = false
# allowed_annotations = []
# platform_runtime_paths = { "os/arch" = "/path/to/binary" }
# Where:
# - runtime-handler: Name used to identify the runtime.
# - runtime_path (optional, string): Absolute path to the runtime executable in
#   the host filesystem. If omitted, the runtime-handler identifier should match
#   the runtime executable name, and the runtime executable should be placed
#   in $PATH.
# - runtime_type (optional, string): Type of runtime, one of: "oci", "vm". If
#   omitted, an "oci" runtime is assumed.
# - runtime_root (optional, string): Root directory for storage of containers
#   state.
# - runtime_config_path (optional, string): the path for the runtime configuration
#   file. This can only be used with when using the VM runtime_type.
# - privileged_without_host_devices (optional, bool): an option for restricting
#   host devices from being passed to privileged containers.
# - allowed_annotations (optional, array of strings): an option for specifying
#   a list of experimental annotations that this runtime handler is allowed to process.
#   The currently recognized values are:
#   "io.kubernetes.cri-o.userns-mode" for configuring a user namespace for the pod.
#   "io.kubernetes.cri-o.cgroup2-mount-hierarchy-rw" for mounting cgroups writably when set to "true".
#   "io.kubernetes.cri-o.Devices" for configuring devices for the pod.
#   "io.kubernetes.cri-o.ShmSize" for configuring the size of /dev/shm.
#   "io.kubernetes.cri-o.UnifiedCgroup.$CTR_NAME" for configuring the cgroup v2 unified block for a container.
#   "io.containers.trace-syscall" for tracing syscalls via the OCI seccomp BPF hook.
#   "io.kubernetes.cri-o.seccompNotifierAction" for enabling the seccomp notifier feature.
#   "io.kubernetes.cri-o.umask" for setting the umask for container init process.
#   "io.kubernetes.cri.rdt-class" for setting the RDT class of a container
#   "seccomp-profile.kubernetes.cri-o.io" for setting the seccomp profile for:
#     - a specific container by using: "seccomp-profile.kubernetes.cri-o.io/<CONTAINER_NAME>"
#     - a whole pod by using: "seccomp-profile.kubernetes.cri-o.io/POD"
#     Note that the annotation works on containers as well as on images.
#     For images, the plain annotation "seccomp-profile.kubernetes.cri-o.io"
#     can be used without the required "/POD" suffix or a container name.
#   "io.kubernetes.cri-o.DisableFIPS" for disabling FIPS mode in a Kubernetes pod within a FIPS-enabled cluster.
# - monitor_path (optional, string): The path of the monitor binary. Replaces
#   deprecated option "conmon".
# - monitor_cgroup (optional, string): The cgroup the container monitor process will be put in.
#   Replaces deprecated option "conmon_cgroup".
# - monitor_exec_cgroup (optional, string): If set to "container", indicates exec probes
#   should be moved to the container's cgroup
# - monitor_env (optional, array of strings): Environment variables to pass to the montior.
#   Replaces deprecated option "conmon_env".
# - platform_runtime_paths (optional, map): A mapping of platforms to the corresponding
#   runtime executable paths for the runtime handler.
# - container_min_memory (optional, string): The minimum memory that must be set for a container.
#   This value can be used to override the currently set global value for a specific runtime. If not set,
#   a global default value of "12 MiB" will be used.
#
# Using the seccomp notifier feature:
#
# This feature can help you to debug seccomp related issues, for example if
# blocked syscalls (permission denied errors) have negative impact on the workload.
#
# To be able to use this feature, configure a runtime which has the annotation
# "io.kubernetes.cri-o.seccompNotifierAction" in the allowed_annotations array.
#
# It also requires at least runc 1.1.0 or crun 0.19 which support the notifier
# feature.
#
# If everything is setup, CRI-O will modify chosen seccomp profiles for
# containers if the annotation "io.kubernetes.cri-o.seccompNotifierAction" is
# set on the Pod sandbox. CRI-O will then get notified if a container is using
# a blocked syscall and then terminate the workload after a timeout of 5
# seconds if the value of "io.kubernetes.cri-o.seccompNotifierAction=stop".
#
# This also means that multiple syscalls can be captured during that period,
# while the timeout will get reset once a new syscall has been discovered.
#
# This also means that the Pods "restartPolicy" has to be set to "Never",
# otherwise the kubelet will restart the container immediately.
#
# Please be aware that CRI-O is not able to get notified if a syscall gets
# blocked based on the seccomp defaultAction, which is a general runtime
# limitation.


[crio.runtime.runtimes.crun]
runtime_path = "/usr/libexec/crio/crun"
runtime_type = ""
runtime_root = "/run/crun"
runtime_config_path = ""
container_min_memory = ""
monitor_path = "/usr/libexec/crio/conmon"
monitor_cgroup = "system.slice"
monitor_exec_cgroup = ""

allowed_annotations = [
        "io.containers.trace-syscall",
]
privileged_without_host_devices = false


[crio.runtime.runtimes.runc]
runtime_path = "/usr/libexec/crio/runc"
runtime_type = ""
runtime_root = "/run/runc"
runtime_config_path = ""
container_min_memory = ""
monitor_path = "/usr/libexec/crio/conmon"
monitor_cgroup = "system.slice"
monitor_exec_cgroup = ""


privileged_without_host_devices = false


# The workloads table defines ways to customize containers with different resources
# that work based on annotations, rather than the CRI.
# Note, the behavior of this table is EXPERIMENTAL and may change at any time.
# Each workload, has a name, activation_annotation, annotation_prefix and set of resources it supports mutating.
# The currently supported resources are "cpuperiod" "cpuquota", "cpushares", "cpulimit" and "cpuset". The values for "cpuperiod" and "cpuquota" are denoted in microseconds.
# The value for "cpulimit" is denoted in millicores, this value is used to calculate the "cpuquota" with the supplied "cpuperiod" or the default "cpuperiod".
# Note that the "cpulimit" field overrides the "cpuquota" value supplied in this configuration.
# Each resource can have a default value specified, or be empty.
# For a container to opt-into this workload, the pod should be configured with the annotation $activation_annotation (key only, value is ignored).
# To customize per-container, an annotation of the form $annotation_prefix.$resource/$ctrName = "value" can be specified
# signifying for that resource type to override the default value.
# If the annotation_prefix is not present, every container in the pod will be given the default values.
# Example:
# [crio.runtime.workloads.workload-type]
# activation_annotation = "io.crio/workload"
# annotation_prefix = "io.crio.workload-type"
# [crio.runtime.workloads.workload-type.resources]
# cpuset = "0-1"
# cpushares = "5"
# cpuquota = "1000"
# cpuperiod = "100000"
# cpulimit = "35"
# Where:
# The workload name is workload-type.
# To specify, the pod must have the "io.crio.workload" annotation (this is a precise string match).
# This workload supports setting cpuset and cpu resources.
# annotation_prefix is used to customize the different resources.
# To configure the cpu shares a container gets in the example above, the pod would have to have the following annotation:
# "io.crio.workload-type/$container_name = {"cpushares": "value"}"

# hostnetwork_disable_selinux determines whether
# SELinux should be disabled within a pod when it is running in the host network namespace
# Default value is set to true
# hostnetwork_disable_selinux = true

# disable_hostport_mapping determines whether to enable/disable
# the container hostport mapping in CRI-O.
# Default value is set to 'false'
# disable_hostport_mapping = false

# timezone To set the timezone for a container in CRI-O.
# If an empty string is provided, CRI-O retains its default behavior. Use 'Local' to match the timezone of the host machine.
# timezone = ""

# The crio.image table contains settings pertaining to the management of OCI images.
#
# CRI-O reads its configured registries defaults from the system wide
# containers-registries.conf(5) located in /etc/containers/registries.conf. If
# you want to modify just CRI-O, you can change the registries configuration in
# this file. Otherwise, leave insecure_registries and registries commented out to
# use the system's defaults from /etc/containers/registries.conf.
[crio.image]

# Default transport for pulling images from a remote container storage.
# default_transport = "docker://"

# The path to a file containing credentials necessary for pulling images from
# secure registries. The file is similar to that of /var/lib/kubelet/config.json
# global_auth_file = ""

# The image used to instantiate infra containers.
# This option supports live configuration reload.
# pause_image = "registry.k8s.io/pause:3.9"

# The path to a file containing credentials specific for pulling the pause_image from
# above. The file is similar to that of /var/lib/kubelet/config.json
# This option supports live configuration reload.
# pause_image_auth_file = ""

# The command to run to have a container stay in the paused state.
# When explicitly set to "", it will fallback to the entrypoint and command
# specified in the pause image. When commented out, it will fallback to the
# default: "/pause". This option supports live configuration reload.
# pause_command = "/pause"

# List of images to be excluded from the kubelet's garbage collection.
# It allows specifying image names using either exact, glob, or keyword
# patterns. Exact matches must match the entire name, glob matches can
# have a wildcard * at the end, and keyword matches can have wildcards
# on both ends. By default, this list includes the "pause" image if
# configured by the user, which is used as a placeholder in Kubernetes pods.
# pinned_images = [
# ]

# Path to the file which decides what sort of policy we use when deciding
# whether or not to trust an image that we've pulled. It is not recommended that
# this option be used, as the default behavior of using the system-wide default
# policy (i.e., /etc/containers/policy.json) is most often preferred. Please
# refer to containers-policy.json(5) for more details.
signature_policy = "/etc/crio/policy.json"

# Root path for pod namespace-separated signature policies.
# The final policy to be used on image pull will be <SIGNATURE_POLICY_DIR>/<NAMESPACE>.json.
# If no pod namespace is being provided on image pull (via the sandbox config),
# or the concatenated path is non existent, then the signature_policy or system
# wide policy will be used as fallback. Must be an absolute path.
# signature_policy_dir = "/etc/crio/policies"

# List of registries to skip TLS verification for pulling images. Please
# consider configuring the registries via /etc/containers/registries.conf before
# changing them here.
# insecure_registries = [
# ]

# Controls how image volumes are handled. The valid values are mkdir, bind and
# ignore; the latter will ignore volumes entirely.
# image_volumes = "mkdir"

# Temporary directory to use for storing big files
# big_files_temporary_dir = ""

# If true, CRI-O will automatically reload the mirror registry when
# there is an update to the 'registries.conf.d' directory. Default value is set to 'false'.
# auto_reload_registries = false

# The crio.network table containers settings pertaining to the management of
# CNI plugins.
[crio.network]

# The default CNI network name to be selected. If not set or "", then
# CRI-O will pick-up the first one found in network_dir.
# cni_default_network = ""

# Path to the directory where CNI configuration files are located.
# network_dir = "/etc/cni/net.d/"

# Paths to directories where CNI plugin binaries are located.
# plugin_dirs = [
#       "/opt/cni/bin/",
# ]

# List of included pod metrics.
# included_pod_metrics = [
# ]

# A necessary configuration for Prometheus based metrics retrieval
[crio.metrics]

# Globally enable or disable metrics support.
# enable_metrics = false

# Specify enabled metrics collectors.
# Per default all metrics are enabled.
# It is possible, to prefix the metrics with "container_runtime_" and "crio_".
# For example, the metrics collector "operations" would be treated in the same
# way as "crio_operations" and "container_runtime_crio_operations".
# metrics_collectors = [
#       "image_pulls_layer_size",
#       "containers_events_dropped_total",
#       "containers_oom_total",
#       "processes_defunct",
#       "operations_total",
#       "operations_latency_seconds",
#       "operations_latency_seconds_total",
#       "operations_errors_total",
#       "image_pulls_bytes_total",
#       "image_pulls_skipped_bytes_total",
#       "image_pulls_failure_total",
#       "image_pulls_success_total",
#       "image_layer_reuse_total",
#       "containers_oom_count_total",
#       "containers_seccomp_notifier_count_total",
#       "resources_stalled_at_stage",
# ]
# The IP address or hostname on which the metrics server will listen.
# metrics_host = "127.0.0.1"

# The port on which the metrics server will listen.
# metrics_port = 9090

# Local socket path to bind the metrics server to
# metrics_socket = ""

# The certificate for the secure metrics server.
# If the certificate is not available on disk, then CRI-O will generate a
# self-signed one. CRI-O also watches for changes of this path and reloads the
# certificate on any modification event.
# metrics_cert = ""

# The certificate key for the secure metrics server.
# Behaves in the same way as the metrics_cert.
# metrics_key = ""

# A necessary configuration for OpenTelemetry trace data exporting
[crio.tracing]

# Globally enable or disable exporting OpenTelemetry traces.
# enable_tracing = false

# Address on which the gRPC trace collector listens on.
# tracing_endpoint = "0.0.0.0:4317"

# Number of samples to collect per million spans. Set to 1000000 to always sample.
# tracing_sampling_rate_per_million = 0

# CRI-O NRI configuration.
[crio.nri]

# Globally enable or disable NRI.
# enable_nri = true

# NRI socket to listen on.
# nri_listen = "/var/run/nri/nri.sock"

# NRI plugin directory to use.
# nri_plugin_dir = "/opt/nri/plugins"

# NRI plugin configuration directory to use.
# nri_plugin_config_dir = "/etc/nri/conf.d"

# Disable connections from externally launched NRI plugins.
# nri_disable_connections = false

# Timeout for a plugin to register itself with NRI.
# nri_plugin_registration_timeout = "5s"

# Timeout for a plugin to handle an NRI request.
# nri_plugin_request_timeout = "2s"

# Necessary information pertaining to container and pod stats reporting.
[crio.stats]

# The number of seconds between collecting pod and container stats.
# If set to 0, the stats are collected on-demand instead.
# stats_collection_period = 0

# The number of seconds between collecting pod/container stats and pod
# sandbox metrics. If set to 0, the metrics/stats are collected on-demand instead.
# collection_period = 0

mao@mao-cri-o:~$ 
```

cri-oの設定ファイルに追記する
```
sudo nano /etc/crio/crio.conf
```

- 追記する1
```
[crio.runtime]
conmon_cgroup = "pod"
cgroup_manager = "cgroupfs"
```

- 追記する2
```
[crio.image]
pause_image="registry.k8s.io/pause:3.6"
```
```
[crio.image]
- # pause_image = "registry.k8s.io/pause:3.9"
+ pause_image = "registry.k8s.io/pause:3.9"
```
```
- #default_runtime = "crun"
+ default_runtime = "runc"
```

設定を反映するためにリロードする
```
sudo systemctl restart cri-o
```

## runCとCNIのインストール
- runC
```
sudo wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

- CNI
```
sudo wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.1.tgz
```

## "kubelet","kubeadm","kubectl"のインストール
リポジトリを追加する
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

インストールをして、バージョンを固定する
```
sudo apt update
sudo apt install kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo apt-mark showhold
```

## Control-Planeでの作業1
クラスタの初期設定をする
```
sudo kubeadm init --apiserver-advertise-address=192.168.10.55 --pod-network-cidr=10.128.0.0/16
```
```
mao@mao-cri-o:~$ sudo kubeadm init --apiserver-advertise-address=192.168.10.55 --pod-network-cidr=10.128.0.0/16
[init] Using Kubernetes version: v1.31.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0823 13:05:00.855747   12076 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local mao-cri-o] and IPs [10.96.0.1 192.168.10.55]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost mao-cri-o] and IPs [192.168.10.55 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost mao-cri-o] and IPs [192.168.10.55 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.386574ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 5.501313388s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node mao-cri-o as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node mao-cri-o as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: chwroq.m0rdyxckr4zb0qpb
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.55:6443 --token chwroq.m0rdyxckr4zb0qpb \
        --discovery-token-ca-cert-hash sha256:e2abcd3797f4c417d228e4ffeb65f2215498933d2057d511701881b49625e629 
mao@mao-cri-o:~$ 
```

- joinコマンドの再表示
```
sudo kubeadm token create --print-join-command
```

## Control-Planeでの作業2
Calicoをインストールする
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/tigera-operator.yaml
kubectl create -f tigera-operator.yaml
```
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/custom-resources.yaml
```
```
- cidr: 192.168.0.0/16
+ cidr: 10.128.0.0/16
```
```
kubectl apply -f custom-resources.yaml
```

## Woker-Nodeでの作業
クラスタにjoinするコマンドを実行する
```
sudo kubeadm join 192.168.10.55:6443 --token chwroq.m0rdyxckr4zb0qpb \
        --discovery-token-ca-cert-hash sha256:e2abcd3797f4c417d228e4ffeb65f2215498933d2057d511701881b49625e629
```

## 確認
クラスタにjoinされていることを確認する
```
kubectl get nodes -o wide
```
```
mao@mao-cri-o:~$ kubectl get nodes -o wide
NAME                    STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
mao-cri-o               Ready    control-plane   35m   v1.31.0   192.168.10.55   <none>        Ubuntu 24.04 LTS   6.8.0-41-generic   cri-o://1.30.4
mao-cri-o-worker-node   Ready    <none>          32s   v1.31.0   192.168.10.56   <none>        Ubuntu 24.04 LTS   6.8.0-41-generic   cri-o://1.30.4
mao@mao-cri-o:~$
```

## コンテナランタイムが混ぜる
containerdのクラスタにcri-oのNodeをjoinしようとしたらエラーになりコンテナランタイムを混ぜての構築はできなかった
