---
title: セカンダリDNSの追加
date: 2025-03-13
slug: secondary-dns
categories:
    - Ubuntu
---

## 理由
ローカル内にあるDNSで名前解決して、そのDNSが落ちているときは別のDNSで通信自体はできるようにしたかった\
そこでセカンダリDNSをどのように追加したら良いのか調べた

## UbuntuDesktopのGUIの場合
- "Settings">"Network">"Ethernet(eno1)">"IPv4">"DNS"
- "192.168.10.1, 192.168.10.109"
- カンマ区切りで入れられる
- 先にあるのがプライマリで、その次がセカンダリになる

## UbuntuServerのCUI（Netplanファイル）の場合
- 追加前
```
network:
  version: 2
  ethernets:
    eno1:
      renderer: NetworkManager
      match: {}
      addresses:
      - "192.168.10.50/24"
      nameservers:
        addresses:
        - 192.168.10.1
```

- 追加後
```
network:
  version: 2
  ethernets:
    eno1:
      renderer: NetworkManager
      match: {}
      addresses:
      - "192.168.10.50/24"
      nameservers:
        addresses:
        - 192.168.10.1
        - 192.168.10.109
```

## "resolvectl"コマンドで確認する
- 追加前
```
mao@mao (x86_64) : /home/mao 
> resolvectl
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (eno1)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.10.1
       DNS Servers: 192.168.10.1
```

- 追加後
```
mao@mao (x86_64) : /home/mao 
> resolvectl
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (eno1)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.10.1
       DNS Servers: 192.168.10.1 192.168.10.109
```

"DNS Servers"の項目に追加されている

## 運用
プライマリをローカルのDNSにすれば、ローカル内の名前解決を先にしてないものはパブリックのDNSに問い合わせる形にできる
かつ、ローカルのDNSがダウンしていてもセカンダリのDNSに問い合わせる形にできる
