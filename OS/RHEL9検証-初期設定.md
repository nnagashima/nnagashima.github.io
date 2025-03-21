# RHEL9検証 ④初期設定

RHEL9における主な変更点についてはRHにて提供されているのでページを参照してください。

[第1章 概要 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/9.0_release_notes/overview)

# 目次
- [RHEL9検証 ④初期設定](#rhel9検証-初期設定)
- [目次](#目次)
- [1. SSH接続と設定について](#1-ssh接続と設定について)
- [2. Hostname変更](#2-hostname変更)
- [3. Firewall設定](#3-firewall設定)
- [4. OS言語設定](#4-os言語設定)
- [5. TimeZone設定](#5-timezone設定)
- [6. SELinuxの設定](#6-selinuxの設定)
- [7. システムの最新化](#7-システムの最新化)
- [8. ユーザーの追加](#8-ユーザーの追加)
- [9. SUDOの設定](#9-sudoの設定)
- [10. Networkの設定](#10-networkの設定)
- [11. NTP設定](#11-ntp設定)
- [12. NFSクライアント](#12-nfsクライアント)
- [13. 主要PKGのバージョンについて](#13-主要pkgのバージョンについて)
- [14. /var/log配下のログ運用](#14-varlog配下のログ運用)
    - [上記までしなくてもいいけど、毎日ログローテートさせたい場合](#上記までしなくてもいいけど毎日ログローテートさせたい場合)
- [15. boot.logの出力](#15-bootlogの出力)

# 1. SSH接続と設定について

---

RHEL9よりデフォルトでSHA1認証が無効化されたことによる影響で、現時点でTerratermでログインすることができません。

※RHEL9だけでなくUbutu22.04でも同じことが発生します。

対処方法としては以下となります。

- Terraterm 5.x以上を使えるなら、RSA形式でも接続できます。
- Terraterm以外のクライアントソフトを使ってログインする
- 古いTerratermで接続したいのであればcrypto-policesのデフォルトをSHA1にする（セキュリティレベルが下がる）

またsshの設定ですが、RHEL9のSSHバージョンからは、/etc/ssh/sshd_configに追加設定することを推奨としていません。
追加設定がある場合には、/etc/ssh/sshd_config.d/配下に設定ファイルを新規作成してください。
本ディレクトリの設定ファイル読み込み順は辞書式順序になるので設定を優先したい場合には、
本ディレクトリにある設定ファイルの番号より先の番号にしてあげた後、sshd -Tコマンドで設定が反映できているか確認してください。
例：40-custom.confなど

# 2. Hostname変更

---

```bash
# hostnamectl set-hostname hostname
# uname -n
hostname
```

# 3. Firewall設定

---

firewalld を使用した ipset でアクセスの可否をリスト管理していましたが、

RHEL9 から ipset が廃止になったという事なので nftables を利用します。

```bash
# cat /etc/firewalld/firewalld.conf | grep nftables
#       - nftables (default)
FirewallBackend=nftables
```

Firewallを利用しない場合には以下コマンドで停止と自動起動をOFFしてください。

```bash
# systemctl disable --now firewalld
```

Firewallを利用する場合にはFirewalldを停止して、nftablesサービスを起動してください。

```bash
# systemctl disable --now firewalld
# systemctl enable --now nftables
```

# 4. OS言語設定

---

localectlで確認します。

```bash
# localectl status
   System Locale: LANG=ja_JP.UTF-8
       VC Keymap: jp
      X11 Layout: jp

# localectl list-locales
C.UTF-8
ja_JP.UTF-8

変更する場合のコマンド例
# localectl set-locale LANG=ja_JP.utf8

日本語PKGがない場合
# dnf isntall glibc-langpack-ja
```

# 5. TimeZone設定

---

timedatectlで確認します。

```bash
# timedatectl 
               Local time: 月 2023-02-06 14:58:43 JST
           Universal time: 月 2023-02-06 05:58:43 UTC
                 RTC time: 月 2023-02-06 05:58:43
                Time zone: Asia/Tokyo (JST, +0900)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

変更する場合のコマンド例
# timedatectl set-timezone Asia/Tokyo
```

# 6. SELinuxの設定

---

従来だと

```
/etc/selinux/config
SELINUX=enforcing
を
SELINUX=disabled
```

にしてOS再起動という流れになりますよね。

RHEL9に関わらず、RHEL9系のOSは今後上記で無効化をしてはいけません。 

そもそも無効化するなとOS作っている側から出ているようです。

じゃあ無効化する方法はないかといったらそうではなく、 RHEL9系の/etc/selinux/configは以下のようになっています。

```
# cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
# grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
# grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
# targeted - Targeted processes are protected,
# minimum - Modification of targeted policy. Only selected processes are protected.
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

SELINUX の無効化はコメントで書かれているように grubby コマンドで行います。

```
grubby --update-kernel ALL --args selinux=0
```

実施すると/boot/loader/entries/にあるconfにselinux=0が反映されます。

```
# cat /boot/loader/entries/d59b51d40a45461f801c17887f8c5e08-5.14.0-162.6.1.el9_1.0.1.x86_64.conf
title Rocky Linux (5.14.0-162.6.1.el9_1.0.1.x86_64) 9.1 (Blue Onyx)
version 5.14.0-162.6.1.el9_1.0.1.x86_64
linux /vmlinuz-5.14.0-162.6.1.el9_1.0.1.x86_64
initrd /initramfs-5.14.0-162.6.1.el9_1.0.1.x86_64.img
options root=/dev/mapper/rl-root ro resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap selinux=0
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky
```

SELinuxの再有効化は 以下のコマンドになります。

```
grubby --update-kernel ALL --remove-args selinux
```

勿論設定後はOS再起動しないと反映されませんので、ご注意を。

ためしに/etc/selinux/configで設定してみたら。。。

OSが壊れて起動しなくなりました。。。

なので皆さん今後RockyLinux9でSELinuxを触る際には気をつけてください。


# 7. システムの最新化

---

インストールやアップデートする際にはyumでもdnfでもどちらでも大丈夫そうです。

```bash
# ls -la /usr/bin/dnf
lrwxrwxrwx. 1 root root 5  1月 31 10:01 /usr/bin/dnf -> dnf-3

# ls -la /usr/bin/yum
lrwxrwxrwx. 1 root root 5  1月 31 10:01 /usr/bin/yum -> dnf-3

# dnf upgrade
```

# 8. ユーザーの追加

---

useraddでユーザー名を追加、ユーザーでSSHでログインするにあたりパスワードも設定します。

```bash
# useradd username
# passwd username
```

# 9. SUDOの設定

---

追加したユーザーにroot 権限を付与したい場合にはvisudoで変更します。

```bash
# visudo

##root権限を全て利用
username  ALL=(ALL)       ALL

##root権限をパスワードなしで利用
username  ALL=NOPASSWD: ALL
```

# 10. Networkの設定

---

RHEL9からNetworkMangerを使用し、/etc/sysconfig/network-scriptsによるネットワーク設定が廃止されました。

```bash
デバイス確認
# nmcli device

IPv4アドレス設定
# nmcli connection modify interface-name ipv4.addresses xxx.xxx.xxx.xxx/xx

ゲートウェイ設定
# nmcli connection modify interface-name ipv4.gateway xxx.xxx.xxx.xxx

DNS設定
# nmcli connection modify interface-name ipv4.dns xxx.xxx.xxx.xxx

DHCP無効
# nmcli connection modify interface-name ipv4.method manual

設定確認
# nmcli device show interface-name 

インターフェース再起動
# nmcli connection down interface-name && nmcli connection up interface-name

IPv6無効化
# vi /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
# sysctl -p
```

# 11. NTP設定

---

Chronyを利用します。

ibrustを使用することで初回同期にかかる時間を改善することができます。

時刻同期する際、poolで設定していますがserverでも設定は可能です。

```bash
de# cat /etc/chrony.conf
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
pool 2.rhel.pool.ntp.org iburst #同期したいNTPサーバ指定

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3　#1秒以上のズレが3回あった場合、slewモードにかわりstepによる時刻調整する

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16 #NTPクライアントからの接続を制限する場合の設定

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Get TAI-UTC offset and leap seconds from the system tz database.
leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony #ログ保管ディレクトリを指定

# Select which information is logged.
#log measurements statistics tracking #取得するログの種類を指定

# RemoteHostからchronycなど確認をさせないようにします。
cmddeny all

設定したらChronyの再起動を実施
# systemctl restart chronyd

動作確認
# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 180.43.145.38                 0   7     0     -     +0ns[   +0ns] +/-    0ns
```

# 12. NFSクライアント

---

NFS共有マウントの方法はNFSクライアントを利用します。

```bash
# dnf install nfs-utilis
# mount -t nfs ipaddress:/NFSサーバのマウントパス /マウントディレクトリ
# df -h
ipaddress:/NFSサーバのマウントパス   5.4T  1.2T  4.2T   23% /マウントディレクトリ
```

システム起動時にマウントしたい場合には/etc/fstabに記載します。

```bash
ipaddress:/NFSサーバのマウントパス /マウントディレクトリ               nfs     defaults        0 0
```

マウントポイントアクセス時にマントする場合には以下のようになります。

```bash
# dnf -y install autofs
# vi /etc/auto.master
/-    /etc/auto.mount

# vi /etc/auto.mount
/マウントパス   -fstype=nfs,rw  ipaddress:/NFSサーバのマウントパス

# systemctl enable --now autofs
# cd /mnt
# dh -h
ipaddress:/NFSサーバのマウントパス   5.4T  1.2T  4.2T   23% /マウントディレクトリ
```

# 13. 主要PKGのバージョンについて

---

```bash
■OPENSSH
# rpm -qa | grep openssh
openssh-8.7p1-24.el9_1.x86_64
openssh-clients-8.7p1-24.el9_1.x86_64
openssh-server-8.7p1-24.el9_1.x86_64

■Apahce
# dnf list all | grep httpd
httpd.x86_64                                         2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
httpd-core.x86_64                                    2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
httpd-devel.x86_64                                   2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
httpd-filesystem.noarch                              2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
httpd-manual.noarch                                  2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
httpd-tools.x86_64                                   2.4.53-7.el9                       rhel-9-for-x86_64-appstream-rpms 
keycloak-httpd-client-install.noarch                 1.1-10.el9                         rhel-9-for-x86_64-appstream-rpms 
libmicrohttpd.i686                                   1:0.9.72-4.el9                     rhel-9-for-x86_64-appstream-rpms 
libmicrohttpd.x86_64                                 1:0.9.72-4.el9                     rhel-9-for-x86_64-appstream-rpms 
python3-keycloak-httpd-client-install.noarch         1.1-10.el9                         rhel-9-for-x86_64-appstream-rpms 
redhat-logos-httpd.noarch                            90.4-1.el9                         rhel-9-for-x86_64-appstream-rpms

■PHP
# dnf list all | grep php
php.x86_64                                           8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-bcmath.x86_64                                    8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-cli.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-common.x86_64                                    8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-dba.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-dbg.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-devel.x86_64                                     8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-embedded.x86_64                                  8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-enchant.x86_64                                   8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-ffi.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-fpm.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-gd.x86_64                                        8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-gmp.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-intl.x86_64                                      8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-ldap.x86_64                                      8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-mbstring.x86_64                                  8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-mysqlnd.x86_64                                   8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-odbc.x86_64                                      8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-opcache.x86_64                                   8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-pdo.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-pear.noarch                                      1:1.10.13-1.el9                    rhel-9-for-x86_64-appstream-rpms 
php-pecl-apcu.x86_64                                 5.1.20-5.el9                       rhel-9-for-x86_64-appstream-rpms 
php-pecl-apcu-devel.x86_64                           5.1.20-5.el9                       rhel-9-for-x86_64-appstream-rpms 
php-pecl-rrd.x86_64                                  2.0.3-3.el9                        rhel-9-for-x86_64-appstream-rpms 
php-pecl-xdebug3.x86_64                              3.1.2-1.el9                        rhel-9-for-x86_64-appstream-rpms 
php-pecl-zip.x86_64                                  1.19.2-6.el9                       rhel-9-for-x86_64-appstream-rpms 
php-pgsql.x86_64                                     8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-process.x86_64                                   8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-snmp.x86_64                                      8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-soap.x86_64                                      8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms 
php-xml.x86_64                                       8.0.20-3.el9                       rhel-9-for-x86_64-appstream-rpms

■Mariadb
# dnf list all | grep mariadb
mariadb.x86_64                                       3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-backup.x86_64                                3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-common.x86_64                                3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-c.i686                             3.2.6-1.el9_0                      rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-c.x86_64                           3.2.6-1.el9_0                      rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-c-config.noarch                    3.2.6-1.el9_0                      rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-c-devel.i686                       3.2.6-1.el9_0                      rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-c-devel.x86_64                     3.2.6-1.el9_0                      rhel-9-for-x86_64-appstream-rpms 
mariadb-connector-odbc.x86_64                        3.1.12-3.el9                       rhel-9-for-x86_64-appstream-rpms 
mariadb-embedded.x86_64                              3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-errmsg.x86_64                                3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-gssapi-server.x86_64                         3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-java-client.noarch                           3.0.3-1.el9                        rhel-9-for-x86_64-appstream-rpms 
mariadb-oqgraph-engine.x86_64                        3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-pam.x86_64                                   3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-server.x86_64                                3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-server-galera.x86_64                         3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms 
mariadb-server-utils.x86_64                          3:10.5.16-2.el9_0                  rhel-9-for-x86_64-appstream-rpms

■NFS
# dnf list all | grep nfs
libnfsidmap.i686                                     1:2.5.4-15.el9                     rhel-9-for-x86_64-baseos-rpms    
libnfsidmap.x86_64                                   1:2.5.4-15.el9                     rhel-9-for-x86_64-baseos-rpms    
libstoragemgmt-nfs-plugin.x86_64                     1.9.3-1.el9                        rhel-9-for-x86_64-appstream-rpms 
nfs-utils.x86_64                                     1:2.5.4-15.el9                     rhel-9-for-x86_64-baseos-rpms    
nfs-utils-coreos.x86_64                              1:2.5.4-15.el9                     rhel-9-for-x86_64-appstream-rpms 
nfs4-acl-tools.x86_64                                0.3.5-8.el9                        rhel-9-for-x86_64-baseos-rpms    
nfsv4-client-utils.x86_64                            1:2.5.4-15.el9                     rhel-9-for-x86_64-appstream-rpms 
pcp-pmda-nfsclient.x86_64                            5.3.7-7.el9                        rhel-9-for-x86_64-appstream-rpms 
sssd-nfs-idmap.x86_64                                2.7.3-4.el9_1.3                    rhel-9-for-x86_64-baseos-rpms    
texlive-mfnfss.noarch                                9:20200406-25.el9                  rhel-9-for-x86_64-appstream-rpms 
texlive-psnfss.noarch                                9:20200406-25.el9                  rhel-9-for-x86_64-appstream-rpms

■Python
# dnf list all | grep python3.x86_64
libcap-ng-python3.x86_64                             0.8.2-7.el9                        @rhel-9-for-x86_64-appstream-rpms
python3.x86_64                                       3.9.14-1.el9_1.1                   @rhel-9-for-x86_64-baseos-rpms   
boost-python3.x86_64                                 1.75.0-8.el9                       rhel-9-for-x86_64-appstream-rpms 
graphviz-python3.x86_64                              2.44.0-25.el9                      rhel-9-for-x86_64-appstream-rpms 
libpeas-loader-python3.x86_64                        1.30.0-4.el9                       rhel-9-for-x86_64-appstream-rpms 
openscap-python3.x86_64                              1:1.3.6-4.el9                      rhel-9-for-x86_64-appstream-rpms 
postgresql-plpython3.x86_64                          13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
systemtap-runtime-python3.x86_64                     4.7-2.el9                          rhel-9-for-x86_64-appstream-rpms

■Samba
# dnf list all | grep samba
ipa-client-samba.x86_64                              4.10.0-8.el9_1                     rhel-9-for-x86_64-appstream-rpms 
pcp-pmda-samba.x86_64                                5.3.7-7.el9                        rhel-9-for-x86_64-appstream-rpms 
python3-samba.i686                                   4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
python3-samba.x86_64                                 4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba.x86_64                                         4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-client.x86_64                                  4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms 
samba-client-libs.i686                               4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-client-libs.x86_64                             4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-common.noarch                                  4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-common-libs.i686                               4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-common-libs.x86_64                             4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-common-tools.x86_64                            4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-krb5-printing.x86_64                           4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms 
samba-libs.i686                                      4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-libs.x86_64                                    4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-vfs-iouring.x86_64                             4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms 
samba-winbind.x86_64                                 4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-winbind-clients.x86_64                         4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms 
samba-winbind-krb5-locator.x86_64                    4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms 
samba-winbind-modules.i686                           4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-winbind-modules.x86_64                         4.16.4-101.el9                     rhel-9-for-x86_64-baseos-rpms    
samba-winexe.x86_64                                  4.16.4-101.el9                     rhel-9-for-x86_64-appstream-rpms

■Postfix
# dnf list all | grep postfix
pcp-pmda-postfix.x86_64                              5.3.7-7.el9                        rhel-9-for-x86_64-appstream-rpms 
postfix.x86_64                                       2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-cdb.x86_64                                   2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-ldap.x86_64                                  2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-mysql.x86_64                                 2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-pcre.x86_64                                  2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-perl-scripts.x86_64                          2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-pgsql.x86_64                                 2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms 
postfix-sqlite.x86_64                                2:3.5.9-19.el9                     rhel-9-for-x86_64-appstream-rpms

■Postgres
# dnf list all | grep postgres
pcp-pmda-postgresql.x86_64                           5.3.7-7.el9                        rhel-9-for-x86_64-appstream-rpms 
postgres-decoderbufs.x86_64                          1.4.0-4.Final.el9                  rhel-9-for-x86_64-appstream-rpms 
postgresql.x86_64                                    13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-contrib.x86_64                            13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-jdbc.noarch                               42.2.18-6.el9_1                    rhel-9-for-x86_64-appstream-rpms 
postgresql-odbc.x86_64                               12.02.0000-6.el9                   rhel-9-for-x86_64-appstream-rpms 
postgresql-plperl.x86_64                             13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-plpython3.x86_64                          13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-pltcl.x86_64                              13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-private-libs.x86_64                       13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-server.x86_64                             13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
postgresql-upgrade.x86_64                            13.7-1.el9_0                       rhel-9-for-x86_64-appstream-rpms 
qt5-qtbase-postgresql.i686                           5.15.3-1.el9                       rhel-9-for-x86_64-appstream-rpms 
qt5-qtbase-postgresql.x86_64                         5.15.3-1.el9                       rhel-9-for-x86_64-appstream-rpms 
tuned-profiles-postgresql.noarch                     2.19.0-1.el9                       rhel-9-for-x86_64-appstream-rpms
```

# 14. /var/log配下のログ運用

---

個人的には/var/log配下のログは当日日付のログのみとして、過去ログは別フォルダに移動したいです。

特に運用周りでrsyslog周りをしっかりしておきたいので修正をしていきます。

手始めに/etc/logrotate.d/rsyslogを無効化で全ての行にコメントアウトを挿入します。

その上で/etc/cron.d配下にrsyslogファイルを以下のように作成します。

以下のようにすることで毎日過去ログはフォルダ毎に保管され、/var/log配下には当日のログファイルのみとなり管理がしやすくなります。

以下ファイルを作成したら過去ログを保存するログフォルダの作成を忘れないようにして下さい。

```bash
/var/log/messages #ローテートしたいファイルパス
/var/log/secure
/var/log/cron
/var/log/maillog
{
    daily #ローテーションする間隔
    rotate 365 #ログ保存の世代数
    compress #ローテーションしたログをgzipで圧縮
    ifempty #ログファイルが空でもローテーション
    missingok #ログファイルが存在しなくてもエラーを出さずに処理続行
    create 0644 root root #ローテーション後に空のログファイルを新規作成。ファイルのパーミッション、ユーザー名、グループ名を指定可能
    dateext #ローテートしたファイルに日付文字列を付ける
    sharedscripts #複数指定したログファイルに対し、postrotateまたはprerotateで記述したコマンドを実行
    postrotate
        /bin/kill -HUP `cat /var/run/rsyslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
    lastaction #postrotateスクリプトが実行された後に以下処理を実行
        /bin/mv /var/log/messages-`date '+%Y%m%d'`.gz /var/log/MESSAGES/messages-`date '+%Y%m%d' -d '1days ago'`.gz
        /bin/mv /var/log/secure-`date '+%Y%m%d'`.gz /var/log/SECURE/secure-`date '+%Y%m%d' -d '1days ago'`.gz
        /bin/mv /var/log/cron-`date '+%Y%m%d'`.gz /var/log/CRON/cron-`date '+%Y%m%d' -d '1days ago'`.gz
        /bin/mv /var/log/maillog-`date '+%Y%m%d'`.gz /var/log/MAIL/maillog-`date '+%Y%m%d' -d '1days ago'`.gz
        /bin/rm -f /var/log/MESSAGES/messages.`date '+%Y%m%d' -d '365days ago'`.gz
        /bin/rm -f /var/log/SECURE/secure.`date '+%Y%m%d' -d '365days ago'`.gz
        /bin/rm -f /var/log/CRON/cron.`date '+%Y%m%d' -d '365days ago'`.gz
        /bin/rm -f /var/log/MAIL/maillog.`date '+%Y%m%d' -d '365days ago'`.gz
 endscript
}
```

ログローテートのテストを実施した後にログがローテートされていることを確認します。

```bash
# logrotate -f /etc/cron.d/rsyslog
# ls -la /var/log/{MESSAGES,CRON,SECURE,MAIL}
```

最後にCRONで処理が実行されるようにし、翌日ログファイルがローテートされていることを確認します。

```bash
# vi /etc/cron.d/rsyslog.cron
0 0 * * * root /usr/sbin/logrotate /etc/cron.d/rsyslog
```

### 上記までしなくてもいいけど、毎日ログローテートさせたい場合

今まではCRONで動作していたログローテートですが、RHEL9からsystemd-timerサービスで管理されるようになっています。

```jsx
# systemctl list-timers
```

を実行するとlogrotate.timerが一覧にあり、次回の実行時間が分かります。
ログローテートのタイミングを修正したい場合には、以下を作成して設定を読み込みしおなします。

```jsx
# vi /etc/systemd/system/logrotate.timer.d/override.conf
※cron のような表記法 (OnCalendar=Mon *-*-* 00:01:00 など) を使用できます。
※詳細は、man systemd.time で参照ください。
# systemctl daemon-reload
```

その後、/etc/logrotate.confの設定をweeklyからdailyに変更などを行うことで毎日ログローテートが実施されます。

# 15. boot.logの出力

---

boot.logがデフォルトで出力されないので、出力したい場合には以下PKGを追加してOS再起動を実施します。

```bash
# dnf install -y plymouth
# reboot
```

OS再起動後に/var/log配下にboot.logが出力されていればOKです。