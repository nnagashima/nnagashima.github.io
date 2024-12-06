# RHEL9検証 ①インストール編

タグ: RHEL

![images.png](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A6%20Cockpit%E5%B0%8E%E5%85%A5%201289da9f37308067a685deffd613c2bf/images.png)

目次

# 利用環境

---

ProxmoxVE 7.3.4

# IOS準備

---

OS Downlodad：rhel-baseos-9.1-x86_64-dvd

https://access.redhat.com/downloads/content/479/ver=/rhel---9/9.1/x86_64/product-software

ProxmoxのStorageにて、ISO ImagesよりDownlodad from URLに

以下URLを入力して、Downloadを実施

https://access.cdn.redhat.com/content/origin/files/sha256/d9/d9dcae2b6e760d0f9dcf4a517bddc227d5fa3f213a8323592f4a07a05aa542a2/rhel-baseos-9.1-x86_64-dvd.iso?user=5ad4fd41c4be00b007836f2b425c5796&_auth_=1675667562_4d59c156ea8ea71e00cb4fce98300825

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled.png)

# 仮想マシン作成

---

Proxmox上でサーバ作成を実施。スペックは以下の通り。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%201.png)

# 仮想マシン起動とOSインストール

---

Install Red Hat Enterprise Linux 9.1を選択

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%202.png)

日本語を選択

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%203.png)

インストール概要の画面がでるので各項目ごとに設定します。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%204.png)

1. RedHatに接続の画面ではRHのアカウント情報を入力し登録します。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%205.png)

1. インストール先でパーティションを作成します。

ここではカスタムを選択しています。

LVMシンプロビジョニングで、構成は自動作成としています。作成したら完了をクリックします。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%206.png)

ファイルシステムはデフォルトでxfsとなっていますが、他にもext4などがあります。

フォーマット形式についてはXFSをRHは推奨としています。

[第1章 利用可能なファイルシステムの概要 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/managing_file_systems/assembly_overview-of-available-file-systems_managing-file-systems#types-of-file-systems_assembly_overview-of-available-file-systems)

[1.5. XFS と ext4 の比較 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/managing_file_systems/comparison-of-xfs-and-ext4_assembly_overview-of-available-file-systems)

1. ユーザーの設定でrootパスワードを設定します。

なお、rootアカウントはデフォルトで無効になっているようです。以下初期状態。

rootで作業しないでねってことみたいですね。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%207.png)

なのでユーザーの作成をします。

なお、作成するにあたりroot設定をしない場合にはsudo権限が必要となるので、

このユーザーを管理者にするにチェックをいれてください。

高度をクリックするとホームディレクトリやユーザーIDやグループIDを設定できます。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%208.png)

1. ソフトウェアの選択

自分でPKGは追加するのと必要ないものを導入はしたくないので最小限のインストールで進めます。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%209.png)

1. ここまで設定できたらインストールの開始をクリックします。

※KDUMPやネットワークとホスト名もGUI上で設定できるので事前に設定したい場合には設定してください。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%2010.png)

インストールが完了したら再起動を実施し、コンソールからログインできることを確認してください。

![Untitled](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A0%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%B7%A8%2092271ebbf7cc468ab17c90ddbed83fc9/Untitled%2011.png)