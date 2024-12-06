# Zabbix6.0のHA構成（GaleraCluster）

![Zabbix_logo-RGB.png](Zabbix6%200%E3%81%AEHA%E6%A7%8B%E6%88%90%EF%BC%88GaleraCluster%EF%BC%89%20ff71ddfe98d34377b1e7eeaa10e75d25/Zabbix_logo-RGB.png)

目次

# はじめに

---

Zabbix6にWebフロントにHA構成が搭載されたので、その機能を試したいと思います。 

ですがフロントだけがHA機能なのでDBは別途考えなれければなりません。

 AWS RDSのようなマネージドサービスを利用すれば楽なのかもしれませんが、 

自宅サーバでZabbixを動かすのであればそんなことは言っていられません。 

なのでDBのHA構成するには比較的容易なGaleraClusterを利用して構築しました。

# 構成

---

Ubuntuサーバ３台のZabbixServer構成とZabbixProxyを動かします。 

ホスト名：zbx6-webdb01(192.168.12.31) / zbx6-webdb02(192.168.12.32) / zbx6-webdb03(192.168.12.33) / zbx6-proxy01(192.168.12.38)

GaleraClusterは2台構成でも動作はしますが、1台になった場合動作しなくなるので3台構成にしています。 

なお、起動しているDBはどのサーバでもRead/Writeが可能となります。

# MariaDBのインストール

---

対象：zbx6-webdb01 / zbx6-webdb02 / zbx6-webdb03

