# Zabbix5.0でPostgresql13＋pgpool-IIを利用したCluster構築

![Zabbix_logo-RGB.png](Zabbix5%200%E3%81%A6%E3%82%99Postgresql13%EF%BC%8Bpgpool-II%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%9FCluster%E6%A7%8B%E7%AF%89%2027f6ddeee06e4af5a6f5ca92627fd413/Zabbix_logo-RGB.png)

目次

# はじめに

---

postgresqlを使ったZabbixの情報はあまり出てこないので、 色々と自分で試しつつ、Zabbix5.0のLTSをCluster構成にしてみました。

pgpoolの公式サイトを参考にしながら作ってはいますが、間違っているところがあるかもしれませんので、 

利用する方はあくまで参考程度にしてくだい。

また今回の投稿では細かい設定パラメータまでは触れません。

実際はパラメータの調整をしていますが利用の仕方によっても変わってくるので、Postgresqlのパタメータをちゃんと実施してくだい。

MySQLと違ってPostgresqlはちゃんとパラメータ調整しないとうまく動作しない可能性があります。

# 事前準備

---

OSはCentOS7の最新版で実施しています。 またPostgresql用に/var/lib/pgsqlのパーティションを作っています。

Active-Standby構成で作成するので、一部共通で設定する箇所、そうでない箇所があるので注意して下さい。

# postgresql+pgpoolの設定

---

< 共通設定 >

hostsファイルに名前を設定します。

```
# echo "192.168.11.41 zbx5-webdb01" >> /etc/hosts
# echo "192.168.11.42 zbx5-webdb02" >> /etc/hosts
```

pgpoolとpostgresqlのリポジトリ設定、インストールします。

```
# yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
# yum -y install https://www.pgpool.net/yum/rpms/4.2/redhat/rhel-7-x86_64/pgpool-II-release-4.2-1.noarch.rpm
# yum -y install postgresql13-server
# yum -y install pgpool-II-pg13-*
# systemctl enable postgresql-13.service
# systemctl enable pgpool.service
```

レプリケーションするのでアーカイブディレクトリを作成して起動します。

```
# su - postgres
$ mkdir /var/lib/pgsql/archivedir
$ exit
# /usr/pgsql-13/bin/postgresql-13-setup initdb
# ls -la /var/lib/pgsql/13/data
```

Postgresqlでtimescaledbを利用します。

```
# tee /etc/yum.repos.d/timescale_timescaledb.repo <<EOL
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/$(rpm -E %{centos})/\$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
EOL

# yum install -y timescaledb-2-postgresql-13
```

< zbx5-webdb01だけの設定 >

timescaledbの初期設定を実施します。デフォルト設定にしています。

```
# timescaledb-tune --pg-config=/usr/pgsql-13/bin/pg_config
Using postgresql.conf at this path:
/var/lib/pgsql/13/data/postgresql.conf

Is this correct? [(y)es/(n)o]: y
Writing backup to:
/tmp/timescaledb_tune.backup202109201052

shared_preload_libraries needs to be updated
Current:
#shared_preload_libraries = ''
Recommended:
shared_preload_libraries = 'timescaledb'
Is this okay? [(y)es/(n)o]: y
success: shared_preload_libraries will be updated

Tune memory/parallelism/WAL and other settings? [(y)es/(n)o]: y
Recommendations based on 3.84 GB of available memory and 2 CPUs for PostgreSQL 13

Memory settings recommendations
Current:
shared_buffers = 128MB
#effective_cache_size = 4GB
#maintenance_work_mem = 64MB
#work_mem = 4MB
Recommended:
shared_buffers = 1005866kB
effective_cache_size = 2946MB
maintenance_work_mem = 502933kB
work_mem = 10058kB
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: memory settings will be updated

Parallelism settings recommendations
Current:
missing: timescaledb.max_background_workers
#max_worker_processes = 8
#max_parallel_workers_per_gather = 2
#max_parallel_workers = 8
Recommended:
timescaledb.max_background_workers = 8
max_worker_processes = 13
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: parallelism settings will be updated

WAL settings recommendations
Current:
#wal_buffers = -1
min_wal_size = 80MB
Recommended:
wal_buffers = 16MB
min_wal_size = 512MB
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: WAL settings will be updated

Miscellaneous settings recommendations
Current:
#default_statistics_target = 100
#random_page_cost = 4.0
#checkpoint_completion_target = 0.5
max_connections = 100
#max_locks_per_transaction = 64
#autovacuum_max_workers = 3
#autovacuum_naptime = 1min
#effective_io_concurrency = 1
Recommended:
default_statistics_target = 500
random_page_cost = 1.1
checkpoint_completion_target = 0.9
max_connections = 50
max_locks_per_transaction = 64
autovacuum_max_workers = 10
autovacuum_naptime = 10
effective_io_concurrency = 200
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: miscellaneous settings will be updated
Saving changes to: /var/lib/pgsql/13/data/postgresql.conf
```

