# Zabbix6.0 HA構成の考察と検証

![Zabbix_logo-RGB.png](Zabbix6%200%20HA%E6%A7%8B%E6%88%90%E3%81%AE%E8%80%83%E5%AF%9F%E3%81%A8%E6%A4%9C%E8%A8%BC%2009d695a82a6047f5bbe308aa9c7bb1cd/Zabbix_logo-RGB.png)

目次

# はじめに

---

Zabbix6.0からHA機能がでたので以下で検証をしていますがZabbixProxyを配置した際には、

Serverの指定を一つしかできないため構成の検討した結果になります。

[Zabbix6.0のHA構成（GaleraCluster）](Zabbix6%200%E3%81%AEHA%E6%A7%8B%E6%88%90%EF%BC%88GaleraCluster%EF%BC%89%20ff71ddfe98d34377b1e7eeaa10e75d25.md)

# Zabbix6.0のHAについて

---

HA構成をした場合、Active側では10051で受付できる状態になりStandby側では10051では受付できる状態ではありません。

```bash
# zabbix_server -R ha_status
Failover delay: 10 seconds
Cluster status:
   #  ID                        Name                      Address                        Status      Last Access
   1. clgdku4en0001kwwdvag9g3yf ray-zbx6-web02            192.168.11.102:10051           standby     1s
   2. clgdkuyqe000178zlbn17gs3o ray-zbx6-web01            192.168.11.101:10051           active      1s

root@ray-zbx6-web01:~# ss -an | grep 10051 | wc -l
284

root@ray-zbx6-web02:~# ss -an | grep 10051 | wc -l
0
```

またプロセスについては以下のようになります。