```
# apt install mariadb-server
# systemctl restart mariadb
# mysql_secure_installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n]
Enabled successfully!
Reloading privilege tables..
 ... Success!

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

# GaleraClusterの設定ファイルを作成します。

---

[zbx6-webdb01]

```
# cp -p /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf_20220916
# vi /etc/mysql/mariadb.conf.d/50-server.cnf
# diff -u0 /etc/mysql/mariadb.conf.d/50-server.cnf_20220916 /etc/mysql/mariadb.conf.d/50-server.cnf
--- /etc/mysql/mariadb.conf.d/50-server.cnf_20220916    2022-04-27 18:23:07.000000000 +0000
+++ /etc/mysql/mariadb.conf.d/50-server.cnf     2022-09-16 13:37:05.104500018 +0000
@@ -109,0 +110,9 @@
+wsrep_on=ON
+wsrep_provider=/usr/lib/galera/libgalera_smm.so
+wsrep_cluster_address="gcomm://192.168.12.31,192.168.12.32,192.168.12.33"
+binlog_format=ROW
+default_storage_engine=InnoDB
+innodb_autoinc_lock_mode=2
+innodb_doublewrite=1
+wsrep_node_address="192.168.12.31"
+wsrep_cluster_name="zabbix_db"
```

[zbx6-webdb02]

```
# diff -u0 /etc/mysql/mariadb.conf.d/50-server.cnf_20220916 /etc/mysql/mariadb.conf.d/50-server.cnf
--- /etc/mysql/mariadb.conf.d/50-server.cnf_20220916    2022-04-27 18:23:07.000000000 +0000
+++ /etc/mysql/mariadb.conf.d/50-server.cnf     2022-09-16 13:09:00.288042923 +0000
@@ -109,0 +110,9 @@
+wsrep_on=ON
+wsrep_provider=/usr/lib/galera/libgalera_smm.so
+wsrep_cluster_address="gcomm://192.168.12.31,192.168.12.32,192.168.12.33"
+binlog_format=ROW
+default_storage_engine=InnoDB
+innodb_autoinc_lock_mode=2
+innodb_doublewrite=1
+wsrep_node_address="192.168.12.32"
+wsrep_cluster_name="zabbix_db"
```

[zbx6-webdb03]

```
# diff -u0 /etc/mysql/mariadb.conf.d/50-server.cnf_20220916 /etc/mysql/mariadb.conf.d/50-server.cnf
--- /etc/mysql/mariadb.conf.d/50-server.cnf_20220916    2022-04-27 18:23:07.000000000 +0000
+++ /etc/mysql/mariadb.conf.d/50-server.cnf     2022-09-16 13:10:06.220384089 +0000
@@ -109,0 +110,9 @@
+wsrep_on=ON
+wsrep_provider=/usr/lib/galera/libgalera_smm.so
+wsrep_cluster_address="gcomm://192.168.12.31,192.168.12.32,192.168.12.33"
+binlog_format=ROW
+default_storage_engine=InnoDB
+innodb_autoinc_lock_mode=2
+innodb_doublewrite=1
+wsrep_node_address="192.168.12.33"
+wsrep_cluster_name="zabbix_db"
```

# GaleraClusterの起動

---

1台目で以下コマンドを実行します。

```
galera_new_cluster
```

2,3台目で以下コマンドを実行します。

```
systemctl restart mariadb
```

# GaleraClusterの起動を確認します。

---

wsrep_local_state_commentがSyncedになっていることを確認します。

```
# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 36
Server version: 10.6.7-MariaDB-2ubuntu1.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show status like 'wsrep_%';
+-------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name                 | Value                                                                                                                                          |
+-------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| wsrep_local_state_uuid        | bb41a595-3038-11ed-a8ed-6f31e730f072                                                                                                           |
| wsrep_protocol_version        | 10                                                                                                                                             |
| wsrep_last_committed          | 2                                                                                                                                              |
| wsrep_replicated              | 0                                                                                                                                              |
| wsrep_replicated_bytes        | 0                                                                                                                                              |
| wsrep_repl_keys               | 0                                                                                                                                              |
| wsrep_repl_keys_bytes         | 0                                                                                                                                              |
| wsrep_repl_data_bytes         | 0                                                                                                                                              |
| wsrep_repl_other_bytes        | 0                                                                                                                                              |
| wsrep_received                | 3                                                                                                                                              |
| wsrep_received_bytes          | 216                                                                                                                                            |
| wsrep_local_commits           | 0                                                                                                                                              |
| wsrep_local_cert_failures     | 0                                                                                                                                              |
| wsrep_local_replays           | 0                                                                                                                                              |
| wsrep_local_send_queue        | 0                                                                                                                                              |
| wsrep_local_send_queue_max    | 1                                                                                                                                              |
| wsrep_local_send_queue_min    | 0                                                                                                                                              |
| wsrep_local_send_queue_avg    | 0                                                                                                                                              |
| wsrep_local_recv_queue        | 0                                                                                                                                              |
| wsrep_local_recv_queue_max    | 1                                                                                                                                              |
| wsrep_local_recv_queue_min    | 0                                                                                                                                              |
| wsrep_local_recv_queue_avg    | 0                                                                                                                                              |
| wsrep_local_cached_downto     | 2                                                                                                                                              |
| wsrep_flow_control_paused_ns  | 0                                                                                                                                              |
| wsrep_flow_control_paused     | 0                                                                                                                                              |
| wsrep_flow_control_sent       | 0                                                                                                                                              |
| wsrep_flow_control_recv       | 0                                                                                                                                              |
| wsrep_flow_control_active     | false                                                                                                                                          |
| wsrep_flow_control_requested  | false                                                                                                                                          |
| wsrep_cert_deps_distance      | 0                                                                                                                                              |
| wsrep_apply_oooe              | 0                                                                                                                                              |
| wsrep_apply_oool              | 0                                                                                                                                              |
| wsrep_apply_window            | 0                                                                                                                                              |
| wsrep_apply_waits             | 0                                                                                                                                              |
| wsrep_commit_oooe             | 0                                                                                                                                              |
| wsrep_commit_oool             | 0                                                                                                                                              |
| wsrep_commit_window           | 0                                                                                                                                              |
| wsrep_local_state             | 4                                                                                                                                              |
| wsrep_local_state_comment     | Synced                                                                                                                                         |
| wsrep_cert_index_size         | 0                                                                                                                                              |
| wsrep_causal_reads            | 0                                                                                                                                              |
| wsrep_cert_interval           | 0                                                                                                                                              |
| wsrep_open_transactions       | 0                                                                                                                                              |
| wsrep_open_connections        | 0                                                                                                                                              |
| wsrep_incoming_addresses      | AUTO,AUTO                                                                                                                                      |
| wsrep_cluster_weight          | 2                                                                                                                                              |
| wsrep_desync_count            | 0                                                                                                                                              |
| wsrep_evs_delayed             |                                                                                                                                                |
| wsrep_evs_evict_list          |                                                                                                                                                |
| wsrep_evs_repl_latency        | 0/0/0/0/0                                                                                                                                      |
| wsrep_evs_state               | OPERATIONAL                                                                                                                                    |
| wsrep_gcomm_uuid              | 3f0a21cb-3039-11ed-91b5-c7627a45a889                                                                                                           |
| wsrep_gmcast_segment          | 0                                                                                                                                              |
| wsrep_applier_thread_count    | 1                                                                                                                                              |
| wsrep_cluster_capabilities    |                                                                                                                                                |
| wsrep_cluster_conf_id         | 2                                                                                                                                              |
| wsrep_cluster_size            | 2                                                                                                                                              |
| wsrep_cluster_state_uuid      | bb41a595-3038-11ed-a8ed-6f31e730f072                                                                                                           |
| wsrep_cluster_status          | Primary                                                                                                                                        |
| wsrep_connected               | ON                                                                                                                                             |
| wsrep_local_bf_aborts         | 0                                                                                                                                              |
| wsrep_local_index             | 0                                                                                                                                              |
| wsrep_provider_capabilities   | :MULTI_MASTER:CERTIFICATION:PARALLEL_APPLYING:TRX_REPLAY:ISOLATION:PAUSE:CAUSAL_READS:INCREMENTAL_WRITESET:UNORDERED:PREORDERED:STREAMING:NBO: |
| wsrep_provider_name           | Galera                                                                                                                                         |
| wsrep_provider_vendor         | Codership Oy <info@codership.com>                                                                                                              |
| wsrep_provider_version        | 4.9(rcece3ba2)                                                                                                                                 |
| wsrep_ready                   | ON                                                                                                                                             |
| wsrep_rollbacker_thread_count | 1                                                                                                                                              |
| wsrep_thread_count            | 2                                                                                                                                              |
+-------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
69 rows in set (0.001 sec)
```

# keepaliveのインストール

keepalivedのインストールを実施します。

```
# apt install keepalived
```

keepalivedの設定を作ります。

[zbx6-webdb01]

```
# cat /etc/keepalived/keepalived.conf
vrrp_script chk_mariadb {
    script "systemctl is-active mariadb"
}

