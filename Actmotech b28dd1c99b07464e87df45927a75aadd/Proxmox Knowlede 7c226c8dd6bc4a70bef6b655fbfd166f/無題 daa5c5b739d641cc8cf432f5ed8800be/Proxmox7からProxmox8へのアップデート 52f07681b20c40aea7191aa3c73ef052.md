# Proxmox7からProxmox8へのアップデート

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

目次

# はじめに

---

Proxmox8が公開されていたいので、Proxmox7から8へのアップデートをメンテナンス時間を確保した上で実施しました。

参考にしたサイトは以下となります。

[Upgrade from 7 to 8 - Proxmox VE](https://www.notion.so/f79249c5c00e49c6b04b67cafc9bd924?pvs=21) 

# 前提

---

仮想マシンはすべて停止した状態にしてチェックを実施します。

```bash
root@ray-hapve01:~# pve7to8 -full
= CHECKING VERSION INFORMATION FOR PVE PACKAGES =

Checking for package updates..
PASS: all packages up-to-date

Checking proxmox-ve package version..
PASS: proxmox-ve package has version >= 7.4-1

Checking running kernel version..
PASS: running kernel '5.15.126-1-pve' is considered suitable for upgrade.

= CHECKING CLUSTER HEALTH/SETTINGS =

PASS: systemd unit 'pve-cluster.service' is in state 'active'
PASS: systemd unit 'corosync.service' is in state 'active'
PASS: Cluster Filesystem is quorate.

Analzying quorum settings and state..
INFO: configured votes - nodes: 4
INFO: configured votes - qdevice: 0
INFO: current expected votes: 4
INFO: current total votes: 4

Checking nodelist entries..
PASS: nodelist settings OK

Checking totem settings..
PASS: totem settings OK

INFO: run 'pvecm status' to get detailed cluster status..

= CHECKING HYPER-CONVERGED CEPH STATUS =

SKIP: no hyper-converged ceph setup detected!

= CHECKING CONFIGURED STORAGES =

PASS: storage 'local' enabled and active.
PASS: storage 'local-lvm' enabled and active.
PASS: storage 'qnap-haproxmox-backup' enabled and active.
PASS: storage 'synology-haproxmox2' enabled and active.
INFO: Checking storage content type configuration..
PASS: no storage content problems found
PASS: no storage re-uses a directory for multiple content types.

= MISCELLANEOUS CHECKS =

INFO: Checking common daemon services..
PASS: systemd unit 'pveproxy.service' is in state 'active'
PASS: systemd unit 'pvedaemon.service' is in state 'active'
PASS: systemd unit 'pvescheduler.service' is in state 'active'
PASS: systemd unit 'pvestatd.service' is in state 'active'
INFO: Checking for supported & active NTP service..
PASS: Detected active time synchronisation unit 'chrony.service'
INFO: Checking for running guests..
PASS: no running guest detected.
INFO: Checking if the local node's hostname 'ray-hapve01' is resolvable..
INFO: Checking if resolved IP is configured on local node..
PASS: Resolved node IP '192.168.11.21' configured and active on single interface.
INFO: Check node certificate's RSA key size
PASS: Certificate 'pve-root-ca.pem' passed Debian Busters (and newer) security level for TLS connections (4096 >= 2048)
PASS: Certificate 'pve-ssl.pem' passed Debian Busters (and newer) security level for TLS connections (2048 >= 2048)
PASS: Certificate 'pveproxy-ssl.pem' passed Debian Busters (and newer) security level for TLS connections (4096 >= 2048)
INFO: Checking backup retention settings..
PASS: no backup retention problems found.
INFO: checking CIFS credential location..
PASS: no CIFS credentials at outdated location found.
INFO: Checking permission system changes..
INFO: Checking custom role IDs for clashes with new 'PVE' namespace..
PASS: none of the 1 custom roles will clash with newly enforced 'PVE' namespace
INFO: Checking if LXCFS is running with FUSE3 library, if already upgraded..
SKIP: not yet upgraded, no need to check the FUSE library version LXCFS uses
INFO: Checking node and guest description/note length..
PASS: All node config descriptions fit in the new limit of 64 KiB
PASS: All guest config descriptions fit in the new limit of 8 KiB
INFO: Checking container configs for deprecated lxc.cgroup entries
PASS: No legacy 'lxc.cgroup' keys found.
INFO: Checking if the suite for the Debian security repository is correct..
PASS: found no suite mismatch
INFO: Checking for existence of NVIDIA vGPU Manager..
PASS: No NVIDIA vGPU Service found.
INFO: Checking bootloader configuration...
SKIP: not yet upgraded, no need to check the presence of systemd-boot
INFO: Check for dkms modules...
SKIP: could not get dkms status
SKIP: No containers on node detected.

= SUMMARY =

TOTAL:    37
PASSED:   32
SKIPPED:  5
WARNINGS: 0
FAILURES: 0
```

WARNやFAILが0であることを確認します。

# Proxmox7から8へアップデート

---

現行バージョンでアップデート漏れがないか確認をします。

```jsx
# apt update
All packages are up to date.

# apt dist-upgrade
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

#PVEが7.4-15以上になっていることを最終確認します。
# pveversion
pve-manager/7.4-17/513c62be (running kernel: 5.15.126-1-pve)

#サブスクリプションなしのリポジトリについては、パッケージリポジトリを参照
sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list

#Cephパッケージリポジトリの更新
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list

#パッケージ情報の更新
apt update 

#Proxmox VE 8.0にアップグレード
apt dist-upgrade
→いくつか質問がでてくるが質問内容は自分が利用している機能に応じて答えていきます。

#最後に再起動を実施します。
# reboot
```

再起動完了後にProxomoxのUIへログインし、バージョンが8になっていることを確認します。