Postgresqlで以下設定をします。

```
# cp -ip /var/lib/pgsql/13/data/postgresql.conf /home/centos/postgresql.conf.org
# vi /var/lib/pgsql/13/data/postgresql.conf
listen_addresses = '*'
archive_mode = on
archive_command = 'cp "%p" "/var/lib/pgsql/archivedir/%f"'
max_wal_senders = 10
max_replication_slots = 10
wal_level = replica
hot_standby = on
wal_log_hints = on
log_checkpoints = on
log_connections = on
log_disconnections = on
log_duration = on
log_directory = '/var/log/postgres_log'
log_filename = 'postgresql-%Y%m%d.log'

# mkdir /var/log/postgres_log
# chown postgres:postgres /var/log/postgres_log
# systemctl start postgresql-13
# systemctl enable postgresql-13
# su - postgres
$ psql -U postgres -p 5432
postgres=# SET password_encryption = 'scram-sha-256';
postgres=# CREATE ROLE pgpool WITH LOGIN;
postgres=# CREATE ROLE repl WITH REPLICATION LOGIN;
postgres=# \password pgpool
postgres=# \password repl
postgres=# \password postgres
postgres=# GRANT pg_monitor TO pgpool;
postgres=# exit
$ exit

# cp -p /var/lib/pgsql/13/data/pg_hba.conf /home/centos/pg_hba.conf.org
# vi /var/lib/pgsql/13/data/pg_hba.conf
host    all             all             samenet                 scram-sha-256
host    replication     all             samenet                 scram-sha-256
```

< 共通設定 >

パスワードなしでSSHで接続できるようにします。

```
# mkdir .ssh
# passwd postgres
# cd .ssh
# ssh-keygen -t rsa -f id_rsa_pgpool
# ssh-copy-id -i id_rsa_pgpool.pub postgres@zbx5-webdb01
# ssh-copy-id -i id_rsa_pgpool.pub postgres@zbx5-webdb02
# su - postgres
$ cd ~/.ssh
$ ssh-keygen -t rsa -f id_rsa_pgpool
$ ssh-copy-id -i id_rsa_pgpool.pub postgres@zbx5-webdb01
$ ssh-copy-id -i id_rsa_pgpool.pub postgres@zbx5-webdb02
$ exit

# su - postgres
$ vi /var/lib/pgsql/.pgpass
zbx5-webdb01:5432:replication:repl:repl
zbx5-webdb02:5432:replication:repl:repl
zbx5-webdb01:5432:postgres:postgres:postgres
zbx5-webdb02:5432:postgres:postgres:postgres
$ chmod 600 /var/lib/pgsql/.pgpass
```

< zbx5-webdb01だけの設定 >

pgpool_node_idファイルを作成します。

```
# vi /etc/pgpool-II/pgpool_node_id
0
```

< zbx5-webdb02だけの設定 >

pgpool_node_idファイルを作成します。

```
# vi /etc/pgpool-II/pgpool_node_id
1
```

< zbx5-webdb01だけの設定 >

pgpoolで以下設定をします。