```bash
# systemctl status zabbix-server
● zabbix-server.service - Zabbix Server
     Loaded: loaded (/lib/systemd/system/zabbix-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-05-01 01:12:09 UTC; 1 day 8h ago
   Main PID: 1120 (zabbix_server)
      Tasks: 48 (limit: 4660)
     Memory: 253.8M
        CPU: 41min 26.669s
     CGroup: /system.slice/zabbix-server.service
             ├─1120 /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
             ├─1150 "/usr/sbin/zabbix_server: ha manager" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             ├─1211 "/usr/sbin/zabbix_server: service manager #1 [processed 0 events, updated 0 event tags, deleted 0 problems, synced 0 service updates, idle 5.019413 sec d>
             ├─1212 "/usr/sbin/zabbix_server: configuration syncer [synced configuration in 0.493845 sec, idle 60 sec]"
             ├─1229 "/usr/sbin/zabbix_server: alert manager #1 [sent 0, failed 0 alerts, idle 5.011746 sec during 5.011823 sec]"
             ├─1230 "/usr/sbin/zabbix_server: alerter #1 [sent 0, failed 0 alerts, idle 1154.936939 sec during 1155.560060 sec]"
             ├─1231 "/usr/sbin/zabbix_server: alerter #2 [sent 0, failed 0 alerts, idle 431.664857 sec during 432.279649 sec]"
             ├─1232 "/usr/sbin/zabbix_server: alerter #3 [sent 0, failed 0 alerts, idle 1153.068563 sec during 1153.710683 sec]"
             ├─1233 "/usr/sbin/zabbix_server: preprocessing manager #1 [queued 0, processed 86 values, idle 5.505399 sec during 5.511586 sec]"
             ├─1234 "/usr/sbin/zabbix_server: preprocessing worker #1 started" ""
             ├─1235 "/usr/sbin/zabbix_server: preprocessing worker #2 started" ""
             ├─1236 "/usr/sbin/zabbix_server: preprocessing worker #3 started" ""
             ├─1237 "/usr/sbin/zabbix_server: lld manager #1 [processed 1 LLD rules, idle 5.059929sec during 5.060007 sec]"
             ├─1238 "/usr/sbin/zabbix_server: lld worker #1 [processed 1 LLD rules, idle 15.089539 sec during 15.109795 sec]"
             ├─1239 "/usr/sbin/zabbix_server: lld worker #2 [processed 1 LLD rules, idle 14.084481 sec during 14.108809 sec]"
             ├─1240 "/usr/sbin/zabbix_server: housekeeper [deleted 0 hist/trends, 0 items/triggers, 0 events, 0 sessions, 0 alarms, 0 audit items, 0 records in 0.140608 sec,>
             ├─1241 "/usr/sbin/zabbix_server: timer #1 [updated 0 hosts, suppressed 0 events in 0.000947 sec, idle 59 sec]"
             ├─1242 "/usr/sbin/zabbix_server: http poller #1 [got 0 values in 0.000837 sec, idle 5 sec]"
             ├─1243 "/usr/sbin/zabbix_server: discoverer #1 [processed 0 rules in 0.001542 sec, idle 60 sec]"
             ├─1244 "/usr/sbin/zabbix_server: history syncer #1 [processed 0 values, 0 triggers in 0.000009 sec, idle 1 sec]"
             ├─1245 "/usr/sbin/zabbix_server: history syncer #2 [processed 0 values, 0 triggers in 0.000017 sec, idle 1 sec]"
             ├─1246 "/usr/sbin/zabbix_server: history syncer #3 [processed 0 values, 0 triggers in 0.000015 sec, idle 1 sec]"
             ├─1247 "/usr/sbin/zabbix_server: history syncer #4 [processed 52 values, 29 triggers in 0.067427 sec, idle 1 sec]"
             ├─1248 "/usr/sbin/zabbix_server: escalator #1 [processed 0 escalations in 0.003060 sec, idle 3 sec]"
             ├─1249 "/usr/sbin/zabbix_server: proxy poller #1 [exchanged data with 1 proxies in 0.002756 sec, idle 1 sec]"
             ├─1250 "/usr/sbin/zabbix_server: self-monitoring [processed data in 0.000023 sec, idle 1 sec]"
             ├─1251 "/usr/sbin/zabbix_server: task manager [processed 0 task(s) in 0.000542 sec, idle 5 sec]"
             ├─1252 "/usr/sbin/zabbix_server: poller #1 [got 0 values in 0.000039 sec, idle 5 sec]"
             ├─1253 "/usr/sbin/zabbix_server: poller #2 [got 0 values in 0.000037 sec, idle 5 sec]"
             ├─1254 "/usr/sbin/zabbix_server: poller #3 [got 0 values in 0.000022 sec, idle 5 sec]"
             ├─1255 "/usr/sbin/zabbix_server: poller #4 [got 0 values in 0.000018 sec, idle 5 sec]"
             ├─1256 "/usr/sbin/zabbix_server: poller #5 [got 0 values in 0.000025 sec, idle 5 sec]"
             ├─1257 "/usr/sbin/zabbix_server: unreachable poller #1 [got 0 values in 0.000031 sec, idle 5 sec]"
             ├─1258 "/usr/sbin/zabbix_server: trapper #1 [processed data in 0.010117 sec, waiting for connection]"
             ├─1259 "/usr/sbin/zabbix_server: trapper #2 [processed data in 0.000004 sec, waiting for connection]"
             ├─1260 "/usr/sbin/zabbix_server: trapper #3 [processed data in 0.001561 sec, waiting for connection]"
             ├─1261 "/usr/sbin/zabbix_server: trapper #4 [processed data in 0.001690 sec, waiting for connection]"
             ├─1262 "/usr/sbin/zabbix_server: trapper #5 [processed data in 0.187784 sec, waiting for connection]"
             ├─1263 "/usr/sbin/zabbix_server: icmp pinger #1 [got 0 values in 0.000019 sec, idle 5 sec]"
             ├─1264 "/usr/sbin/zabbix_server: alert syncer [queued 0 alerts(s), flushed 0 result(s) in 0.001999 sec, idle 1 sec]"
             ├─1265 "/usr/sbin/zabbix_server: history poller #1 [got 0 values in 0.000025 sec, idle 1 sec]"
             ├─1266 "/usr/sbin/zabbix_server: history poller #2 [got 3 values in 0.094188 sec, idle 1 sec]"
             ├─1267 "/usr/sbin/zabbix_server: history poller #3 [got 4 values in 0.097644 sec, idle 1 sec]"
             ├─1268 "/usr/sbin/zabbix_server: history poller #4 [got 5 values in 0.112005 sec, idle 1 sec]"
             ├─1269 "/usr/sbin/zabbix_server: history poller #5 [got 3 values in 0.092138 sec, idle 1 sec]"
             ├─1270 "/usr/sbin/zabbix_server: availability manager #1 [queued 0, processed 0 values, idle 5.017609 sec during 5.017691 sec]"
             ├─1271 "/usr/sbin/zabbix_server: trigger housekeeper [deleted 0 problems records in 0.001982 sec, idle for 60 second(s)]"
             └─1272 "/usr/sbin/zabbix_server: odbc poller #1 [got 0 values in 0.000024 sec, idle 5 sec]"

May 01 01:12:09 ray-zbx6-web01 systemd[1]: Starting Zabbix Server...
May 01 01:12:09 ray-zbx6-web01 systemd[1]: Started Zabbix Server.

root@ray-zbx6-web02:~# systemctl status zabbix-server
● zabbix-server.service - Zabbix Server
     Loaded: loaded (/lib/systemd/system/zabbix-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-05-01 01:12:28 UTC; 1 day 8h ago
   Main PID: 1099 (zabbix_server)
      Tasks: 2 (limit: 4660)
     Memory: 9.6M
        CPU: 20.694s
     CGroup: /system.slice/zabbix-server.service
             ├─1099 /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
             └─1102 "/usr/sbin/zabbix_server: ha manager" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""

 5月 01 01:12:27 ray-zbx6-web02 systemd[1]: Starting Zabbix Server...
 5月 01 01:12:28 ray-zbx6-web02 systemd[1]: Started Zabbix Server.
```

