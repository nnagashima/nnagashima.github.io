# RHEL9検証-Cockpit導入

# 目次
- [RHEL9検証-Cockpit導入](#rhel9検証-cockpit導入)
- [目次](#目次)
- [1.Cockpitとは](#1cockpitとは)
- [2.Cockpitのインストールと起動（自動起動有効化）](#2cockpitのインストールと起動自動起動有効化)
- [3.Cockpit画面の表示](#3cockpit画面の表示)
- [4.Cockpitの起動ポート変更](#4cockpitの起動ポート変更)
- [最後に](#最後に)

# 1.Cockpitとは

---

Webコンソールで以下を含むさまざまな管理タスクの実行が可能です。

- サービスの管理
- ユーザーアカウントの管理
- システムサービスの管理および監視
- ネットワークインターフェイスおよびファイアウォールの設定
- システムログの確認
- 仮想マシンの管理
- 診断レポートの作成
- カーネルダンプ設定の設定
- SELinux の設定
- ソフトウェアの更新
- システムサブスクリプションの管理

# 2.Cockpitのインストールと起動（自動起動有効化）

---

RHELで記載していますが、同様にRockyLinux/Ubuntuでも同様に利用できます。

```jsx
# dnf install -y cockpit
# systemctl enable --now cockpit.socket
```

# 3.Cockpit画面の表示

---

インストール後に、https://IPAddress:9090でアクセスすることで管理画面が表示されるので、

作成したユーザでログインすることで様々なOS情報を確認することができます。

# 4.Cockpitの起動ポート変更

---

以下のように設定します。

Cockpitソケット設定用のディレクトリを作成

```jsx
# mkdir /etc/systemd/system/cockpit.socket.d/
```

Confを作成（9090ポートから9091へ変更します）

```jsx
# vi /etc/systemd/system/cockpit.socket.d/listen.conf
[Socket]
ListenStream=
ListenStream=9091
```

SELinuxが有効な場合には以下を実行（設定したポートにバインドできるようにする）

```jsx
# semanage port -a -t websm_port_t -p tcp 9091
```

変更を再読み込みして、cockpit.socketを再起動します。

```jsx
# systemctl daemon-reload
# systemctl restart cockpit.socket
```

変更したポート番号でURLが表示されればOKです。

# 最後に

---

起動ポート変更以外にログインページにバナーを追加したり、アイドルロックを設定したりすることができます。

[RHEL 9 Web コンソールを使用したシステムの管理](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html-single/managing_systems_using_the_rhel_9_web_console/index#proc_providing-feedback-on-red-hat-documentation_system-management-using-the-rhel-9-web-console)