```
# cp -p /etc/pgpool-II/pgpool.conf.sample-stream /etc/pgpool-II/pgpool.conf
# vi /etc/pgpool-II/pgpool.conf
listen_addresses = '*'
port = 9999
sr_check_user = 'pgpool'
sr_check_password = ''
health_check_period = 5
health_check_timeout = 30
health_check_user = 'pgpool'
health_check_password = ''
health_check_max_retries = 3
backend_hostname0 = 'zbx5-webdb01'
backend_port0 = 5432
backend_weight0 = 1
backend_data_directory0 = '/var/lib/pgsql/13/data'
backend_flag0 = 'ALLOW_TO_FAILOVER'
backend_application_name0 = 'zbx5-webdb01'
backend_hostname1 = 'zbx5-webdb02'
backend_port1 = 5432
backend_weight1 = 1
backend_data_directory1 = '/var/lib/pgsql/13/data'
backend_flag1 = 'ALLOW_TO_FAILOVER'
backend_application_name1 = 'zbx5-webdb02'

failover_command = '/etc/pgpool-II/failover.sh %d %h %p %D %m %H %M %P %r %R %N %S'
follow_primary_command = '/etc/pgpool-II/follow_primary.sh %d %h %p %D %m %H %M %P %r %R'
recovery_user = 'postgres'
recovery_password = ''
recovery_1st_stage_command = 'recovery_1st_stage'
enable_pool_hba = on
use_watchdog = on
delegate_IP = '192.168.11.43'
if_up_cmd = '/usr/bin/sudo /sbin/ip addr add $_IP_$/24 dev eth0 label eth0:0'
if_down_cmd = '/usr/bin/sudo /sbin/ip addr del $_IP_$/24 dev eth0'
arping_cmd = '/usr/bin/sudo /usr/sbin/arping -U $_IP_$ -w 1 -I eth0'
if_cmd_path = '/sbin'
arping_path = '/usr/sbin'
hostname0 = 'zbx5-webdb01'
wd_port0 = 9090
pgpool_port0 = 9999
hostname1 = 'zbx5-webdb02'
wd_port1 = 9090
pgpool_port1 = 9999
wd_lifecheck_method = 'heartbeat'
wd_interval = 10
heartbeat_hostname0 = 'zbx5-webdb01'
heartbeat_port0 = 9694
heartbeat_device0 = ''
heartbeat_hostname1 = 'zbx5-webdb02'
heartbeat_port1 = 9694
heartbeat_device1 = ''
wd_heartbeat_keepalive = 2
wd_heartbeat_deadtime = 30
wd_escalation_command = '/etc/pgpool-II/escalation.sh'
log_destination = 'stderr'
logging_collector = on
log_directory = '/var/log/pgpool_log'
log_filename = 'pgpool-%Y%m%d.log'
log_truncate_on_rotation = on
log_rotation_age = 1d
log_rotation_size = 0
enable_consensus_with_half_votes = on

# scp -p /etc/pgpool-II/pgpool.conf root@zbx5-webdb02:/etc/pgpool-II/pgpool.conf

# cp -p /etc/pgpool-II/recovery_1st_stage.sample /var/lib/pgsql/13/data/recovery_1st_stage
# cp -p /etc/pgpool-II/pgpool_remote_start.sample /var/lib/pgsql/13/data/pgpool_remote_start
# chown postgres:postgres /var/lib/pgsql/13/data/{recovery_1st_stage,pgpool_remote_start}
# su - postgres
$ psql template1 -c "CREATE EXTENSION pgpool_recovery"
```

< 共通設定 >

ログフォルダ、フェイルオーバー、pgpoolの仮装IPを停止させるスクリプト、クライアント認証の設定します。

