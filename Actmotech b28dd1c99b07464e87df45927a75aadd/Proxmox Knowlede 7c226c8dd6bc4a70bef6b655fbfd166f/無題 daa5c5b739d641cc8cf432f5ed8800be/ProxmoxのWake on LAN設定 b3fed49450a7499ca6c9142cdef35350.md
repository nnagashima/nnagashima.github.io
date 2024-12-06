# ProxmoxのWake on LAN設定

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

目次

# はじめに

---

Proxmoxが万が一DOWNしてしまった際に、リモート先の踏み台から起動できるようにしたいので、

Wake on LANの設定を実施しました。

# 前提

---

- 機種：Desk mini x300
- OS：Proxmox VE 8

# BIOS設定

---

起動する際にDELボタンをクリックしてBIOS画面を表示させます。

- ACPI Configurationを選択＞ PCI Devices Power Onを有効化

# Proxmox設定

---

## 其1 Proxmoxにログインして、MACアドレスを調べます。(太字がOnboard LANです）

---

```bash
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UP group default qlen 1000
    link/ether **a8:a1:59:67:f4:fb** brd ff:ff:ff:ff:ff:ff
3: enx1cc035040466: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr1 state UP group default qlen 1000
    link/ether 1c:c0:35:04:04:66 brd ff:ff:ff:ff:ff:ff
4: tailscale0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1280 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none 
    inet 100.112.104.50/32 scope global tailscale0
       valid_lft forever preferred_lft forever
    inet6 fd7a:115c:a1e0::4a01:6832/128 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::218:f5d9:2d23:faa0/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
5: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether a8:a1:59:67:f4:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.21/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::aaa1:59ff:fe67:f4fb/64 scope link 
       valid_lft forever preferred_lft forever
6: vmbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 1c:c0:35:04:04:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.21.21/24 scope global vmbr1
       valid_lft forever preferred_lft forever
    inet6 fe80::1ec0:35ff:fe04:466/64 scope link 
       valid_lft forever preferred_lft forever
```

Proxmoxの管理画面に移動し、対象ノードのオプションをクリックして、調べたMACアドレスを「Wake on LAN用MACアドレス」の欄に入力します。

## 其2 NICのWake on LANを有効化します。

---

ethtoolがインストールされていなければ、インストールします。

```bash
# apt install ethtool
```

ethtoolを利用して、Wake on LANが有効化されているか確認します。

Wake-on: gであれば有効ですが、dの場合は無効となります。

```bash
# ethtool enp1s0 | grep -i wake
Supports Wake-on: pumbg
Wake-on: d
```

コマンドで有効化することこもできますが、有効になるのが次回起動時のみとなるため恒久的にはなりません。

そのため起動時に必ず設定されるようにCRONに記載して一度OS再起動をし、Wake on LANが有効化されているか確認します。

```bash
 # crontab -e
 @reboot /usr/sbin/ethtool -s enp1s0 wol g 
```

# 動作確認

---

一度Proxmoxをシャットダウンを実施して、Wake on LANを実行して起動するか確認します。

Clusterを組んでいる場合には、別のProxmoxから対象ノードを右クリックでWake on LANをクリックすることで起動できます。

Clusterを組んでいない場合にはWindows限定ですが、[nWOL](https://n-archives.net/software/nwol/)というツールがあります。

私はProxmox以外にもWake on LANしたいものがあるので、nWOLを利用して起動しています。