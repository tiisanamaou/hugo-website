---
title: TraefikとCloudflareでワイルドカードSSL証明書を発行する
date: 2026-01-30
slug: traefik-cloudflare-ssl
categories:
    - Traefik
    - Cloudflare
---

## VM準備
"qemu-guest-agent"と"nano"をインストールする
```
sudo apt install qemu-guest-agent nano
```

"Docker"をインストールする
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

バージョン確認
```
mao@traefik-server:~$ sudo docker version
Client: Docker Engine - Community
 Version:           29.1.3
 API version:       1.52
 Go version:        go1.25.5
 Git commit:        f52814d
 Built:             Fri Dec 12 14:49:32 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.1.3
  API version:      1.52 (minimum version 1.44)
  Go version:       go1.25.5
  Git commit:       fbf3ed2
  Built:            Fri Dec 12 14:49:32 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.1
  GitCommit:        dea7da592f5d1d2b7755e3a161be07f43fad8f75
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Traefikの設定
設定ファイルの一部をAI(Claude Opus 4.5,Gemini 3 Flash)に生成してもらっています

### ファイル構成
フォルダ構成は下記の通り
```
traefik-docker/
├── .env                  # 環境変数
├── compose-nginx.yaml    # nginxサンプルアプリ設定（個別）
├── compose.yaml          # Traefik本体の設定
├── dynamic.yaml          # Traefikの動的設定
└── traefik.yaml          # Traefikの静的設定
└── acme.json             # 取得したSSL証明書が保存されるファイル
```

### ファイルの中身
- compose.yaml
```
services:
  traefik:
    image: traefik:3.6.5
    container_name: traefik
    restart: unless-stopped
    ports:
      # HTTP
      - "80:80"
      # HTTPS
      - "443:443"
      # ダッシュボード（開発環境のみ）
      - "8080:8080"
    volumes:
      # 静的設定ファイル
      - ./traefik.yaml:/etc/traefik/traefik.yaml:ro
      # 動的設定ファイル
      - ./dynamic.yaml:/etc/traefik/dynamic.yaml:ro
      # Let's Encrypt 証明書保存先
      - ./acme.json:/etc/traefik/acme.json
      # Docker ソケット（Docker プロバイダー使用時）
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # カスタム証明書（任意）
      # - ./certs:/etc/traefik/certs:ro
    networks:
      - traefik-public
    # 環境変数設定（Cloudflare DNS チャレンジ用）
    # .env ファイルから読み込み、明示的に設定
    env_file:
      - .env
    environment:
      CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN}
      TZ: "Asia/Tokyo"
    labels:
      - traefik.enable=true
      - traefik.docker.network=traefik-public
      - traefik.http.services.traefik-dashboard.loadbalancer.server.port=8080
      - traefik.http.routers.traefik-dashboard-http.entrypoints=http
      #- traefik.http.routers.traefik-dashboard-http.rule=Host(`traefik.local`)
    command:
      - "--api.dashboard=true"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.http.address=:80"
      - "--providers.docker.exposedByDefault=false"

networks:
  traefik-public:
    name: traefik-public
```

- .env
```
# ============================================================
# Cloudflare 認証設定
# ============================================================
CF_DNS_API_TOKEN=*****
```

- traefik.yaml\
SSL証明書をステージング用と本番用を切り替えられるように記載している
```
# ============================================================
# Traefik 静的設定ファイル (traefik.yml)
# ============================================================
# このファイルはTraefikの起動時に読み込まれる静的設定です。
# 変更を反映するにはTraefikの再起動が必要です。

# ------------------------------------------------------------
# グローバル設定
# ------------------------------------------------------------
global:
  # 匿名の使用統計を送信するかどうか
  checkNewVersion: true
  # 新バージョンのチェックを行うかどうか
  sendAnonymousUsage: false

# ------------------------------------------------------------
# API / ダッシュボード設定
# ------------------------------------------------------------
api:
  # ダッシュボードを有効にする
  dashboard: true
  # APIを有効にする（本番環境では false 推奨）
  insecure: true
  # デバッグモードを有効にする
  debug: false

