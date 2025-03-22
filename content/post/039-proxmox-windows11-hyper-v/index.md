---
title: Proxmox上にWindows11をインストールしてHyper-V使えるようにする（1/2）
date: 2025-03-23
slug: proxmox-windows11-hyper-v-1
categories:
    - Proxmox
    - Windows
    - Hyper-V
---

## 環境
- Promox 8.3.0
  - kernel: 6.8.12-8-pve
  - pve-manager: 8.3.4
- Windows11 24H2
- virtio-win-0.1.266

## 準備
Windows11
- https://www.microsoft.com/ja-jp/software-download/windows11

virtio（vitrio-win-0.1.266.iso）
- https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/

上記は両方ともPromoxにISOとしてアップロードする

##  Windows11のVM構築
ISOイメージはWindows11を選択する\
ゲストOSを"Microsoft Windows"にし、バージョンを"11/2022/2025"にする\
その下にある"VirtIOドライバ用の追加ドライブを追加"にチェックを入れる\
ISOイメージを選択する（"vitrio-win-0.1.266.iso"）\
![](01.png)

"EFIストレージ"を保存する場所を選択する\
"Qemuエージェント"にチェックを入れる\
"TPM追加"にチェックを入れる\
"TPMストレージ"を保存する場所を選択する\
"バージョン"は"v2.0"を選択する\
![](02.png)

ディスクは100GBくらい確保しておく
![](03.png)

CPUの項目では"種別"を"host"にする
![](04.png)

メモリは16GBにする
![](05.png)

ネットワークはそのまま
![](06.png)

確認に画面になり、問題なければ"完了"を押す
![](07.png)


## Windows11インストール
設定を進めていきます
![](08.png)
![](09.png)
![](10.png)

プロダクトキーは後で入力するので"プロダクトキーがありません"を選択する
![](11.png)

"Windows 11 Pro"を選択する
![](12.png)
![](13.png)

"Windows 11 をインストールする場所の選択"の画面になったら、上のメニューの"Load Driver"か下の"ハードウェアが表示されませんか？ドライバーを読み込み、ハードウェアにアクセスします。"をクリックする
![](14.png)

画面が変わったら"参照"をクリックし、"virtio-win-0.1.226"を選択します
![](15.png)
![](16.png)

"amd64">"w11"を選択し、OKをクリックします
"VirtIO SCSI pass-through controller"を選択して、"インストール"をクリックします
するとディスクが表示されます
![](17.png)
![](18.png)

続けて他のドライバーもインストールします
![](19.png)
上のメニューの"Load Driver"をクリックします（"適用される通知とライセンス条項"が表示されたら"同意する"をクリックします）
![](20.png)
"参照"をクリックし、"virtio-win-0.1.226"を選択します
"NetKVM">"w11">"amd64"を選択し、OKをクリックします
"VirtIO Ethernet Adapter"を選択して"インストール"をクリックします
![](21.png)
![](22.png)
![](23.png)

同じ手順で
"Balloon">"w11">"amd64"を選択し、OKをクリックします
"VirtIO Balloon Driver"を選択して"インストール"をクリックします
![](24.png)
![](25.png)
![](26.png)

次へを押します
"インストール準備完了"が表示されたら"インストール"をクリックします
インストールが始まります
![](27.png)
![](28.png)

ここから先は通常のWindows11と同じようにセットアップしていきます
![](29.png)
![](30.png)

ここまででWindows11のインストールは完了です

## Hyper-Vのインストール
Windows11のインストールが終わったらHyper-Vをインストールしていきます

コントロールパネルを開き、"プログラム"をクリックします
![](31.png)

"Windowsの機能の有効化または無効化"をクリックし、"Hyper-V"にチェックを入れ、"OK"をクリックします\
再起動をします
![](32.png)
![](33.png)
![](34.png)
![](35.png)

タスクバーのWindowsマークをクリックし、"すべて"をクリックします
![](36.png)

"Windowsツール"をクリックします
![](37.png)

"Hyper-Vマネージャ"を右クリックして"ショートカットの作成"をします（デスクトップに作成しておきます）
![](38.png)
![](39.png)

ダブルクリックで開けます
![](40.png)
