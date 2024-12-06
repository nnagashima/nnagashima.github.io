# AmazonLinux2023検証 初期設定

![Untitled](AmazonLinux2023%E6%A4%9C%E8%A8%BC%20%E5%88%9D%E6%9C%9F%E8%A8%AD%E5%AE%9A%20b0aa31c9251f42afa8158dbcd99f0385/Untitled.png)

AmazonLinux2023における情報は以下を参照してください。

https://aws.amazon.com/jp/linux/amazon-linux-2023/faqs/

**特に気をつけることとしてはAmazonLinux2023にはEPELリポジトリがサポートされていません。**

# 目次

---

# 1. SSH接続

---

デフォルトでSHA1認証が無効化されたことによる影響で、現時点でTerratermでログインすることができません。

対処方法としては以下となります。

- Terratermが対応するまで待つ
- Terraterm以外のクライアントソフトを使ってログインする
- Terratermで接続したいのであればcrypto-policesのデフォルトをSHA1にする（セキュリティレベルが下がる）

なので筆者はTerraterm以外のクライアントソフトを使う方法が良いと考えます。（RLoginとか）

# 2. Hostname変更

---

```bash
# hostnamectl set-hostname hostname
# uname -n
hostname
```

# 3. OS言語設定

---

localectlで確認します。

```bash
# localectl status
System Locale: LANG=C.UTF-8
    VC Keymap: (unset)
   X11 Layout: (unset)

# localectl list-locales
ja_JP.UTF-8

変更する場合のコマンド例
# localectl set-locale LANG=ja_JP.UTF-8
```

# 4. TimeZone設定

---

timedatectlで確認します。

```bash
# timedatectl 
               Local time: 水 2023-04-19 03:52:04 UTC
           Universal time: 水 2023-04-19 03:52:04 UTC
                 RTC time: 水 2023-04-19 03:52:03
                Time zone: n/a (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

変更する場合のコマンド例
# timedatectl set-timezone Asia/Tokyo
```

# 5. システムの最新化

---

インストールやアップデートする際にはyumでもdnfでもどちらでも大丈夫そうです。

```bash
# ls -la /usr/bin/dnf
lrwxrwxrwx. 1 root root 5  1月 31 10:01 /usr/bin/dnf -> dnf-3

# ls -la /usr/bin/yum
lrwxrwxrwx. 1 root root 5  1月 31 10:01 /usr/bin/yum -> dnf-3

# dnf upgrade
```

# 6. ユーザーの追加

---

useraddでユーザー名を追加、ユーザーでSSHでログインするにあたりパスワードも設定します。

```bash
# useradd ユーザー名
# passwd ユーザー名

# su - 作成したユーザー名
$ mkdir .ssh
$ chmod 700 .ssh
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/username/.ssh/id_rsa
Your public key has been saved in /home/username/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Wy2897kyvT0ImH5qV58FIvhvEDsH4oExt9EiM1c12Is username@hostname
The key's randomart image is:
+---[RSA 3072]----+
|         o.+o    |
|      * = o ..   |
|       X = . .   |
|      . =.E.o .  |
|       .S+*=.. . |
|        .==+..  .|
|        o .=+.o o|
|         o +=o.* |
|        ..+. o=oo|
+----[SHA256]-----+
※RSA3072で作成されるようですね。
```

# 7. SUDOの設定

---

追加したユーザーにroot 権限を付与したい場合にはvisudoで変更します。

```bash
# visudo 

##root権限を全て利用
username  ALL=(ALL)       ALL

##root権限をパスワードなしで利用
username  ALL=NOPASSWD: ALL
```

# 8. Networkの設定

---

AWSの場合、インスタンス作成時にプライベートIPアドレスを割り当てて固定にすることができるので、

OS側での設定はしません。

また起動中の場合はシャットダウンしないとプライベートIPアドレスを固定にすることができないので注意してください。

# 9. NTP設定

---

Chronyを利用します。

ibrustを使用することで初回同期にかかる時間を改善することができます。

```bash
# cat /etc/chrony.d/ntp-pool.sources/etc/chrony.d/ntp-pool.sources
pool 同期したいNTPサーバを指定 ibrust

設定したらChronyの再起動を実施
# systemctl restart chronyd

動作確認
# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 180.43.145.38                 0   7     0     -     +0ns[   +0ns] +/-    0ns
```