vrrp_script chk_zabbix {
    script "systemctl is-active zabbix-server"
}

vrrp_script chk_apache {
    script "systemctl is-active apache2"
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18
    virtual_router_id 1
    priority 300
    virtual_ipaddress {
        192.168.12.34
    }
    track_script {
        chk_mariadb
        chk_zabbix
        chk_apache
    }
}
```

[zbx6-webdb02]

```
# cat /etc/keepalived/keepalived.conf
vrrp_script chk_mariadb {
    script "systemctl is-active mariadb"
}

vrrp_script chk_zabbix {
    script "systemctl is-active zabbix-server"
}

vrrp_script chk_apache {
    script "systemctl is-active apache2"
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18
    virtual_router_id 1
    priority 200
    virtual_ipaddress {
        192.168.12.34
    }
    track_script {
        chk_mariadb
        chk_zabbix
        chk_apache
    }
}
```

[zbx6-webdb03]

```
root@ray-zbx6-webdb03:~# cat /etc/keepalived/keepalived.conf
vrrp_script chk_mariadb {
    script "systemctl is-active mariadb"
}

vrrp_script chk_zabbix {
    script "systemctl is-active zabbix-server"
}

vrrp_script chk_apache {
    script "systemctl is-active apache2"
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18
    virtual_router_id 1
    priority 100
    virtual_ipaddress {
        192.168.12.34
    }
    track_script {
        chk_mariadb
        chk_zabbix
        chk_apache
    }
}
```

設定完了後、keepalivedを再起動します。

```
# systemctl restart keepalived.service
```

# Zabbix6.0のインストール

対象：zbx6-webdb01 / zbx6-webdb02 / zbx6-webdb03

Zabbix6のインストールをします。 

またZabbixAgentについてはAgent2を利用するためにAgent2をインストールします。 

違いについては以下に記載があります。 https://www.zabbix.com/documentation/6.0/jp/manual/appendix/agent_comparison

```
# curl -O https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bubuntu22.04_all.deb
# dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
# apt update
# apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2
```

対象：zbx6-webdb01

```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
# zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

