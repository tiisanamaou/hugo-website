---
title: 証明書発行手順（Let's Encrypt）
date: 2024-12-17
slug: certbot-lets-encrypt
categories:
    - Let's-Encrypt
    - certbot
    - SSL
---

## 背景
ローカルの開発環境をHTTPSでアクセスしたいが、自己署名証明書だと警告が出たりと問題があるので、正規の証明書を発行して使用する

## 証明書の発行準備
証明書を発行する用のVMを作ったのでsshで接続する
```
ssh mao@192.168.10.11
```

certbotをインストールする
```
sudo apt install certbot 
```

オプションを確認するためにヘルプを参照する
```
certbot help all
```

下記は今回必要なオプションを抜粋して記載（実際は全て表示される）
```
mao@lets-encrypt-certbot:~$ certbot help all
usage: 
  certbot [SUBCOMMAND] [options] [-d DOMAIN] [-d DOMAIN] ...

Certbot can obtain and install HTTPS/TLS/SSL certificates.  By default,
it will attempt to use a webserver both for obtaining and installing the
certificate. The most common SUBCOMMANDS and flags are:

obtain, install, and renew certificates:
    certonly        Obtain or renew a certificate, but do not install it
   -d DOMAINS       Comma-separated list of domains to obtain a certificate for

  --manual          Obtain certificates interactively, or using shell script hooks

options:
  -h, --help            show this help message and exit
  -d DOMAIN, --domains DOMAIN, --domain DOMAIN
                        Domain names to include. For multiple domains you can use multiple -d flags or enter a comma separated list of domains as a
                        parameter. All domains will be included as Subject Alternative Names on the certificate. The first domain will be used as the
                        certificate name, unless otherwise specified or if you already have a certificate with the same name. In the case of a name
                        conflict, a number like -0001 will be appended to the certificate name. (default: Ask)

register:
  Options for account registration

  --register-unsafely-without-email
                        Specifying this flag enables registering an account with no email address. This is strongly discouraged, because you will be unable
                        to receive notice about impending expiration or revocation of your certificates or problems with your Certbot installation that
                        will lead to failure to renew. (default: False)
  -m EMAIL, --email EMAIL
                        Email used for registration and recovery contact. Use comma to register multiple emails, ex: u1@example.com,u2@example.com.
                        (default: Ask).
```

## SSL証明書を発行する
実行するコマンドは下記の通り
```
certbot certonly
--manual
--server https://acme-v02.api.letsencrypt.org/directory
--preferred-challenges dns
-d internal.onodera.dev
--register-unsafely-without-email
```

オプションの説明（発行するドメインに置き換える）
- "certonly"
  - 証明書の発行のみをする
- "--manual"
  - マニュアルで指定する
- "--server https://acme-v02.api.letsencrypt.org/directory"
  - 証明書の発行をするサーバーを指定する
- "--preferred-challenges dns"
  - チャレンジ方法をDNSにする
- "-d *.internal.onodera.dev"
  - 証明書を発行するドメインを指定する
  - 今回はサブドメインで証明書を発行する
- "--register-unsafely-without-email"
  - メールアドレスを登録せずに証明書を発行する

### コマンド実行
```
sudo certbot certonly --manual --server https://acme-v02.api.letsencrypt.org/directory --preferred-challenges dns -d internal.onodera-program.com --register-unsafely-without-email
```

### 実行結果
```
mao@lets-encrypt-certbot:~$ sudo certbot certonly --manual --server https://acme-v02.api.letsencrypt.org/directory --preferred-challenges dns -d internal.onodera-program.com --register-unsafely-without-email
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
Requesting a certificate for internal.onodera-program.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

_acme-challenge.internal.onodera-program.com.

with the following value:

***1b5eLnL15fqIdYkuSaqRu1CPGmgq6FTgnEHbZVTs

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.internal.onodera-program.com.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/internal.onodera-program.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/internal.onodera-program.com/privkey.pem
This certificate expires on 2025-03-08.
These files will be updated when the certificate renews.

NEXT STEPS:
- This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
mao@lets-encrypt-certbot:~$ 
```

### 発行手順
- 利用規約を読み同意するか求められるので、"Y"を押す
- DNSのTXTレコードに値を登録するよう求められるので、DNSに値を登録する
- DNSにTXTレコードが反映されているかdigコマンド等で確認する
- TXTレコードが反映されていたら、Enterを押す
- 証明書が発行される

#### TXTレコードの確認方法
下記コマンドを実行する（ドメイン名は登録したドメインを指定）
```
dig -t txt _acme-challenge.internal.onodera-program.com
```