```
# chown postgres:postgres /var/log/postgres_log
# mkdir /var/log/pgpool_log/
# chown postgres:postgres /var/log/pgpool_log/
# cp -p /etc/pgpool-II/failover.sh{.sample,}
# cp -p /etc/pgpool-II/follow_primary.sh{.sample,}
# chown postgres:postgres /etc/pgpool-II/{failover.sh,follow_primary.sh}
# cp -p /etc/pgpool-II/escalation.sh{.sample,}
# chown postgres:postgres /etc/pgpool-II/escalation.sh

# echo 'pgpool:'`pg_md5 pgpool` >> /etc/pgpool-II/pcp.conf
# vi /etc/pgpool-II/pool_hba.conf
host    all         all         192.168.11.41/32              scram-sha-256
host    all         all         192.168.11.42/32              scram-sha-256
host    all         all         192.168.11.43/32              scram-sha-256
host    all         pgpool           0.0.0.0/0          scram-sha-256
host    all         postgres         0.0.0.0/0          scram-sha-256

# su - postgres
$ echo 'localhost:9898:pgpool:pgpool' > ~/.pcppass
$ chmod 600 ~/.pcppass

# su - postgres
$ echo 'postgres' > ~/.pgpoolkey
$ chmod 600 ~/.pgpoolkey
$ pg_enc -m -k ~/.pgpoolkey -u pgpool -p
$ pg_enc -m -k ~/.pgpoolkey -u postgres -p
$ cat /etc/pgpool-II/pool_passwd
pgpool:AESrkH81/afiH7E6qaImGSQqw==
postgres:AESbPro4vs9cT622am0+RjU3Q==

# vi /etc/pgpool-II/escalation.sh
PGPOOLS=(zbx5-webdb01 zbx5-webdb02)
VIP=192.168.11.43
DEVICE=eth0

# vi /etc/sysconfig/pgpool
OPTS=" -D -n"

# systemctl start pgpool.service
```

< zbx5-webdb01だけの設定 >

zbx5-webdb02のDBを構築します。以下のような状態になったら構築完了です。

```
# pcp_recovery_node -h 192.168.11.43 -p 9898 -U pgpool -n 1
Password:
pcp_recovery_node -- Command Successful

# psql -h 192.168.11.43 -p 9999 -U pgpool postgres -c "show pool_nodes"
ユーザ pgpool のパスワード:
 node_id |   hostname   | port | status | lb_weight |  role   | select_cnt | load_balance_node | replication_delay | replicatio
n_state | replication_sync_state | last_status_change
---------+--------------+------+--------+-----------+---------+------------+-------------------+-------------------+-----------
--------+------------------------+---------------------
 0       | zbx5-webdb01 | 5432 | up     | 0.500000  | primary | 5163       | false             | 0                 |
        |                        | 2022-01-10 01:28:32
 1       | zbx5-webdb02 | 5432 | up     | 0.500000  | standby | 160        | true              | 0                 | streaming
        | async                  | 2022-01-10 01:32:36
(2 行)

```

# Pacemaker+Zabbixの設定

---

Pacemakerのインストールと設定

< 共通設定 >

```
# yum -y install pacemaker pcs
# passwd hacluster
# systemctl start pcsd
# systemctl enable pcsd
```

< zbx5-webdb01だけの設定 >

```
# pcs cluster auth zbx5-webdb01 zbx5-webdb02
# pcs cluster setup --name zabbix-cluster zbx5-webdb01 zbx5-webdb02
# pcs cluster start --all
# pcs cluster enable --all
# pcs cluster status
# pcs property set stonith-enabled=false
# pcs property set no-quorum-policy=ignore
# pcs resource defaults resource-stickiness=INFINITY
# pcs resource defaults migration-threshold=2
# pcs resource defaults
```

Zabbixのインストールと設定

< 共通設定 >

```
# yum -y install https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum -y install zabbix-server-pgsql zabbix-agent zabbix-agent2
# yum -y install centos-release-scl
# vi /etc/yum.repos.d/zabbix.repo
[zabbix-frontend]
enabled=1

# yum -y install zabbix-web-pgsql-scl zabbix-apache-conf-scl
```

< zbx5-webdb01だけの設定 >

```
# psql -h 192.168.11.43 -p 9999 -U pgpool postgres -c "show pool_nodes"
# sudo -u postgres createuser --pwprompt zabbix
# sudo -u postgres createdb -O zabbix zabbix
# zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

< 共通設定 >

```
# cp -p /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.bak
# diff -u /etc/zabbix/zabbix_server.conf.bak /etc/zabbix/zabbix_server.conf
DBHost=192.168.11.43
DBName=zabbix
DBPassword=zabbix
DBPort=9999
ProxyConfigFrequency=60
# scp -p /etc/zabbix/zabbix_server.conf root@zbx5-webdb02:/etc/zabbix/zabbix_server.conf

# vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Tokyo