# ------------------------------------------------------------
# ログ設定
# ------------------------------------------------------------
log:
  # ログレベル: DEBUG, INFO, WARN, ERROR, FATAL, PANIC
  level: INFO
  # ログ出力先（省略時は標準出力）
  # filePath: "/var/log/traefik/traefik.log"
  # ログフォーマット: common, json
  format: common

# ------------------------------------------------------------
# アクセスログ設定
# ------------------------------------------------------------
accessLog:
  # アクセスログ出力先（省略時は標準出力）
  # filePath: "/var/log/traefik/access.log"
  # ログフォーマット: common, json
  format: common
  # バッファリングサイズ（パフォーマンス向上のため）
  bufferingSize: 100
  # フィルタリング設定
  filters:
    # ステータスコードでフィルタ
    statusCodes:
      - "200-299"
      - "400-499"
      - "500-599"
    # 最小リクエスト時間でフィルタ（ms）
    # minDuration: "10ms"

# ------------------------------------------------------------
# エントリーポイント設定
# ------------------------------------------------------------
# エントリーポイントは外部からのトラフィックを受け付けるポートです
entryPoints:
  # HTTP エントリーポイント
  web:
    address: ":80"
    # HTTPからHTTPSへのリダイレクト設定
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
          permanent: true

  # HTTPS エントリーポイント
  websecure:
    address: ":443"
    http:
      tls:
        # デフォルトの証明書リゾルバー
        certResolver: letsencrypt
        # TLSオプション（任意）
        # options: default

  # カスタムポート例（API用など）
  # api:
  #   address: ":8080"

# ------------------------------------------------------------
# 証明書リゾルバー設定 (Let's Encrypt / ACME + Cloudflare)
# ------------------------------------------------------------
certificatesResolvers:
  # Cloudflare DNS-01 チャレンジ（ワイルドカード証明書対応）
  letsencrypt:
    acme:
      # Let's Encrypt アカウント用メールアドレス
      #email: "your-email@example.com"
      email: ""
      # 証明書の保存先
      storage: "/etc/traefik/acme.json"
      # 本番用エンドポイント（テスト完了後にこちらを使用）
      # caServer: "https://acme-v02.api.letsencrypt.org/directory"
      # テスト用エンドポイント（レート制限回避、最初はこちらで動作確認）
      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      # Cloudflare DNS-01 チャレンジ設定
      dnsChallenge:
        # Cloudflare プロバイダーを使用
        provider: cloudflare
        # DNS レコード伝播の待機時間（秒）
        # Cloudflareは通常速いため短めでOK
        delayBeforeCheck: "10s"
        # 使用するDNSリゾルバー（任意）
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
        # ワイルドカード証明書を取得する場合はこれが必須
        # 環境変数 CF_API_EMAIL, CF_DNS_API_TOKEN が必要
      # ------------------------------------------------------------
      # 証明書の自動更新設定
      # ------------------------------------------------------------
      # 証明書の有効期限チェック間隔（デフォルト: 24時間）
      # 1日1回、証明書の有効期限をチェック
      #certificatesDuration: 2160  # 証明書の有効期限（時間）デフォルト90日

# ------------------------------------------------------------
# プロバイダー設定
# ------------------------------------------------------------
# 動的設定の読み込み元を指定します
providers:
  # ファイルプロバイダー（動的設定ファイル）
  #file:
    # 動的設定ファイルのパス
    #filename: "/etc/traefik/dynamic.yml"
    # または設定ファイルが格納されたディレクトリを指定
    # directory: "/etc/traefik/config/"
    # ファイル変更の監視
    #watch: true

  # Docker プロバイダー
  docker:
    # Docker ソケットへのパス
    endpoint: "unix:///var/run/docker.sock"
    # 明示的にラベル設定されたコンテナのみ公開
    exposedByDefault: false
    # Docker ネットワーク名
    network: traefik-network
    # Swarm モードを有効にする場合
    # swarmMode: true

# ------------------------------------------------------------
# ヘルスチェック / Ping
# ------------------------------------------------------------
ping:
  # ヘルスチェックエンドポイントのエントリーポイント
  entryPoint: web

