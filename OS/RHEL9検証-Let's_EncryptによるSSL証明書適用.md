# RHEL9検証 ⑥Let's EncryptによるSSL証明書適用

# 目次
- [RHEL9検証 ⑥Let's EncryptによるSSL証明書適用](#rhel9検証-lets-encryptによるssl証明書適用)
- [目次](#目次)
- [1.前提](#1前提)
- [2.EPELリポジトリを有効化してEPELリポジトリをインストール](#2epelリポジトリを有効化してepelリポジトリをインストール)
- [3.Snapdのインストール](#3snapdのインストール)
- [4.Snapの起動とCertbotのインストール](#4snapの起動とcertbotのインストール)
- [5.Let’s EncryptからSSL証明書の取得とssl.confへ反映](#5lets-encryptからssl証明書の取得とsslconfへ反映)
- [6.Webサイト評価をA+にするための修正](#6webサイト評価をaにするための修正)


# 1.前提

---

80ポートと443ポートがインターネット環境からアクセスできることが望ましいですが、

ポート番号を変更して公開している方もいるかと思うので、標準のやり方ではなくDNS TXT recordを自分で登録して、

SSL証明書を発行するやり方を実施します。

なお、事前にサーバのGlobalIPとDNSが紐づけておいてください。（Aレコードが登録されている）

# 2.EPELリポジトリを有効化してEPELリポジトリをインストール

---

```bash
# subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms
リポジトリー 'codeready-builder-for-rhel-9-x86_64-rpms' は、このシステムに対して有効になりました。

# dnf install \
https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
サブスクリプション管理リポジトリーを更新しています。
Red Hat CodeReady Linux Builder for RHEL 9 x86_64 (RPMs)                                                                                      5.5 MB/s | 4.8 MB     00:00    
epel-release-latest-9.noarch.rpm                                                                                                               14 kB/s |  18 kB     00:01    
依存関係が解決しました。
==============================================================================================================================================================================
 パッケージ                                  アーキテクチャー                      バージョン                               リポジトリー                                サイズ
==============================================================================================================================================================================
インストール:
 epel-release                                noarch                                9-5.el9                                  @commandline                                 18 k

トランザクションの概要
==============================================================================================================================================================================
インストール  1 パッケージ

合計サイズ: 18 k
インストール後のサイズ: 25 k
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  準備             :                                                                                                                                                      1/1 
  インストール中   : epel-release-9-5.el9.noarch                                                                                                                          1/1 
  scriptletの実行中: epel-release-9-5.el9.noarch                                                                                                                          1/1 
Many EPEL packages require the CodeReady Builder (CRB) repository.
It is recommended that you run /usr/bin/crb enable to enable the CRB repository.

  検証             : epel-release-9-5.el9.noarch                                                                                                                          1/1 
インストール済みの製品が更新されています。

インストール済み:
  epel-release-9-5.el9.noarch                                                                                                                                                 

完了しました
```

# 3.Snapdのインストール

---

```bash
# dnf install snapd
サブスクリプション管理リポジトリーを更新しています。
メタデータの期限切れの最終確認: 0:00:06 時間前の 2023年04月26日 20時27分43秒 に実施しました。
依存関係が解決しました。
==============================================================================================================================================================================
 パッケージ                                    アーキテクチャー             バージョン                              リポジトリー                                        サイズ
==============================================================================================================================================================================
インストール:
 snapd                                         x86_64                       2.58.3-1.el9                            epel                                                 15 M
アップグレード:
 selinux-policy                                noarch                       34.1.43-1.el9_1.2                       rhel-9-for-x86_64-baseos-rpms                        55 k
 selinux-policy-targeted                       noarch                       34.1.43-1.el9_1.2                       rhel-9-for-x86_64-baseos-rpms                       6.7 M
依存関係のインストール:
 bash-completion                               noarch                       1:2.11-4.el9                            rhel-9-for-x86_64-baseos-rpms                       459 k
 libpkgconf                                    x86_64                       1.7.3-9.el9                             rhel-9-for-x86_64-baseos-rpms                        38 k
 pkgconf                                       x86_64                       1.7.3-9.el9                             rhel-9-for-x86_64-baseos-rpms                        45 k
 pkgconf-m4                                    noarch                       1.7.3-9.el9                             rhel-9-for-x86_64-baseos-rpms                        16 k
 pkgconf-pkg-config                            x86_64                       1.7.3-9.el9                             rhel-9-for-x86_64-baseos-rpms                        12 k
 snap-confine                                  x86_64                       2.58.3-1.el9                            epel                                                2.5 M
 snapd-selinux                                 noarch                       2.58.3-1.el9                            epel                                                192 k

トランザクションの概要
==============================================================================================================================================================================
インストール    8 パッケージ
アップグレード  2 パッケージ

ダウンロードサイズの合計: 25 M
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
(1/10): snapd-selinux-2.58.3-1.el9.noarch.rpm                                                                                                 1.1 MB/s | 192 kB     00:00    
(2/10): snap-confine-2.58.3-1.el9.x86_64.rpm                                                                                                  7.9 MB/s | 2.5 MB     00:00    
(3/10): pkgconf-1.7.3-9.el9.x86_64.rpm                                                                                                        148 kB/s |  45 kB     00:00    
(4/10): bash-completion-2.11-4.el9.noarch.rpm                                                                                                 2.1 MB/s | 459 kB     00:00    
(5/10): pkgconf-m4-1.7.3-9.el9.noarch.rpm                                                                                                      41 kB/s |  16 kB     00:00    
(6/10): snapd-2.58.3-1.el9.x86_64.rpm                                                                                                          15 MB/s |  15 MB     00:00    
(7/10): pkgconf-pkg-config-1.7.3-9.el9.x86_64.rpm                                                                                              46 kB/s |  12 kB     00:00    
(8/10): libpkgconf-1.7.3-9.el9.x86_64.rpm                                                                                                     141 kB/s |  38 kB     00:00    
(9/10): selinux-policy-34.1.43-1.el9_1.2.noarch.rpm                                                                                           289 kB/s |  55 kB     00:00    
(10/10): selinux-policy-targeted-34.1.43-1.el9_1.2.noarch.rpm                                                                                  11 MB/s | 6.7 MB     00:00    
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                                                           11 MB/s |  25 MB     00:02     
Extra Packages for Enterprise Linux 9 - x86_64                                                                                                1.6 MB/s | 1.6 kB     00:00    
GPG 鍵 0x3228467C をインポート中:
 Userid     : "Fedora (epel9) <epel@fedoraproject.org>"
 Fingerprint: FF8A D134 4597 106E CE81 3B91 8A38 72BF 3228 467C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9
これでよろしいですか? [y/N]: y
鍵のインポートに成功しました
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                     1/1 
  準備             :                                                                                                                                                      1/1 
  アップグレード中 : selinux-policy-34.1.43-1.el9_1.2.noarch                                                                                                             1/12 
  scriptletの実行中: selinux-policy-34.1.43-1.el9_1.2.noarch                                                                                                             1/12 
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                    2/12 
  アップグレード中 : selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                    2/12 
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                    2/12 
  scriptletの実行中: snapd-selinux-2.58.3-1.el9.noarch                                                                                                                   3/12 
  インストール中   : snapd-selinux-2.58.3-1.el9.noarch                                                                                                                   3/12 
  scriptletの実行中: snapd-selinux-2.58.3-1.el9.noarch                                                                                                                   3/12 
  インストール中   : libpkgconf-1.7.3-9.el9.x86_64                                                                                                                       4/12 
  インストール中   : pkgconf-1.7.3-9.el9.x86_64                                                                                                                          5/12 
  インストール中   : pkgconf-m4-1.7.3-9.el9.noarch                                                                                                                       6/12 
  インストール中   : pkgconf-pkg-config-1.7.3-9.el9.x86_64                                                                                                               7/12 
  インストール中   : bash-completion-1:2.11-4.el9.noarch                                                                                                                 8/12 
  インストール中   : snap-confine-2.58.3-1.el9.x86_64                                                                                                                    9/12 
  インストール中   : snapd-2.58.3-1.el9.x86_64                                                                                                                          10/12 
  scriptletの実行中: snapd-2.58.3-1.el9.x86_64                                                                                                                          10/12 
  整理             : selinux-policy-34.1.43-1.el9.noarch                                                                                                                11/12 
  scriptletの実行中: selinux-policy-34.1.43-1.el9.noarch                                                                                                                11/12 
  整理             : selinux-policy-targeted-34.1.43-1.el9.noarch                                                                                                       12/12 
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9.noarch                                                                                                       12/12 
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                   12/12 
  scriptletの実行中: snapd-selinux-2.58.3-1.el9.noarch                                                                                                                  12/12 
  scriptletの実行中: selinux-policy-targeted-34.1.43-1.el9.noarch                                                                                                       12/12 
  検証             : snap-confine-2.58.3-1.el9.x86_64                                                                                                                    1/12 
  検証             : snapd-2.58.3-1.el9.x86_64                                                                                                                           2/12 
  検証             : snapd-selinux-2.58.3-1.el9.noarch                                                                                                                   3/12 
  検証             : pkgconf-1.7.3-9.el9.x86_64                                                                                                                          4/12 
  検証             : pkgconf-m4-1.7.3-9.el9.noarch                                                                                                                       5/12 
  検証             : bash-completion-1:2.11-4.el9.noarch                                                                                                                 6/12 
  検証             : libpkgconf-1.7.3-9.el9.x86_64                                                                                                                       7/12 
  検証             : pkgconf-pkg-config-1.7.3-9.el9.x86_64                                                                                                               8/12 
  検証             : selinux-policy-34.1.43-1.el9_1.2.noarch                                                                                                             9/12 
  検証             : selinux-policy-34.1.43-1.el9.noarch                                                                                                                10/12 
  検証             : selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                                                                                   11/12 
  検証             : selinux-policy-targeted-34.1.43-1.el9.noarch                                                                                                       12/12 
インストール済みの製品が更新されています。

アップグレード済み:
  selinux-policy-34.1.43-1.el9_1.2.noarch                                           selinux-policy-targeted-34.1.43-1.el9_1.2.noarch                                          
インストール済み:
  bash-completion-1:2.11-4.el9.noarch  libpkgconf-1.7.3-9.el9.x86_64  pkgconf-1.7.3-9.el9.x86_64         pkgconf-m4-1.7.3-9.el9.noarch  pkgconf-pkg-config-1.7.3-9.el9.x86_64 
  snap-confine-2.58.3-1.el9.x86_64     snapd-2.58.3-1.el9.x86_64      snapd-selinux-2.58.3-1.el9.noarch 

完了しました!
```

# 4.Snapの起動とCertbotのインストール

---

```bash
# ln -s /var/lib/snapd/snap /snap
# echo 'export PATH=$PATH:/var/lib/snapd/snap/bin' > /etc/profile.d/snap.sh
# systemctl enable --now snapd snapd.socket
Created symlink /etc/systemd/system/multi-user.target.wants/snapd.service → /usr/lib/systemd/system/snapd.service.
Created symlink /etc/systemd/system/sockets.target.wants/snapd.socket → /usr/lib/systemd/system/snapd.socket.

# snap install certbot --classic
Setup snap "snapd" (18933) security profiles                                                                                                                                 -
2023-04-26T20:36:34+09:00 INFO Waiting for automatic snapd restart...
certbot 2.5.0 from Certbot Project (certbot-eff✓) installed

# ln -s /snap/bin/certbot /usr/bin/certbot
```

# 5.Let’s EncryptからSSL証明書の取得とssl.confへ反映

---

前提に記載した通り理由があって80ポート、443ポートで解放していないサイトもあるので、

TXTレコードを利用した方法で実施します。

途中でDNS TXTレコードが表示されるので登録後に、少し経過してからEnterをすることで正常に完了します。

※DNSレコード登録してすぐに実行すると失敗する可能性あります。

```bash
[root@ray-rhel9 ~]# certbot certonly --manual --key-type rsa -d DomainName --preferred-challenges dns 
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): EmailAddress

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
Requesting a certificate for DomainName

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

xxxxxxxxxxxxxxxxxxxxxxx #マスキングしています

with the following value:

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx #マスキングしています

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/xxxxxxxxxxxxxxxxxxxxxxx
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/DomainName/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/DomainName/privkey.pem
This certificate expires on 2023-07-25.
These files will be updated when the certificate renews.

NEXT STEPS:
- This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.
We were unable to subscribe you the EFF mailing list because your e-mail address appears to be invalid. You can try again later by visiting https://act.eff.org.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

# ls -la /etc/letsencrypt/live/DomainName/
合計 4
drwxr-xr-x 2 root root  93  4月 26 20:45 .
drwx------ 3 root root  46  4月 26 20:45 ..
-rw-r--r-- 1 root root 692  4月 26 20:45 README
lrwxrwxrwx 1 root root  42  4月 26 20:45 cert.pem -> ../../archive/DomainName/cert1.pem
lrwxrwxrwx 1 root root  43  4月 26 20:45 chain.pem -> ../../archive/DomainName/chain1.pem
lrwxrwxrwx 1 root root  47  4月 26 20:45 fullchain.pem -> ../../archive/DomainName/fullchain1.pem
lrwxrwxrwx 1 root root  45  4月 26 20:45 privkey.pem -> ../../archive/DomainName/privkey1.pem

# cat /etc/httpd/conf.d/ssl.conf | grep test.
SSLCertificateFile /etc/letsencrypt/live/DomainName/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/DomainName/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/DomainName/chain.pem

[root@ray-rhel9 ray-adm]# systemctl restart httpd
```

DigicertでSSLのCheckを実施します。

[SSL Certificate Checker - Diagnostic Tool | DigiCert.com](https://www.digicert.com/help/)

![](/OS/RHEL9検証-Let's_EncryptによるSSL証明書適用/Untitled.png)

SSLで公開したサイトのWebページの強度をQualys SSL Labsで確認します。

※設定はApache関連で設定した内容に基づいたもので実施しています。

[SSL Server Test (Powered by Qualys SSL Labs)](https://www.ssllabs.com/ssltest/)

![](/OS/RHEL9検証-Let's_EncryptによるSSL証明書適用/Untitled1.png)

# 6.Webサイト評価をA+にするための修正

---

評価内容をみて①から③を実施しスコアに変化はありませんでしたが、④を実行することでA+の評価に変わりました。

![](/OS\RHEL9検証-Let's_EncryptによるSSL証明書適用\Untitled2.png)

①DNS CAAが赤文字

DNS CAAとはドメインのSSL証明書に署名を許可する認証局を指定するDNSエントリです。

CAAレコードで認証局を指定することによりSSL証明書の改ざんを防ぐことができます。

CAAレコードは以下で生成できるので、生成したレコードを管理しているDNSにCAAレコードを追加してください。

[CAA Record Generator](https://sslmate.com/caa/)

②Cipher SuitesでいくつかWEAKが発生

httpd.confのSSLCipherSuiteにて、WEAKとなっているものを除きます。

※Handshake Simulationを確認し、必要と思われるものは閲覧できるようにして下さい。

※よくわからない人はIPAが推奨を定義しているので参考にすると良いかも

[](https://www.ipa.go.jp/security/crypto/guideline/gmcbt80000005ufv-att/tls_cipher_suite_config_20200707.pdf)

③TLS証明書のRSA鍵を4096bitへ変更

```bash
# certbot certonly --manual --key-type rsa --rsa-key-size 4096 -d DomainName --preferred-challenges dns
```

④HSTSの設定

Webブラウザに対して常にHTTPSによる通信を行うよう指示

```bash
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```