# su - postgres
$ pg_enc -m -k ~/.pgpoolkey -u zabbix -p
$ cat /etc/pgpool-II/pool_passwd
pgpool:AESrkH81/afiH7E6qaImGSQqw==
postgres:AESbPro4vs9cT622am0+RjU3Q==
zabbix:AESKpmPAQPMz9iPjyQzYY5j+w==
```

< zbx5-webdb01だけの設定 >

一旦Zabbixを起動して、ブラウザから初期設定を実施します。

```
# systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

初期設定で完了したら停止し、初期設定ファイルをzbx5-webdb02へ転送します。

```
# systemctl stop zabbix-server zabbix-agent httpd rh-php72-php-fpm
# scp /etc/zabbix/web/zabbix.conf.php zbx5-webdb02:/etc/zabbix/web/zabbix.conf.php
```

< zbx5-webdb02だけの設定 >

```
# chown apache:apache /etc/zabbix/web/zabbix.conf.php
```

< 共通設定 >

pcsのリソースをモニタリングするのに、statusページを用意します。

```
# vi /etc/httpd/conf.d/status.conf
ExtendedStatus On

<Location /server-status>
    SetHandler server-status
    Require local
</Location>
```

< zbx5-webdb01だけの設定 >

pcsのリソースを作成します。

```
# pcs resource create zabbix-server systemd:zabbix-server op monitor interval="10s" timeout="15s" --group zabbix-group
# pcs resource create apache ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://127.0.0.1/server-status" op monitor interval=10s --group zabbix-group
# pcs resource create apache-vip ocf:heartbeat:IPaddr2 ip=192.168.11.44 cidr_netmask=24 op monitor interval=10s --group zabbix-group
# pcs resource create rh-php72-php-fpm systemd:rh-php72-php-fpm op monitor interval="10s" timeout="15s" --group zabbix-group

# pcs constraint colocation add zabbix-server with apache-vip INFINITY
# pcs constraint colocation add zabbix-server with apache INFINITY
# pcs constraint colocation add zabbix-server with rh-php72-php-fpm INFINITY

# pcs resource create pgpool-vip ocf:heartbeat:IPaddr2 ip=192.168.11.43 cidr_netmask=24
# pcs resource clone pgpool-vip

# pcs resource create pgpool-vip-ping ocf:pacemaker:ping dampen=5s multiplier=1000 host_list=192.168.11.43
# pcs resource clone pgpool-vip-ping
```

設定を確認します。

```
# pcs status
Cluster name: zabbix-cluster
Stack: corosync
Current DC: zbx5-webdb01 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Mon Jan 10 02:02:53 2022
Last change: Mon Jan 10 01:29:30 2022 by hacluster via crmd on zbx5-webdb01

2 nodes configured
8 resource instances configured

Online: [ zbx5-webdb01 zbx5-webdb02 ]

Full list of resources:

 Resource Group: zabbix-group
     zabbix-server      (systemd:zabbix-server):        Started zbx5-webdb01
     apache     (ocf::heartbeat:apache):        Started zbx5-webdb01
     rh-php72-php-fpm   (systemd:rh-php72-php-fpm):     Started zbx5-webdb01
     apache-vip (ocf::heartbeat:IPaddr2):       Started zbx5-webdb01
 Clone Set: pgpool-vip-clone [pgpool-vip]
     Started: [ zbx5-webdb01 zbx5-webdb02 ]
 Clone Set: pgpool-vip-ping-clone [pgpool-vip-ping]
     Started: [ zbx5-webdb01 zbx5-webdb02 ]

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

ここまで実施してapacheのVIPにしたアドレスでZabbix画面が表示できれば完了です。

# 最後に

---

ここまでやるのに何回かぶっ壊したりしています笑 一番苦戦したのはpgpoolの部分ですね。

4.2から色々パラメータが変わったので理解するのに苦労しました。 

また構築終わった後にPostgresのパラメータと睨めっこしてZabbixがちゃんと動くように真剣にパラメータ調整しました。 

多分ここまで設定を弄ったのは初めてかもしれません。

今回は自信がないのでconfの詳細は割愛させていただきます。 

自己学習でやっているので内容に間違いがあるような気がするので冒頭でも書いていますが参考程度にして下さい。