対象：zbx6-webdb01 / zbx6-webdb02 / zbx6-webdb03

```
# cp -p /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf_20220916
# vi /etc/zabbix/zabbix_server.conf
# diff -u0 /etc/zabbix/zabbix_server.conf_20220916 /etc/zabbix/zabbix_server.conf
--- /etc/zabbix/zabbix_server.conf_20220916     2022-09-16 14:42:33.786549816 +0000
+++ /etc/zabbix/zabbix_server.conf      2022-09-16 14:58:33.433000801 +0000
@@ -129,0 +130 @@
+DBPassword=password　#DBパスワードを記載
@@ -443,0 +445 @@
+CacheSize=256M　#足りないとZabbixサーバプロセスがクラッシュします。大きすぎると起動に時間がかかります。
@@ -629,0 +632 @@
+ProxyConfigFrequency=60　#ZabbixProxyとの同期頻度
@@ -657,0 +661 @@
+AllowRoot=1　#ZabbixServerをrootで起動
@@ -720 +724 @@
-StatsAllowedIP=127.0.0.1
+StatsAllowedIP=192.168.12.31 #Zabbixの内部統計を取得するのに利用するIPを記載します。
@@ -986,0 +991 @@
+HANodeName=zbx6-webdb01 #HAモードを設定する際に識別するノード名を記載します。
@@ -996,0 +1002 @@
+NodeAddress=192.168.12.31:10051 #Zabbix の Web フロントエンドと Zabbix サーバーのアクティブなノードとの間の通信で使用します。

# diff -U0 zabbix_agent2.conf_20220917 zabbix_agent2.conf
--- zabbix_agent2.conf_20220917 2022-08-29 07:33:19.000000000 +0000
+++ zabbix_agent2.conf  2022-09-16 15:49:18.439999155 +0000
@@ -80 +80 @@
-Server=127.0.0.1
+Server=192.168.12.31,192.168.12.32,192.168.12.33 #ZabbixServerもしくはZabbixProxyのIPアドレスを記載します。
@@ -133 +133 @@
-ServerActive=127.0.0.1
+ServerActive=192.168.12.31,192.168.12.32,192.168.12.33 #ZabbixServerもしくはZabbixProxyのIPアドレスを記載します。
@@ -144 +144 @@
-Hostname=Zabbix server
+Hostname=ray-zbx6-webdb01 #ZabbixのUIにて登録するホスト名を記載します。

# systemctl restart zabbix-server zabbix-agent2 apache2
# systemctl enable zabbix-server zabbix-agent2 apache2
```

Zabbixを起動しても日本語フォントに対応していないので、フォントを追加します。

対象：zbx6-webdb01 / zbx6-webdb02 / zbx6-webdb03

```
# dpkg-reconfigure locales
# reboot
# apt install fonts-ipafont-gothic
# ln -s /usr/share/fonts/opentype/ipafont-gothic/ipagp.ttf /usr/share/zabbix/assets/fonts/ipagp.ttf
# vi /usr/share/zabbix/include/defines.inc.php
【変更前】
define('ZBX_GRAPH_FONT_NAME', 'graphfont');
define('ZBX_FONT_NAME', 'graphfont');

【変更後】
define('ZBX_GRAPH_FONT_NAME', 'ipagp');
define('ZBX_FONT_NAME', 'ipagp');
```

# ZabbixProxyのインストール

ZabbixServerの負荷を極力減らすため、監視対象からのデータ収集をZabbixProxyに任せます。