Zabbix6.0のHA機能があるにも関わらずPacemakerなどのHAクラスタリングを組むのは、

今までの冗長構成と何も変わらなくなってしまうので、Keepalivedと自作チェックツールを駆使して冗長構成を取ります。

※OSはUbuntu22.04になります。

# Keepalivedのインストール&設定

---

以下コマンドでインストールします。

```bash
# apt install keepalived
```

自作チェックツールをいくつか作成し、/etc/keepalivedに保存します。（権限与えるの忘れないように）

①ZabbixServerチェック

```bash
# cat check_zabbix.sh 
#!/usr/bin/bash

timeout 3 curl -v http://localhost:10051
status=$?

#10051につなげる場合には52が応答
if [ $status -eq 52 ]; then
  logger "zabbix-server processes are alive."
  exit 0
#10051につなげない場合には7が応答
elif [ $status -eq 7 ]; then
  logger "zabbix-server process that is hanging up."
  exit 1
else
  logger "Something is wrong."
  exit 1
fi
```

②Apacheチェック

```bash
# cat check_apache2.sh 
#!/usr/bin/bash

timeout 3 curl -v http://localhost
status=$?

#80につなげる場合には0が応答
if [ $status -eq 0 ]; then
  logger "apache2 processes are alive."
  exit 0
#80につなげない場合には7が応答
elif [ $status -eq 7 ]; then
  logger "apache2 process that is hanging up."
  exit 1
else
  logger "Something is wrong."
  exit 1
fi
```

/etc/keepalived/keepalived.confを作成します。

- vrrp_scriptの中身は適時変更してください。
- vrrp_instance VI_1のStateはMASTER/BACKUPと記載できますが、priorityの優先順序に準拠になります。（あまり意味ない）
- interfaceはVIPを割り当てるインターフェースを指定して下さい。
- virtual_router_idはActiveとStandby両方同じ番号を振って下さい。
- priorityはActiveにした方の数字を小さくしてください。
- advert_intはVRRPの広告パケット送信間隔です。
- virtual_ipaddressは割り当てるVIPになります。
- 最後にVIPを割り当てる際に監視する内容をtrack_scriptを使います。

```bash
global_defs {
}

vrrp_script chk_zabbix {
    script "/etc/keepalived/check_zabbix.sh"
    interval 2
    fall 2
    rise 2
}

vrrp_script chk_apache2 {
    script "/etc/keepalived/check_apache2.sh"
    interval 2
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 1
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.11.103/24 dev eth0
    }
    track_script {
      chk_zabbix
      chk_apache2
    }
}
```

# Keepalivedの起動を動作確認

---

Active/Standby側でkeepalivedを起動します。

```bash
# systemctl --now enable keepalived
```

ZabbixServerAvtive側の動作確認します。