"ANSWER SECTION:"に値が記載されていれば反映されている
```
; <<>> DiG 9.10.6 <<>> -t txt _acme-challenge.internal.onodera-program.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61386
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;_acme-challenge.internal.onodera-program.com. IN TXT

;; ANSWER SECTION:
_acme-challenge.internal.onodera-program.com. 300 IN TXT "***1b5eLnL15fqIdYkuSaqRu1CPGmgq6FTgnEHbZVTs"

;; Query time: 93 msec
;; SERVER: 192.168.10.1#53(192.168.10.1)
;; WHEN: Sun Dec 08 11:32:45 JST 2024
;; MSG SIZE  rcvd: 129
```

## 証明書をダウンロードする
### 所有者変更
- chown:"change owner":所有者変更
- chmod:"change mode":アクセス権変更

**注意1**\
下記の方法でコピーしてもファイルの実態は別の場所にあり、シンボリックリンクがコピーされるだけなので、ローカルにダウンロードする際に権限エラーになってしまうので、**注意2**の方法でダウンロードできる
```
sudo su -
cp -r /etc/letsencrypt/live/internal.onodera-program.com /home/mao
cd /home/mao
chown -R mao internal.onodera-program.com
exit
```
```
mao@lets-encrypt-certbot:~$ sudo su -
root@lets-encrypt-certbot:~# cp -r /etc/letsencrypt/live/internal.onodera-program.com /home/mao
root@lets-encrypt-certbot:~# cd /home/mao
root@lets-encrypt-certbot:/home/mao# ls -l
total 4
drwxr-xr-x 2 root root 4096 Dec  8 03:05 internal.onodera-program.com
root@lets-encrypt-certbot:/home/mao# chown -R mao internal.onodera-program.com
root@lets-encrypt-certbot:/home/mao# ls -l
total 4
drwxr-xr-x 2 mao root 4096 Dec  8 03:05 internal.onodera-program.com
root@lets-encrypt-certbot:/home/mao# exit
logout
mao@lets-encrypt-certbot:~$
```

**注意2**\
上記のファイルをコピーしても別の場所にあるファイルをリンクしているだけなのでダウンロード時にエラーになってしまったので、
ファイル本体をコピーして所有権を変更する

シンボリックリンクのファイル本体（リンク先）がどこにあるか確認する
```
mao@lets-encrypt-certbot:~/internal.onodera-program.com$ ls -l
total 4
-rw-r--r-- 1 mao root 692 Dec  8 03:05 README
lrwxrwxrwx 1 mao root  52 Dec  8 03:05 cert.pem -> ../../archive/internal.onodera-program.com/cert1.pem
lrwxrwxrwx 1 mao root  53 Dec  8 03:05 chain.pem -> ../../archive/internal.onodera-program.com/chain1.pem
lrwxrwxrwx 1 mao root  57 Dec  8 03:05 fullchain.pem -> ../../archive/internal.onodera-program.com/fullchain1.pem
lrwxrwxrwx 1 mao root  55 Dec  8 03:05 privkey.pem -> ../../archive/internal.onodera-program.com/privkey1.pem
```

#### 変更手順
root権限にする
```
sudo su -
```
ファイルをホームディレクトリへコピーする
```
cp -r /etc/letsencrypt/archive/internal.onodera-program.com /home/mao
```
所有者を変更する
```
cd /home/mao
chown -R mao internal.onodera-program.com
```
root権限から抜ける
```
exit
```

### 証明書をローカルにダウンロードする
- Macで作業
```
scp -r mao@192.168.10.11:/home/mao/internal.onodera-program.com /Users/username
```
```
% > scp -r mao@192.168.10.11:/home/mao/internal.onodera-program.com /Users/username
mao@192.168.10.11's password: 
privkey1.pem                                                                          100%  241    35.4KB/s   00:00    
README                                                                                100%  692    57.3KB/s   00:00    
fullchain1.pem                                                                        100% 2872   272.6KB/s   00:00    
cert1.pem                                                                             100% 1306   134.0KB/s   00:00    
chain1.pem 
```

ダウンロードしたファイルの説明
- "privkey1.pem":秘密鍵
- "fullchain1.pem":証明書＋中間証明書の連結ファイル
- "cert1.pem":証明書
- "chain1.pem":中間証明書

この証明書をアップロードしたりして使用できる

## 参考URL
- Certbot documentation
    - https://eff-certbot.readthedocs.io/en/stable/
- Let’s Encrypt ドキュメント
    - https://letsencrypt.org/ja/docs/
- 無料SSL証明書のLet’s Encryptとは？
    - https://ssl.sakura.ad.jp/column/letsencrypt/
- シンボリックリンクの作成と削除
    - https://qiita.com/colorrabbit/items/2e99304bd92201261c60