# 10. NFSクライアント

---

共有フォルダマウントはEFSが主軸になると思いますので、EFSマウントヘルパーを利用した形を記載します。

```bash
# dnf install amazon-efs-utils
# mount -t efs -o tls efs-name:/ /mnt/efs/
# df -h
efs-name:/マウントパス   8.0E  0  8.0E   0% /mnt/efs
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

# 11. 主要PKGのバージョンについて

---

```bash
■OPENSSH
# rpm -qa | grep openssh
openssh-8.7p1-8.amzn2023.0.4.x86_64
openssh-server-8.7p1-8.amzn2023.0.4.x86_64
openssh-clients-8.7p1-8.amzn2023.0.4.x86_64

■Apahce
# dnf list all | grep httpd
generic-logos-httpd.noarch                                        18.0.0-12.amzn2023.0.3                      amazonlinux     
httpd.x86_64                                                      2.4.56-1.amzn2023                           amazonlinux     
httpd-core.x86_64                                                 2.4.56-1.amzn2023                           amazonlinux     
httpd-devel.x86_64                                                2.4.56-1.amzn2023                           amazonlinux     
httpd-filesystem.noarch                                           2.4.56-1.amzn2023                           amazonlinux     
httpd-manual.noarch                                               2.4.56-1.amzn2023                           amazonlinux     
httpd-tools.x86_64                                                2.4.56-1.amzn2023                           amazonlinux     
libmicrohttpd.x86_64                                              1:0.9.73-1.amzn2023.0.2                     amazonlinux     
libmicrohttpd-devel.x86_64                                        1:0.9.73-1.amzn2023.0.2                     amazonlinux     
libmicrohttpd-doc.noarch                                          1:0.9.73-1.amzn2023.0.2                     amazonlinux     
python3-sphinxcontrib-httpdomain.noarch                           1.7.0-11.amzn2023.0.2                       amazonlinux

■PHP
# dnf list all | grep php
php-pear.noarch                                                   1:1.10.13-2.amzn2023.0.4                    amazonlinux     
php8.1.x86_64                                                     8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-bcmath.x86_64                                              8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-cli.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-common.x86_64                                              8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-dba.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-dbg.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-devel.x86_64                                               8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-embedded.x86_64                                            8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-enchant.x86_64                                             8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-ffi.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-fpm.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-gd.x86_64                                                  8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-gmp.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-intl.x86_64                                                8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-ldap.x86_64                                                8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-mbstring.x86_64                                            8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-mysqlnd.x86_64                                             8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-odbc.x86_64                                                8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-opcache.x86_64                                             8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-pdo.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-pgsql.x86_64                                               8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-process.x86_64                                             8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-soap.x86_64                                                8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-tidy.x86_64                                                8.1.16-1.amzn2023.0.1                       amazonlinux     
php8.1-xml.x86_64                                                 8.1.16-1.amzn2023.0.1                       amazonlinux     
sphinx-php.x86_64                                                 2.2.11-24.amzn2023.0.4                      amazonlinux     
texlive-graphpaper.noarch                                         9:svn58661-59.amzn2023.0.2                  amazonlinux

■Mariadb
# dnf list all | grep mariadb
mariadb-connector-c.x86_64                                        3.1.13-1.amzn2023.0.3                       amazonlinux     
mariadb-connector-c-config.noarch                                 3.1.13-1.amzn2023.0.3                       amazonlinux     
mariadb-connector-c-devel.x86_64                                  3.1.13-1.amzn2023.0.3                       amazonlinux     
mariadb-connector-c-test.x86_64                                   3.1.13-1.amzn2023.0.3                       amazonlinux     
mariadb105.x86_64                                                 3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-backup.x86_64                                          3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-common.x86_64                                          3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-connect-engine.x86_64                                  3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-cracklib-password-check.x86_64                         3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-devel.x86_64                                           3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-errmsg.x86_64                                          3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-gssapi-server.x86_64                                   3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-oqgraph-engine.x86_64                                  3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-pam.x86_64                                             3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-rocksdb-engine.x86_64                                  3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-server.x86_64                                          3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-server-utils.x86_64                                    3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-sphinx-engine.x86_64                                   3:10.5.18-1.amzn2023.0.1                    amazonlinux     
mariadb105-test.x86_64                                            3:10.5.18-1.amzn2023.0.1                    amazonlinux

