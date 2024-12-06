# RockyLinux9+Zabbix6+Postgresql14のインストール

![Zabbix_logo-RGB.png](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Zabbix_logo-RGB.png)

目次

# はじめに

---

RockyLinux9でZabbix6とPostgresql14を利用して自宅でZabbixを作ったナレッジです。

以前MySQLを使った高可用性で作成しましたが、リソースを使うのでシンプルにシングル構成にしました。

※その後、フロントUbutnu22.04でHA構成にして、DBをk8s基盤で動いているPostgresOperatorで動かしました。

# Zabbix6の要件は以下になる

---

[2 要件](https://www.zabbix.com/documentation/6.0/jp/manual/installation/requirements)

# OSの言語設定

---

デフォルトでは以下となっていましたので、日本語を追加します。

```bash
# localectl
System Locale: LANG=C.UTF-8
VC Keymap: jp
X11 Layout: jp
```

以下コマンドで日本語PKGを追加してデフォルト言語を設定します。

```bash
# dnf -y install langpacks-ja glibc-langpack-ja
# localectl set-locale LANG=ja_JP.UTF-8
```

設定後デフォルトになっていることを確認します。

```bash
# localectl 
   System Locale: LANG=ja_JP.UTF-8
       VC Keymap: jp
      X11 Layout: jp
```

# Firewalldの無効化と停止

---

必要なポート開ければ良いですが、Firewallを今回は無効化して停止します。

```yaml
# systemctl disable firewalld
Removed "/etc/systemd/system/multi-user.target.wants/firewalld.service".
Removed "/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service".
# systemctl stop firewalld
```

# レポジトリの追加

---

ZabbixとPostgresqlのリポジトリを追加します。

```bash
# rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
# dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

# Postgresqlのインストーとセットアップ

---

デフォルトのPostgreSQLを無効化してリポジトリから持ってきたPostgreSQLをインストールします。

```bash
# dnf -qy module disable postgresql
# dnf update -y
# dnf install -y postgresql14-server
```

インストールしたらPostgreSQLのセットアップを実施と起動ならびに自動起動の有効化を行います。

起動後にpsqlで接続ができることを確認しております。

```bash
# /usr/pgsql-14/bin/postgresql-14-setup initdb
Initializing database ... OK

# systemctl enable postgresql-14
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql-14.service → /usr/lib/systemd/system/postgresql-14.service.

# systemctl start postgresql-14
# systemctl status postgresql-14
● postgresql-14.service - PostgreSQL 14 database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-01-31 19:49:04 JST; 3s ago
       Docs: https://www.postgresql.org/docs/14/static/
    Process: 22434 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 22439 (postmaster)
      Tasks: 8 (limit: 50518)
     Memory: 16.6M
        CPU: 39ms
     CGroup: /system.slice/postgresql-14.service
             ├─22439 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/
             ├─22440 "postgres: logger "
             ├─22442 "postgres: checkpointer "
             ├─22443 "postgres: background writer "
             ├─22444 "postgres: walwriter "
             ├─22445 "postgres: autovacuum launcher "
             ├─22446 "postgres: stats collector "
             └─22447 "postgres: logical replication launcher "

Jan 31 19:49:04 ray-zbx6 systemd[1]: Starting PostgreSQL 14 database server...
Jan 31 19:49:04 ray-zbx6 postmaster[22439]: 2023-01-31 19:49:04.047 JST [22439] LOG:  redirecting log output to logging collector process
Jan 31 19:49:04 ray-zbx6 postmaster[22439]: 2023-01-31 19:49:04.047 JST [22439] HINT:  Future log output will appear in directory "log".
Jan 31 19:49:04 ray-zbx6 systemd[1]: Started PostgreSQL 14 database server.

# sudo -i -u postgres psql
SELECT version();
                                                 version                                                  
----------------------------------------------------------------------------------------------------------
 PostgreSQL 14.6 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.2.1 20220127 (Red Hat 11.2.1-9), 64-bit
(1 row)

$ ¥q
```

# Zabbixのインストール

---

必要なPKGをインストールします。

```bash
# dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent2 zabbix-web-japanese
```

Zabbixに必要なDBを作るために、PostgreSQLのpostgresユーザーにパスワードを付与します。

```bash
# sudo -u postgres psql -c "alter user postgres with password 'postgres'"
```

pg_hba.confを編集し、パスワード認証を設定します。

なお、scram-sha-256 認証方式を利用をします。

```bash
# vi /var/lib/pgsql/14/data/pg_hba.conf
# "local" is for Unix domain socket connections only
local   all             all                                     **scram-sha-256**
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
```

ZabbixユーザーならびにZabbixのデータベースを作成します。

```bash
# useradd zabbix-user
# sudo -u postgres createuser --pwprompt zabbix-user
could not change directory to "/root": Permission denied
Enter password for new role: zabbix #設定したいパスワード
Enter it again: zabbix #設定したいパスワード

# sudo -u postgres createdb -O zabbix-user zabbix
could not change directory to "/root": Permission denied
Password: postgres #postgresユーザーのパスワード

# zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix-user psql zabbix
```

ZabbixServerの設定を編集します。編集する箇所はDBに周りの設定になります。

```bash
# vi /etc/zabbix/zabbix_server.conf
DBName=zabbix
DBUser=zabbix-user
DBPassword=zabbix
ProxyConfigFrequency=60　#Proxyとの同期間隔
```

ZabbixAgentの設定を編集します。編集する箇所は監視する際に必要な箇所になります。

```bash
# vi /etc/zabbix/zabbix_agent2.conf
Server=192.168.11.62 #ZabbixサーバもしくはプロキシのIP
ServerActive=192.168.11.62 #ZabbixサーバもしくはプロキシのIP
Hostname=ray-zbx6 #任意でわかりやすい名前
```

zabbix-server, zabbix-agent2, httpd, php-fpm の起動・自動起動有効化をします。

```bash
# systemctl restart zabbix-server zabbix-agent2 httpd php-fpm
# systemctl enable zabbix-server zabbix-agent2 httpd php-fpm
# systemctl status zabbix-server zabbix-agent2 httpd php-fpm
```

http://IPアドレス/zabbixをブラウザで開いたらデフォルトの言語を日本語にすることで以下画面になると思います。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled.png)

インストールするにあたっての前提条件のチェックがあるのでALL OKになっていることを確認します。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%201.png)

Database TLS encryptionはチェック外しました。

トランスポート暗号化を有効したい場合にはチェックを入れてください。

そのほかはデータベース名やユーザー、パスワードといったところを入力してください。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%202.png)

サーバ名はわかりやすい名前にし、タイムゾーンはAsia/Tokyoにしました。

テーマはお好みで。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%203.png)

最後に設定値の確認がでるので確認します。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%204.png)

インストールが完了すると以下の画面が表示されます。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%205.png)

ログイン画面がでるのでユーザー名：Admin、パスワード：zabbixにしてログインします。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%206.png)

ログインしたらZabbix Serverの名前をzabbix_agent2.confで設定した名前に変更、IPアドレスも同様に変更します。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%207.png)

その後、エージェントの状態のZBXが緑色になっていれば正常となりインストールは完了となります。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%208.png)

# 2023/02/02 追記 TimescaleDBの追加

---

TimescaleDBを追加することで性能向上やディスク使用量の削減に期待していきます。

時系列DBの機能を適用した親テーブルをハイパーテーブルと呼び、データ内部はチャンクと呼ばれるテーブルに分割されます。

また恩恵としてディスク領域の節約やHousekeeper による削除処理の負荷軽減に繋がりそうなので導入していきます。

以下が導入手順です。

```bash
# リポジトリを追加します。
# tee /etc/yum.repos.d/timescale_timescaledb.repo <<EOL
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/$(rpm -E %{rhel})/\$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
EOL

# timescaledbをインストールします
# dnf install timescaledb-2-postgresql-14

# timescaledb-tuneコマンドを使用してPostgresqlのチューニングをします。max-connsを使って接続数を明示的に指定しています。
# timescaledb-tune --pg-config=/usr/pgsql-14/bin/pg_config --max-conns=100

# postgresqlの再起動
# systemctl restart postgresql-14

# zabbix データベースに TimesaceleDB 拡張をインストール
# echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u zabbix-user psql zabbix

# TimescaleDB のスキーマ作成スクリプト timescaledb.sql を実行
# cat /usr/share/zabbix-sql-scripts/postgresql/timescaledb.sql | sudo -u zabbix-user psql zabbix

# /etc/zabbix/zabbix_server.confに以下追記（サポートしていないTimesacleDBバージョンを利用する場合）
AllowUnsupportedDBVersions=1
```

# Zabbix Proxyのインストール

---

リポジトリ追加やPostgreSQLインストールまでは同じです。

以下からZabbixProxyの手順になります。

※ZabbixServerとは別のサーバで実施しています。

```bash
# dnf install zabbix-proxy-pgsql zabbix-sql-scripts zabbix-selinux-policy
# sudo -u postgres createuser --pwprompt zabbix
# sudo -u postgres createdb -O zabbix zabbix_proxy
# cat /usr/share/zabbix-sql-scripts/postgresql/proxy.sql | sudo -u zabbix psql zabbix_proxy
# vi /etc/zabbix/zabbix_proxy.conf
Server=ZabbixServerのIPアドレス
Hostname=ZabbixServerのプロキシに登録するホスト名
DBPassword=zabbix
ConfigFrequency=60 #Serverとの同期間隔

# systemctl restart zabbix-proxy
# systemctl enable zabbix-proxy
```

以下画面でZabbixProxyで設定したHostname欄の内容を、プロキシ名に追加します。

追加後に最新データ受信時刻が更新されていればOKです。

未同期の場合にはZabbixProxyのログやZabbixServerのログを見てみてください。

![Untitled](RockyLinux9+Zabbix6+Postgresql14%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%2004f354490ebf458b9883a62d93539bd4/Untitled%209.png)