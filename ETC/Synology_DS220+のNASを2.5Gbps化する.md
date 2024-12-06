[⚫️TOPへ戻る](https://actmotech.xyz/)

[⚫️ETC Knowledgeへ戻る](/ETC/top)

# Synology DS220+のNASを2.5Gbps化する

目次
- [Synology DS220+のNASを2.5Gbps化する](#synology-ds220のnasを25gbps化する)
- [はじめに](#はじめに)
- [購入したもの](#購入したもの)
- [SynologyのOSバージョン確認](#synologyのosバージョン確認)
- [Synologyに搭載されているCPU確認](#synologyに搭載されているcpu確認)
- [Githubからドライバーをダウンロード](#githubからドライバーをダウンロード)
- [ドライバーのインストール](#ドライバーのインストール)
- [SynologyにSSHでログイン](#synologyにsshでログイン)
- [再度ドライバーのインストール](#再度ドライバーのインストール)

# はじめに

これからやることは非公式の内容ですので、何かあっても自己責任です。

筆者は成功していますが他所で同じように成功するかは分かりませんのでよく考えて実施してください。

# 購入したもの

Synology DS220+のNASを2.5Gbps化するにあたり、USB-LAN2500Rを購入しました。

[Planex USB to Type A Wired LAN Adapter (USB 3.2 Gen1) Transfer Rate Up to 2.5Gbps Supports Multi-Gigabit USB-LAN2500R](https://www.amazon.co.jp/gp/product/B07SC6DGQL/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)

# SynologyのOSバージョン確認

SynologyのUIにログインしてコントロールパネルから情報センターより確認します。

![](/ETC/Synology_DS220+のNASを2.5Gbps化する/image01.png)

# Synologyに搭載されているCPU確認

以下ページから確認できます。

今回はSynology DS220+なので、Geminilakeになります。

[Synology NAS に搭載されている CPU の種類は？ - Synology ナレッジセンター](https://kb.synology.com/ja-jp/DSM/tutorial/What_kind_of_CPU_does_my_NAS_have)

# Githubからドライバーをダウンロード

対応しているOSバージョンとCPU種類を確認したら対象のドライバーを以下からダウンロードします。

DSM7の場合にはDSM7.xとついたものをダウンロードしてください。

筆者が記載しているときの安定版は2.16.3-1 DSM7.xでしたので、

ダウンロードしたファイルはr8152-geminilake-2.16.3-1.spkになります。

[https://github.com/bb-qq/r8152/releases](https://github.com/bb-qq/r8152/releases)

# ドライバーのインストール

Synologyにダウンロードしたドライバーをパッケージセンターから手動インストールします。

最初は失敗しますが、失敗したままで次に進んでください。

# SynologyにSSHでログイン

SSHでログインできない場合にはUI上でSSHを許可してください。

その後Githubに記載があったコマンドを実行してください。

```bash
sudo install -m 4755 -o root -D /var/packages/r8152/target/r8152/spk_su /opt/sbin/spk_su
```

# 再度ドライバーのインストール

Synologyにダウンロードしたドライバーをパッケージセンターから手動インストールします。

1回目は失敗しましたが今回は成功しますので、ネットワーク設定を確認します。

ネットワークインターフェースにLAN3が追加されていれば完了です。

[⚫️TOPへ戻る](https://actmotech.xyz/)

[⚫️ETC Knowledgeへ戻る](/ETC/top)