■NFS
# dnf list all | grep nfs
libnfsidmap.x86_64                                                1:2.5.4-2.rc3.amzn2023.0.3                  @System         
nfs-utils.x86_64                                                  1:2.5.4-2.rc3.amzn2023.0.3                  @System         
libnfs.x86_64                                                     4.0.0-4.amzn2023.0.2                        amazonlinux     
libnfs-devel.x86_64                                               4.0.0-4.amzn2023.0.2                        amazonlinux     
libnfs-utils.x86_64                                               4.0.0-4.amzn2023.0.2                        amazonlinux     
libnfsidmap-devel.x86_64                                          1:2.5.4-2.rc3.amzn2023.0.3                  amazonlinux     
libstoragemgmt-nfs-plugin.x86_64                                  1.9.4-5.amzn2023.0.2                        amazonlinux     
nfs-stats-utils.x86_64                                            1:2.5.4-2.rc3.amzn2023.0.3                  amazonlinux     
nfs-utils-coreos.x86_64                                           1:2.5.4-2.rc3.amzn2023.0.3                  amazonlinux     
nfsv4-client-utils.x86_64                                         1:2.5.4-2.rc3.amzn2023.0.3                  amazonlinux     
sssd-nfs-idmap.x86_64                                             2.5.0-1.amzn2023.0.3                        amazonlinux     
texlive-mfnfss.noarch                                             9:svn46036-59.amzn2023.0.2                  amazonlinux     
texlive-mfnfss-doc.noarch                                         9:svn46036-59.amzn2023.0.2                  amazonlinux     
texlive-nfssext-cfr.noarch                                        9:svn43640-59.amzn2023.0.2                  amazonlinux     
texlive-nfssext-cfr-doc.noarch                                    9:svn43640-59.amzn2023.0.2                  amazonlinux     
texlive-plnfss.noarch                                             9:svn15878.1.1-59.amzn2023.0.2              amazonlinux     
texlive-plnfss-doc.noarch                                         9:svn15878.1.1-59.amzn2023.0.2              amazonlinux     
texlive-psnfss.noarch                                             9:svn54694-59.amzn2023.0.2                  amazonlinux     
texlive-psnfss-doc.noarch                                         9:svn54694-59.amzn2023.0.2                  amazonlinux

■Python
# dnf list all | grep python3.x86_64
python3.x86_64                                                    3.9.16-1.amzn2023.0.3                       @System         
boost-mpich-python3.x86_64                                        1.75.0-4.amzn2023.0.2                       amazonlinux     
boost-openmpi-python3.x86_64                                      1.75.0-4.amzn2023.0.2                       amazonlinux     
boost-python3.x86_64                                              1.75.0-4.amzn2023.0.2                       amazonlinux     
libcap-ng-python3.x86_64                                          0.8.2-4.amzn2023.0.2                        amazonlinux     
libpeas-loader-python3.x86_64                                     1.32.0-1.amzn2023.0.3                       amazonlinux     
openscap-python3.x86_64                                           1:1.3.7-1.amzn2023.0.1                      amazonlinux     
postgresql15-plpython3.x86_64                                     15.0-1.amzn2023.0.2                         amazonlinux     
systemtap-runtime-python3.x86_64                                  4.8-3.amzn2023.0.5                          amazonlinux

