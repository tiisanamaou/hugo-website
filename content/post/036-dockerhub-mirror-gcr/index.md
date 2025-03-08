---
title: DockerHubのミラーを設定する
date: 2025-03-08
slug: dockerhub-mirror-gcr
categories:
    - Docker
---

## 理由
2025年4月1日から認証されていないユーザーは、1時間あたり10pullに制限されるみたいなので回避策を試してみる
- https://docs.docker.com/docker-hub/usage/

## 環境
- Ubuntu 24.04.2 LTS
- Docker Engine Community Version 28.0.1

## 設定ファイルを作成する
デフォルトでは設定ファイルがないのでファイルを作成する
- "etc/docker/"に"daemon.json"ファイルがあるか確認し、ない場合は下記の手順で設定ファイルを作成する
```
ls /etc/docker
```

"daemon.json"ファイルを作成する
```
touch daemon.json
nano deamon.json
```

下記を追記する（参考URL）
- https://cloud.google.com/artifact-registry/docs/pull-cached-dockerhub-images?hl=ja
```
{
  "registry-mirrors": ["https://mirror.gcr.io"]
}
```

他の設定もある場合は下記のように","を入れる
- 例
```
{
    "insecure-registries": ["192.168.10.17"],
    "registry-mirrors": ["https://mirror.gcr.io"]
}
```

設定ファイルをコピーしてdockerを再起動する
```
sudo cp -f ./daemon.json /etc/docker/daemon.json
sudo systemctl restart docker
```

次回以降編集する場合または、すでに設定ファイルがある場合は、下記のように設定ファイルを編集してdockerを再起動する
```
sudo nano /etc/docker/daemon.json
sudo systemctl restart docker
```

## 設定が反映されているか確認する
下記コマンドで確認する
```
sudo docker system info
```

- "Registry Mirrors:"の項目に追加されている
```
mao@mao (x86_64) : /home/mao 
> sudo docker system info
Client: Docker Engine - Community
 Version:    28.0.1
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.21.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx

Server:
 Containers: 19
  Running: 0
  Paused: 0
  Stopped: 19
 Images: 6
 Server Version: 28.0.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
 runc version: v1.2.4-0-g6c52b3f
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-55-generic
 Operating System: Ubuntu 24.04.2 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 20
 Total Memory: 62.54GiB
 Name: mao
 ID: ******
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  192.168.10.17
  ::1/128
  127.0.0.0/8
 Registry Mirrors:
  https://mirror.gcr.io/
 Live Restore Enabled: false
```

## 参考URL
- DockerHubのレート制限を受けないようにmirror.gcr.ioを使う
    - https://blog.monophile.net/posts/20201101_docker_mirror_gcr_io.html
- キャッシュに保存された Docker Hub イメージの pull
    - https://cloud.google.com/artifact-registry/docs/pull-cached-dockerhub-images?hl=ja
- daemon.jsonでDockerエンジンのオプション指定 (/etc/docker, insecure-registries)
    - https://devlights.hatenablog.com/entry/2021/11/30/010150
- dockerd daemon
    - https://docs.docker.com/reference/cli/dockerd/#daemon-configuration-file
