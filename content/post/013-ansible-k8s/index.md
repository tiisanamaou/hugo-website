---
title: Ansibleを使ってKubernetesの初期設定をしてみた
date: 2024-07-27
slug: ansible-k8s
categories:
    - Ansible
    - Kubernetes
---

## 環境
- Ansible 2.16.9
- Kubernetes 1.30.3
- containerd 1.7.20
- runC 1.1.13
- cni-plugins 1.5.1

## ファイル・ディレクトリ構成
```
.
├── ansible.cfg
├── host.yaml
└── playbook_k8s.yaml

1 directory, 3 files
```

ansible.cfg
```
[defaults]
# fingerprintを検証しない設定
host_key_checking = False
```

host.yaml
```
# YAML は ”---" から開始する
---
# "all" グループの宣言
all:
  # "all" グループに含まれるホストに関する情報を定義する宣言
  hosts:
    # 管理対象ノードの情報を定義する宣言
    ansible-test-server: 
      ansible_host: 192.168.10.18
      ansible_user: mao
      ansible_password: mao
      ansible_ssh_private_key_file: /home/mao/ansible-test/ansible-ssh
      ansible_port: 22
```

playbook_k8s.yaml
```
# Ansible-playbook
- name: k8s setup
  hosts:
    - all
  become: yes
  tasks:
##### 1
    - name:
      ansible.builtin.get_url:
        dest: /home/mao/
        url: https://github.com/containerd/containerd/releases/download/v1.7.20/containerd-1.7.20-linux-amd64.tar.gz

    - name: bin containerd
      ansible.builtin.unarchive:
        remote_src: true
        src: /home/mao/containerd-1.7.20-linux-amd64.tar.gz
        dest: /usr/local

    - name:
      ansible.builtin.get_url:
        dest: /home/mao/
        url: https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

    - name:
      ansible.builtin.copy:
        remote_src: true
        src: /home/mao/containerd.service
        dest: /etc/systemd/system/containerd.service

    - name: daemon reload
      ansible.builtin.systemd_service:
        daemon_reload: true

    - name: containerd enable
      ansible.builtin.systemd_service:
        name: containerd
        enabled: true

##### 2
    - name: runC download
      ansible.builtin.get_url:
        dest: /home/mao/
        url: https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64

    - name: runC install
      ansible.builtin.command:
        cmd: install -m 755 runc.amd64 /usr/local/sbin/runc

##### 3
    - name: CNI(Container Network Interface) plugin
      ansible.builtin.get_url:
        dest: /home/mao/
        url: https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz

    - name: make directory
      ansible.builtin.file:
        path: /opt/cni/bin
        state: directory
    
    - name: 
      ansible.builtin.unarchive:
        remote_src: true
        src: /home/mao/cni-plugins-linux-amd64-v1.5.1.tgz
        dest: /opt/cni/bin

##### 4
    - name:
      ansible.builtin.shell:
        cmd: |
          cat > /etc/modules-load.d/k8s.conf << EOF
          overlay
          br_netfilter
          EOF

    - name:
      ansible.builtin.command:
        cmd: modprobe overlay

    - name:
      ansible.builtin.command:
        cmd: modprobe br_netfilter

    - name:
      ansible.builtin.shell:
        cmd: |
          cat > /etc/sysctl.d/k8s.conf << EOF
          net.bridge.bridge-nf-call-iptables  = 1
          net.bridge.bridge-nf-call-ip6tables = 1
          net.ipv4.ip_forward                 = 1
          EOF

    - name:
      ansible.builtin.command:
        cmd: sysctl --system

##### 5
    - name: make directory containerd
      ansible.builtin.file:
        path: /etc/containerd
        state: directory

    - name: cpoy
      ansible.builtin.shell:
        cmd: sudo containerd config default | sudo tee /etc/containerd/config.toml

    - name: 1
      ansible.builtin.lineinfile:
        dest: /etc/containerd/config.toml
        state: present
        backrefs: yes
        regexp: sandbox_image = "registry.k8s.io/pause:3.8"
        line: '    sandbox_image = "registry.k8s.io/pause:3.9"'

    - name: 2
      ansible.builtin.lineinfile:
        dest: /etc/containerd/config.toml
        state: present
        backrefs: yes
        regexp: SystemdCgroup = false
        line: '            SystemdCgroup = true'

    - name: 3
      ansible.builtin.service:
        name: containerd
        state: restarted

##### 6
    - name: install
      ansible.builtin.apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - gpg
        update_cache: yes
        state: present

    - name:
      ansible.builtin.shell:
        cmd: curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    - name:
      ansible.builtin.shell:
        cmd: echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    # kubeadm:1.30.3,kubectl:1.30.3,kubelet:1.30.3
    - name:
      ansible.builtin.apt:
        pkg:
          - kubelet
          - kubeadm
          - kubectl
        update_cache: yes
        state: present
```

## 内容
上記playbookの中にある"#####"は内容ごとに分割しています。
ファイルを分けるのは後々やりたいと思っています。
- 1 containerdのインストール
- 2 runCのインストール
- 3 CNI pluginのインストール
- 4 パラメータの設定
- 5 containerdの追加設定
- 6 kubeadm等のインストール

上記playbookとは別にIPアドレスの固定化、kubernetesクラスタへの参加は手動です。\
こちらも近いうちに自動化できるようにしようと思います。\
そうすれば自動でクラスタが構築できるようになります！

あとはリポジトリの追加等、Ansibleモジュールをちゃんと利用した方法へ置き換えていこうと思います。