■Samba
# dnf list all | grep samba
python3-samba.x86_64                                              2:4.17.5-0.amzn2023.0.2                     amazonlinux     
python3-samba-devel.x86_64                                        2:4.17.5-0.amzn2023.0.2                     amazonlinux     
python3-samba-test.x86_64                                         2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba.x86_64                                                      2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-client.x86_64                                               2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-client-libs.x86_64                                          2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-common.noarch                                               2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-common-libs.x86_64                                          2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-common-tools.x86_64                                         2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-dc-libs.x86_64                                              2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-dcerpc.x86_64                                               2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-devel.x86_64                                                2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-krb5-printing.x86_64                                        2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-ldb-ldap-modules.x86_64                                     2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-libs.x86_64                                                 2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-pidl.noarch                                                 2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-test.x86_64                                                 2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-test-libs.x86_64                                            2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-usershares.x86_64                                           2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-vfs-iouring.x86_64                                          2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-winbind.x86_64                                              2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-winbind-clients.x86_64                                      2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-winbind-krb5-locator.x86_64                                 2:4.17.5-0.amzn2023.0.2                     amazonlinux     
samba-winbind-modules.x86_64                                      2:4.17.5-0.amzn2023.0.2                     amazonlinux

■Postfix
# dnf list all | grep postfix
postfix.x86_64                                                    2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-cdb.x86_64                                                2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-ldap.x86_64                                               2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-lmdb.x86_64                                               2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-mysql.x86_64                                              2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-pcre.x86_64                                               2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-perl-scripts.x86_64                                       2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-pgsql.x86_64                                              2:3.7.2-4.amzn2023.0.4                      amazonlinux     
postfix-sqlite.x86_64                                             2:3.7.2-4.amzn2023.0.4                      amazonlinux

■Postgres
# dnf list all | grep postgres
collectd-postgresql.x86_64                                        5.12.0-16.amzn2023.0.2                      amazonlinux     
postgresql15.x86_64                                               15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-contrib.x86_64                                       15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-docs.x86_64                                          15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-llvmjit.x86_64                                       15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-plperl.x86_64                                        15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-plpython3.x86_64                                     15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-pltcl.x86_64                                         15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-private-devel.x86_64                                 15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-private-libs.x86_64                                  15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-server.x86_64                                        15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-server-devel.x86_64                                  15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-static.x86_64                                        15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-test.x86_64                                          15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-test-rpm-macros.noarch                               15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-upgrade.x86_64                                       15.0-1.amzn2023.0.2                         amazonlinux     
postgresql15-upgrade-devel.x86_64                                 15.0-1.amzn2023.0.2                         amazonlinux     

```

# 12. /var/log配下について

---

AmazonLinux2023の/var/log配下を確認すると従来のシステムログとかがない！？

該当フォルダを確認するとREADMEなるものがある。

```bash
# ls -la /var/log
合計 1556
drwxr-xr-x.  9 root   root             16384  4月 23 00:00 .
drwxr-xr-x. 19 root   root               266  4月 18 12:02 ..
lrwxrwxrwx.  1 root   root                39  4月  5 03:12 README -> ../../usr/share/doc/systemd/README.logs
drwxr-xr-x.  4 root   root                28  4月 19 16:15 amazon
drwx------.  2 root   root                23  4月 18 12:02 audit
-rw-rw----.  1 root   utmp               384  4月 18 12:03 btmp
drwxr-x---.  2 chrony chrony           16384  4月 27 00:08 chrony
-rw-r-----.  1 root   adm               6242  4月 19 10:45 cloud-init-output.log
-rw-r--r--.  1 root   adm             256851  4月 19 10:45 cloud-init.log
-rw-r--r--.  1 root   root            844962  4月 27 12:22 dnf.librepo.log
-rw-r--r--.  1 root   root            287173  4月 27 12:22 dnf.log
-rw-r--r--.  1 root   root             81429  4月 27 12:22 dnf.rpm.log
-rw-r--r--.  1 root   root               540  4月 27 12:22 hawkey.log
-rw-r--r--.  1 root   root              2779  4月 19 16:15 hawkey.log-20230423
drwxr-sr-x+  3 root   systemd-journal     46  4月 18 12:02 journal
-rw-rw-r--.  1 root   utmp            292584  4月 27 11:58 lastlog
drwx------.  2 root   root                 6  4月  5 03:12 private
drwxr-xr-x.  2 root   root               243  4月 27 00:07 sa
drwxr-x---.  2 root   root               167  4月 23 00:00 sssd
-rw-------.  1 root   root                 0  4月  5 03:12 tallylog
-rw-rw-r--.  1 root   utmp             13056  4月 27 11:58 wtmp

# cat /var/log/README 
You are looking for the traditional text log files in /var/log, and they aregone?

