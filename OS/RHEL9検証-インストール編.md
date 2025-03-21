# RHEL9検証-インストール編


# 目次


# 利用環境

---

ProxmoxVE

# IOS準備

---

OS Downlodad：rhel-baseos-9.1-x86_64-dvd

https://access.redhat.com/downloads/content/479/ver=/rhel---9/9.1/x86_64/product-software

ProxmoxのStorageにて、ISO ImagesよりDownlodad from URLに

以下URLを入力して、Downloadを実施

https://access.cdn.redhat.com/content/origin/files/sha256/d9/d9dcae2b6e760d0f9dcf4a517bddc227d5fa3f213a8323592f4a07a05aa542a2/rhel-baseos-9.1-x86_64-dvd.iso?user=5ad4fd41c4be00b007836f2b425c5796&_auth_=1675667562_4d59c156ea8ea71e00cb4fce98300825

![](\OS\RHEL9検証-インストール編\Untitled.png)

# 仮想マシン作成

---

Proxmox上でサーバ作成を実施。スペックは以下の通り。

![](OS\RHEL9検証-インストール編\Untitled1.png)

# 仮想マシン起動とOSインストール

---

Install Red Hat Enterprise Linux 9.1を選択

![](\OS\RHEL9検証-インストール編\Untitled2.png)

日本語を選択

![](\OS\RHEL9検証-インストール編\Untitled3.png)

インストール概要の画面がでるので各項目ごとに設定します。

![](\OS\RHEL9検証-インストール編\Untitled4.png)

1. RedHatに接続の画面ではRHのアカウント情報を入力し登録します。

![](\OS\RHEL9検証-インストール編\Untitled5.pngg)

1. インストール先でパーティションを作成します。

ここではカスタムを選択しています。

LVMシンプロビジョニングで、構成は自動作成としています。作成したら完了をクリックします。

![](\OS\RHEL9検証-インストール編\Untitled6.png)

ファイルシステムはデフォルトでxfsとなっていますが、他にもext4などがあります。

フォーマット形式についてはXFSをRHは推奨としています。

[第1章 利用可能なファイルシステムの概要 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/managing_file_systems/assembly_overview-of-available-file-systems_managing-file-systems#types-of-file-systems_assembly_overview-of-available-file-systems)

[1.5. XFS と ext4 の比較 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/managing_file_systems/comparison-of-xfs-and-ext4_assembly_overview-of-available-file-systems)

1. ユーザーの設定でrootパスワードを設定します。

なお、rootアカウントはデフォルトで無効になっているようです。以下初期状態。

rootで作業しないでねってことみたいですね。

![](\OS\RHEL9検証-インストール編\Untitled7.png)

なのでユーザーの作成をします。

なお、作成するにあたりroot設定をしない場合にはsudo権限が必要となるので、

このユーザーを管理者にするにチェックをいれてください。

高度をクリックするとホームディレクトリやユーザーIDやグループIDを設定できます。

![](\OS\RHEL9検証-インストール編\Untitled8.png)

1. ソフトウェアの選択

自分でPKGは追加するのと必要ないものを導入はしたくないので最小限のインストールで進めます。

![](\OS\RHEL9検証-インストール編\Untitled9.png)

1. ここまで設定できたらインストールの開始をクリックします。

※KDUMPやネットワークとホスト名もGUI上で設定できるので事前に設定したい場合には設定してください。

![](\OS\RHEL9検証-インストール編\Untitled10.png)

インストールが完了したら再起動を実施し、コンソールからログインできることを確認してください。

![](\OS\RHEL9検証-インストール編\Untitled11.png)