```bash
# systemctl status keepalived
● keepalived.service - Keepalive Daemon (LVS and VRRP)
     Loaded: loaded (/lib/systemd/system/keepalived.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-05-02 10:09:40 UTC; 12min ago
   Main PID: 385491 (keepalived)
      Tasks: 2 (limit: 4660)
     Memory: 2.0M
        CPU: 5.567s
     CGroup: /system.slice/keepalived.service
             ├─385491 /usr/sbin/keepalived --dont-fork
             └─385492 /usr/sbin/keepalived --dont-fork

May 02 10:21:58 ray-zbx6-web01 root[389317]: zabbix-server processes are alive.
May 02 10:22:00 ray-zbx6-web01 root[389325]: zabbix-server processes are alive.
May 02 10:22:02 ray-zbx6-web01 root[389337]: zabbix-server processes are alive.
May 02 10:22:02 ray-zbx6-web01 root[389338]: apache2 processes are alive.
May 02 10:22:04 ray-zbx6-web01 root[389346]: zabbix-server processes are alive.
May 02 10:22:08 ray-zbx6-web01 root[389361]: zabbix-server processes are alive.
May 02 10:22:08 ray-zbx6-web01 root[389362]: apache2 processes are alive.
May 02 10:22:10 ray-zbx6-web01 root[389370]: apache2 processes are alive.
May 02 10:22:12 ray-zbx6-web01 root[389378]: zabbix-server processes are alive.
May 02 10:22:16 ray-zbx6-web01 root[389424]: apache2 processes are alive.

# ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.11.101/24 brd 192.168.11.255 scope global eth0
    inet 192.168.11.103/24 scope global secondary eth0
    inet6 fe80::b440:afff:fed1:8840/64 scope link
```

ZabbixServerStandby側の動作確認します。

```bash
root@ray-zbx6-web02:~#  systemctl status keepalived
● keepalived.service - Keepalive Daemon (LVS and VRRP)
     Loaded: loaded (/lib/systemd/system/keepalived.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-05-02 10:15:10 UTC; 9min ago
   Main PID: 387634 (keepalived)
      Tasks: 2 (limit: 4660)
     Memory: 2.0M
        CPU: 4.146s
     CGroup: /system.slice/keepalived.service
             ├─387634 /usr/sbin/keepalived --dont-fork
             └─387635 /usr/sbin/keepalived --dont-fork

 5月 02 10:24:06 ray-zbx6-web02 root[390473]: apache2 processes are alive.
 5月 02 10:24:10 ray-zbx6-web02 root[390491]: apache2 processes are alive.
 5月 02 10:24:24 ray-zbx6-web02 root[390548]: apache2 processes are alive.
 5月 02 10:24:26 ray-zbx6-web02 root[390555]: apache2 processes are alive.
 5月 02 10:24:30 ray-zbx6-web02 root[390600]: apache2 processes are alive.
 5月 02 10:24:36 ray-zbx6-web02 root[390625]: zabbix-server process that is hanging up.
 5月 02 10:24:40 ray-zbx6-web02 root[390642]: zabbix-server process that is hanging up.
 5月 02 10:24:46 ray-zbx6-web02 root[390667]: zabbix-server process that is hanging up.
 5月 02 10:24:52 ray-zbx6-web02 root[390694]: zabbix-server process that is hanging up.
 5月 02 10:24:54 ray-zbx6-web02 root[390706]: apache2 processes are alive.

# ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.11.102/24 brd 192.168.11.255 scope global eth0
    inet6 fe80::91:8fff:fe1b:cc80/64 scope link
```

# Keepalivedの切り替わりテスト

---

ZabbixServerを停止して、VIPの切り替わりが発生するか確認します。

```bash
root@ray-zbx6-web01:~# systemctl stop zabbix-server

root@ray-zbx6-web01:~# ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.11.101/24 brd 192.168.11.255 scope global eth0
    inet6 fe80::b440:afff:fed1:8840/64 scope link

root@ray-zbx6-web02:~# ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.11.102/24 brd 192.168.11.255 scope global eth0
    inet 192.168.11.103/24 scope global secondary eth0
    inet6 fe80::91:8fff:fe1b:cc80/64 scope link

root@ray-zbx6-web02:~# zabbix_server -R ha_status
Failover delay: 10 seconds
Cluster status:
   #  ID                        Name                      Address                        Status      Last Access
   1. clgdku4en0001kwwdvag9g3yf ray-zbx6-web02            192.168.11.102:10051           active      3s
   2. clgdkuyqe000178zlbn17gs3o ray-zbx6-web01            192.168.11.101:10051           stopped     1m 43s

最後にZabbixServerのUIアクセス（http://192.168.11.103/zabbix）して、問題ないことを確認できました。
```