Here's an explanation on what's going on:

You are running a systemd-based OS where traditional syslog has been replaced
with the Journal. The journal stores the same (and more) information as classic
syslog. To make use of the journal and access the collected log data simply
invoke "journalctl", which will output the logs in the identical text-based
format the syslog files in /var/log used to be. For further details, please
refer to journalctl(1).

Alternatively, consider installing one of the traditional syslog
implementations available for your distribution, which will generate the
classic log files for you. Syslog implementations such as syslog-ng or rsyslog
may be installed side-by-side with the journal and will continue to function
the way they always did.

Thank you!

Further reading:
        man:journalctl(1)
        man:systemd-journald.service(8)
        man:journald.conf(5)
        http://0pointer.de/blog/projects/the-journal.html
```

要するにjournalを利用するかsyslogをインストールしてログ運用をしてくださいとのこと

```yaml
# dnf list | grep rsyslog
rsyslog.x86_64                                                    8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-crypto.x86_64                                             8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-doc.noarch                                                8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-elasticsearch.x86_64                                      8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-logrotate.x86_64                                          8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-mmaudit.x86_64                                            8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-mmfields.x86_64                                           8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-mmjsonparse.x86_64                                        8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-mmkubernetes.x86_64                                       8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-mmnormalize.x86_64                                        8.2204.0-3.amzn2023.0.2                     amazonlinux     
rsyslog-openssl.x86_64                                            8.2204.0-3.amzn2023.0.2                     amazonlinux     

# dnf install rsyslog
メタデータの期限切れの最終確認: 0:56:05 時間前の 2023年04月27日 12時14分15秒 に実施しました。
依存関係が解決しました。
==============================================================================================================================================================================================
 パッケージ                                      アーキテクチャー                     バージョン                                              リポジトリー                              サイズ
==============================================================================================================================================================================================
インストール:
 rsyslog                                         x86_64                               8.2204.0-3.amzn2023.0.2                                 amazonlinux                               784 k
依存関係のインストール:
 libestr                                         x86_64                               0.1.11-1.amzn2023.0.2                                   amazonlinux                                26 k
 libfastjson                                     x86_64                               0.99.9-1.amzn2023.0.2                                   amazonlinux                                38 k
弱い依存関係のインストール:
 rsyslog-logrotate                               x86_64                               8.2204.0-3.amzn2023.0.2                                 amazonlinux                                11 k

トランザクションの概要
==============================================================================================================================================================================================
インストール  4 パッケージ

ダウンロードサイズの合計: 860 k
インストール後のサイズ: 2.8 M
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
(1/4): libestr-0.1.11-1.amzn2023.0.2.x86_64.rpm                                                                                                               330 kB/s |  26 kB     00:00    
(2/4): libfastjson-0.99.9-1.amzn2023.0.2.x86_64.rpm                                                                                                           469 kB/s |  38 kB     00:00    
(3/4): rsyslog-logrotate-8.2204.0-3.amzn2023.0.2.x86_64.rpm                                                                                                   419 kB/s |  11 kB     00:00    
(4/4): rsyslog-8.2204.0-3.amzn2023.0.2.x86_64.rpm                                                                                                             4.9 MB/s | 784 kB     00:00    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                                                                          3.0 MB/s | 860 kB     00:00     
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  準備             :                                                                                                                                                                      1/1 
  インストール中   : libfastjson-0.99.9-1.amzn2023.0.2.x86_64                                                                                                                             1/4 
  インストール中   : libestr-0.1.11-1.amzn2023.0.2.x86_64                                                                                                                                 2/4 
  インストール中   : rsyslog-logrotate-8.2204.0-3.amzn2023.0.2.x86_64                                                                                                                     3/4 
  インストール中   : rsyslog-8.2204.0-3.amzn2023.0.2.x86_64                                                                                                                               4/4 
  scriptletの実行中: rsyslog-8.2204.0-3.amzn2023.0.2.x86_64                                                                                                                               4/4 