# ------------------------------------------------------------
# サーバートランスポート設定
# ------------------------------------------------------------
serversTransport:
  # バックエンドへの接続時にTLS証明書を検証しない（開発環境用）
  insecureSkipVerify: false
  # ルート証明書の追加
  # rootCAs:
  #   - "/etc/traefik/certs/ca.crt"
  # 最大アイドル接続数
  maxIdleConnsPerHost: 200

```

- dynamic.yaml
```
# ------------------------------------------------------------
# HTTP 設定
# ------------------------------------------------------------
http:
  # ==========================================================
  # ルーター設定
  # ==========================================================
  # ルーターはリクエストをサービスに振り分けるルールを定義します
  routers:
    # ----------------------------------------------------------
    # 例1: シンプルなWebアプリケーション
    # ----------------------------------------------------------
    webapp:
      # ルールの定義（Host, PathPrefix, Headers などを組み合わせ可能）
      rule: "Host(`*.example.dev`)"
      # 使用するエントリーポイント
      entryPoints:
        - websecure
      # 振り分け先のサービス
      service: webapp-service
      # 適用するミドルウェア（複数指定可能）
      middlewares:
        - security-headers
      # TLS設定
      tls:
        certResolver: letsencrypt
```

- acme.json\
上記ファイルは空ファイルを作成して下記コマンドを実行して権限を付与する\
※SSLの証明書情報はファイルの中に記載される
```
touch acme.json
chmod 600 acme.json
```

- compose-nginx.yaml\
サンプルアプリケーションを想定
```
services:
  nginx:
    image: nginx:1.29.4
    container_name: nginx-sample
    restart: unless-stopped
    networks:
      - traefik-network
    # カスタムHTMLを配置する場合は以下のコメントを解除
    # volumes:
    #   - ./html:/usr/share/nginx/html:ro
    labels:
      # Traefik で公開する
      - "traefik.enable=true"
      # ルーター設定（実際のドメインに変更してください）
      - "traefik.http.routers.nginx.rule=Host(`nginx.example.dev`)"
      - "traefik.http.routers.nginx.entrypoints=websecure"
      - "traefik.http.routers.nginx.tls.certresolver=letsencrypt"
      # ワイルドカード証明書の設定
      - "traefik.http.routers.nginx.tls.domains[0].main=example.dev"
      - "traefik.http.routers.nginx.tls.domains[0].sans=*.example.dev"
      # サービス設定
      - "traefik.http.services.nginx.loadbalancer.server.port=80"
      # ミドルウェア適用
      - "traefik.http.routers.nginx.middlewares=security-headers@file"

networks:
  # 外部ネットワークとしてTraefikのネットワークを参照
  traefik-network:
    external: true
```

### 起動
下記コマンドを実行する
```
sudo docker compose up -d
sudo docker compose down -v
```

Dashboardへは下記のようなURLからアクセスできる
- "http://IP:8080/dashboard/"

## SSL証明書
この時点では"acme.json"に証明書情報は記載されない\
SSL証明書を使用するアプリケーションを追加した際に記載される

コマンドを実行してnginxを起動します
```
docker compose -f compose.nginx.yaml up -d 
```

DNSにドメイン名とIPを登録してhttpsでアクセスできるか確認します\
アクセスできれば問題なく実行できています\
アクセスする際に信頼できない証明書のエラーがでるが、ステージング用の証明書を使用しているので、一旦無視する

## 本番環境のSSL証明書を使用する
"traefik.yaml"に記載されているステージング用をコメントアウトして、本番用のコメントアウトを解除する
- ステージング用
```
# 本番用エンドポイント（テスト完了後にこちらを使用）
# caServer: "https://acme-v02.api.letsencrypt.org/directory"
# テスト用エンドポイント（レート制限回避、最初はこちらで動作確認）
caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
```

- 本番用
```
# 本番用エンドポイント（テスト完了後にこちらを使用）
caServer: "https://acme-v02.api.letsencrypt.org/directory"
# テスト用エンドポイント（レート制限回避、最初はこちらで動作確認）
# caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
```

これで警告が出ることなく接続できます

あとは必要なアプリケーションを実行していけばhttpsで接続できるようになります
