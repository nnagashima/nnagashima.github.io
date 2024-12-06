# RHEL9検証 ③GUI＋VNC ServerでGUI操作したい

タグ: RHEL

![images.png](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A6%20Cockpit%E5%B0%8E%E5%85%A5%201289da9f37308067a685deffd613c2bf/images.png)

目次

# はじめに

---

Guacamoleでログインしていますが、ブラウザ操作をしたい時にVNC接続がしたいなと思いました。

Guacamole上でVNC接続もできるようにVNCサーバをインストールします。

# GUIのPKGをインストール

---

GUI関連のPKGをインストールします。

```bash
# dnf group install "Server with GUI"
# systemctl set-default graphical
# systemctl get-default
# reboot
```

# VNCサーバをインストール＆設定

---

VNCサーバを利用するにあたりインストールと、ユーザを作成しておきます。

```bash
# dnf install tigervnc-server
# useradd vnc-user
# passwd vnc-user
# visudo
vnc-user ALL=NOPASSWD:ALL
```

VNCパスワードの設定とディスプレイのサイズなどを設定します。

```bash
# su - vnc-user
$ vncpasswd
Password:
Verify:
Would you like to enter a view-only password (y/n)? n

$ vi .vnc/config
session=gnome
securitytypes=vncauth,tlsvnc
geometry=1920x1080
```

VNCを利用するディスプレイ番号とユーザー名を定義して、起動します。

```bash
$ vi /etc/tigervnc/vncserver.users
:2=vnc-user

$ sudo systemctl enable --now vncserver@:2
$ sudo systemctl is-active vncserver@\:2
active
$ ss -an | grep 5902
tcp   LISTEN 0      5                                                                                 0.0.0.0:5902               0.0.0.0:*           
tcp   LISTEN 0      5                                                                                    [::]:5902                  [::]:*
```

Guacamoleの接続設定に戻り、VNC接続での情報を追加してGuacamoleからVNC接続できれば完了です。

これでProxmoxのUIとか、ZabbixのUIとかも触れるようになったのでGoodです！