Created symlink /etc/systemd/system/multi-user.target.wants/rsyslog.service → /usr/lib/systemd/system/rsyslog.service.

  検証             : rsyslog-8.2204.0-3.amzn2023.0.2.x86_64                                                                                                                               1/4 
  検証             : libestr-0.1.11-1.amzn2023.0.2.x86_64                                                                                                                                 2/4 
  検証             : libfastjson-0.99.9-1.amzn2023.0.2.x86_64                                                                                                                             3/4 
  検証             : rsyslog-logrotate-8.2204.0-3.amzn2023.0.2.x86_64                                                                                                                     4/4 
==============================================================================================================================================================================================
WARNING:
  A newer release of "Amazon Linux" is available.

  Available Versions:

  Version 2023.0.20230419:
    Run the following command to upgrade to 2023.0.20230419:

      dnf upgrade --releasever=2023.0.20230419

    Release notes:
     https://docs.aws.amazon.com/linux/al2023/release-notes/relnotes.html

==============================================================================================================================================================================================

インストール済み:
  libestr-0.1.11-1.amzn2023.0.2.x86_64      libfastjson-0.99.9-1.amzn2023.0.2.x86_64      rsyslog-8.2204.0-3.amzn2023.0.2.x86_64      rsyslog-logrotate-8.2204.0-3.amzn2023.0.2.x86_64     

# rpm -qa | grep rsyslog
rsyslog-logrotate-8.2204.0-3.amzn2023.0.2.x86_64
rsyslog-8.2204.0-3.amzn2023.0.2.x86_64

# systemctl enable --now rsyslog
# systemctl status  rsyslog
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
     Active: active (running) since Thu 2023-04-27 13:11:09 JST; 6s ago
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
   Main PID: 1069958 (rsyslogd)
      Tasks: 3 (limit: 2250)
     Memory: 2.7M
        CPU: 4.309s
     CGroup: /system.slice/rsyslog.service
             └─1069958 /usr/sbin/rsyslogd -n

# ls -la /var/log
合計 2896
drwxr-xr-x.  9 root   root              16384  4月 27 13:10 .
drwxr-xr-x. 19 root   root                266  4月 18 12:02 ..
lrwxrwxrwx.  1 root   root                 39  4月  5 03:12 README -> ../../usr/share/doc/systemd/README.logs
drwxr-xr-x.  4 root   root                 28  4月 19 16:15 amazon
drwx------.  2 root   root                 23  4月 18 12:02 audit
-rw-rw----.  1 root   utmp                384  4月 18 12:03 btmp
drwxr-x---.  2 chrony chrony            16384  4月 27 00:08 chrony
-rw-r-----.  1 root   adm                6242  4月 19 10:45 cloud-init-output.log
-rw-r--r--.  1 root   adm              256851  4月 19 10:45 cloud-init.log
-rw-r--r--.  1 root   root             847315  4月 27 13:10 dnf.librepo.log
-rw-r--r--.  1 root   root             299506  4月 27 13:10 dnf.log
-rw-r--r--.  1 root   root              82159  4月 27 13:10 dnf.rpm.log
-rw-r--r--.  1 root   root                900  4月 27 13:10 hawkey.log
-rw-r--r--.  1 root   root               2779  4月 19 16:15 hawkey.log-20230423
drwxr-sr-x+  3 root   systemd-journal      46  4月 18 12:02 journal
-rw-rw-r--.  1 root   utmp             292584  4月 27 13:05 lastlog
-rw-------.  1 root   root                  0  4月 27 13:10 maillog
-rw-------.  1 root   root            1371199  4月 27 13:36 messages
drwx------.  2 root   root                  6  4月  5 03:12 private
drwxr-xr-x.  2 root   root                243  4月 27 00:07 sa
-rw-------.  1 root   root              17278  4月 27 13:11 secure
-rw-------.  1 root   root                  0  4月 27 13:10 spooler
drwxr-x---.  2 root   root                167  4月 23 00:00 sssd
-rw-------.  1 root   root                  0  4月  5 03:12 tallylog
-rw-rw-r--.  1 root   utmp              13824  4月 27 13:05 wtmp
```

maillog、messages、secure、spoolerが増えましたね。

# 13. /var/log配下のログ運用

---

個人的には/var/log配下のログは当日日付のログのみとして、過去ログは別フォルダに移動したいです。

特に運用周りでrsyslog周りをしっかりしておきたいので修正をしていきます。

```bash
# ls -la /etc/logrotate.d/
合計 44
drwxr-xr-x.  2 root root    96  4月 27 13:10 .
drwxr-xr-x. 78 root root 16384  4月 27 13:10 ..
-rw-r--r--.  1 root root   130 10月 14  2019 btmp　※1世代保持でMonthlyでローテート
-rw-r--r--.  1 root root   223  1月 31 05:05 chrony　※7世代保持でDailyでローテート
-rw-r--r--.  1 root root    88  4月 27  2022 dnf　※4世代保持でWeeklyでローテート
-rw-r--r--.  1 root root   408  2月  2 07:13 psacct　※31世代保持でDailyでローテート
-rw-r--r--.  1 root root   226  2月  2 10:16 rsyslog　※/etc/logroate.confで4世代保持でWeeklyでローテート
-rw-r--r--.  1 root root   237  2月 15 07:20 sssd　※2世代保持でWeeklyでローテート
-rw-r--r--.  1 root root   145 10月 14  2019 wtmp　※1世代保持でMonthlyでローテート
```

処理はcronを利用しますのでcronieのPKGを追加します。

```bash
# dnf install cronie
メタデータの期限切れの最終確認: 5:12:34 時間前の 2023年04月27日 12時14分15秒 に実施しました。
依存関係が解決しました。
================================================================================================================================================================
 パッケージ                             アーキテクチャー               バージョン                                     リポジトリー                        サイズ