```
# wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bubuntu22.04_all.deb
# dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
# apt update
# apt install zabbix-proxy-mysql zabbix-sql-scripts
# mysql -uroot -p
password
mysql> create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix_proxy.* to zabbix@localhost;
mysql> quit;

# sed '/^$/d' /etc/zabbix/zabbix_proxy.conf | grep -v "#"
ProxyMode=1
Server=192.168.12.31,192.168.12.32,192.168.12.33
Hostname=zbx6-proxy01
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_proxy.pid
SocketDir=/run/zabbix
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password
ConfigFrequency=60
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
AllowRoot=1
StatsAllowedIP=192.168.12.38

# systemctl restart zabbix-proxy
# systemctl enable zabbix-proxy
```

# ZabbixAgentを監視対象へインストール

あとは監視したい対象にZabbixAgent2をインストールして設定し起動をします。 

インストールをしたらZabbixUIへログインをしてホストの登録とテンプレートを適用することで監視を開始することができます。

# 肝心のHAクラスターの動作確認

ActiveなZabbixServer上で以下コマンドを実行します。 

Activeモードの確認はsystemctl status zabbix-serverを実行することで、 ZabbixServerのプロセスツリーを見ることでわかります。 

Activeの場合は必要なプロセスが多岐に渡り起動していますが、Standbyはメインプロセスとha managerプロセスのみです。

 unavailableに至ってはメインプロセスのみです。

```
# zabbix_server -R ha_status
Failover delay: 60 seconds
Cluster status:
   #  ID                        Name                      Address                        Status      Last Access
   1. cl84lvk3w0001r955qois5756 zbx6-webdb01          192.168.12.31:10051            standby     4s
   2. cl84m0pdx0001qq579h999k90 zbx6-webdb03          192.168.12.33:10051            unavailable 7d 1h 1m 32s
   3. cl84m50g60001kg56q8pcaymn zbx6-webdb02          192.168.12.32:10051            active      0s
```

HAの切り替わり時間はデフォルトだと60秒になりますが、カスタムで変更することも可能です。 

可能な範囲は10秒から15分になります。設定はActiveモードで動作している対象で実施します。

```
# zabbix_server -R ha_set_failover_delay=10s
```

フェイルオーバーについてはha_manager プロセス5秒おきにDBとハードビートを送信して、

自身のステータスや他ノードのステータスを監視します。

 ha manager プロセスはハートビートによってアクセスした時刻を更新し、アクティブノードのアクセス時刻が、 

ha_set_failover_delayで設定した時間以上更新されていない場合は障害となりフェイルオーバーを実行されます。

フェイルオーバーテストについては、Avtiveのzabbix-serverプロセスを停止することでStandbyに切り替わったことを確認します。

```
#  zabbix_server -R ha_status
Failover delay: 10 seconds
Cluster status:
   #  ID                        Name                      Address                        Status      Last Access
   1. cl84lvk3w0001r955qois5756 ray-zbx6-webdb01          192.168.12.31:10051            active      3s
   2. cl84m0pdx0001qq579h999k90 ray-zbx6-webdb03          192.168.12.33:10051            unavailable 7d 1h 14m 51s
   3. cl84m50g60001kg56q8pcaymn ray-zbx6-webdb02          192.168.12.32:10051            stopped     11s
```

# 最後に

冒頭にも書いたようにDBのHAは含まれていないので、DBに対してどうするかは考えないといけないかと思います。 

今回はZabbixServerのHA機能ですが、今後ZabbixProxyのHA機能が実装予定ということなのでUpdate情報は追っていこうと思います。

ZabbixのRoadmapを見ると6.4ではEnd-user Web monitoringやパブリッククラウド向けの監視テンプレート開発が入っています。 

7.2ではAPMも入っているのでSaaSで提供しているモニタリングツールの後をしっかり追っていこうという意思が伝わってきます。 

これからZabbixがどういった方向に向かっていくのかにも注目していきたいと思います。 

[Zabbix roadmap](https://www.zabbix.com/roadmap)