================================================================================================================================================================
インストール:
 cronie                                 x86_64                         1.5.7-1.amzn2023.0.2                           amazonlinux                         115 k
依存関係のインストール:
 cronie-anacron                         x86_64                         1.5.7-1.amzn2023.0.2                           amazonlinux                          32 k

トランザクションの概要
================================================================================================================================================================
インストール  2 パッケージ

ダウンロードサイズの合計: 147 k
インストール後のサイズ: 341 k
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
(1/2): cronie-1.5.7-1.amzn2023.0.2.x86_64.rpm                                                                                   1.0 MB/s | 115 kB     00:00    
(2/2): cronie-anacron-1.5.7-1.amzn2023.0.2.x86_64.rpm                                                                           285 kB/s |  32 kB     00:00    
----------------------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                                            691 kB/s | 147 kB     00:00     
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  準備             :                                                                                                                                        1/1 
  インストール中   : cronie-anacron-1.5.7-1.amzn2023.0.2.x86_64                                                                                             1/2 
  scriptletの実行中: cronie-anacron-1.5.7-1.amzn2023.0.2.x86_64                                                                                             1/2 
  インストール中   : cronie-1.5.7-1.amzn2023.0.2.x86_64                                                                                                     2/2 
  scriptletの実行中: cronie-1.5.7-1.amzn2023.0.2.x86_64                                                                                                     2/2 
Created symlink /etc/systemd/system/multi-user.target.wants/crond.service → /usr/lib/systemd/system/crond.service.

  検証             : cronie-1.5.7-1.amzn2023.0.2.x86_64                                                                                                     1/2 
  検証             : cronie-anacron-1.5.7-1.amzn2023.0.2.x86_64                                                                                             2/2 
================================================================================================================================================================
WARNING:
  A newer release of "Amazon Linux" is available.

  Available Versions:

  Version 2023.0.20230419:
    Run the following command to upgrade to 2023.0.20230419:

      dnf upgrade --releasever=2023.0.20230419

     https://docs.aws.amazon.com/linux/al2023/release-notes/relnotes.html

================================================================================================================================================================

インストール済み:
  cronie-1.5.7-1.amzn2023.0.2.x86_64                                         cronie-anacron-1.5.7-1.amzn2023.0.2.x86_64                                        

完了しました!

# systemctl enable --now crond
# systemctl status crond
● crond.service - Command Scheduler
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: active (running) since Thu 2023-04-27 17:29:22 JST; 1s ago
   Main PID: 1093861 (crond)
      Tasks: 1 (limit: 2250)
     Memory: 984.0K
        CPU: 9ms
     CGroup: /system.slice/crond.service
             └─1093861 /usr/sbin/crond -n
```

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