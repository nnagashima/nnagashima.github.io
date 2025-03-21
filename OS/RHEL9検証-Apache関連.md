# RHEL9検証 ⑤Apache関連

主な変更点は以下を参照してください。

[1.2. Apache HTTP Server への主な変更点 Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/9/html/deploying_web_servers_and_reverse_proxies/apache-changes-rhel9_setting-apache-http-server)

# 目次
- [RHEL9検証 ⑤Apache関連](#rhel9検証-apache関連)
- [目次](#目次)
- [1. Apache インストール](#1-apache-インストール)
- [3. Apacheの設定（主にセキュリティ周り）](#3-apacheの設定主にセキュリティ周り)
- [4.Apache起動](#4apache起動)
- [5.TestPage確認](#5testpage確認)
- [6.WebページのSSL化に伴い、mod\_sslのインストール](#6webページのssl化に伴いmod_sslのインストール)
- [7.Apache(mod\_ssl)の設定（主にセキュリティ周り）](#7apachemod_sslの設定主にセキュリティ周り)
- [8.VirtualHost設定](#8virtualhost設定)


# 1. Apache インストール

---

Apacheのインストール

```bash
# dnf install httpd
サブスクリプション管理リポジトリーを更新しています。
メタデータの期限切れの最終確認: 2:41:09 時間前の 2023年04月18日 13時57分01秒 に実施しました。
依存関係が解決しました。
================================================================================================================================================================================================================
 パッケージ                                        アーキテクチャー                      バージョン                                       リポジトリー                                                    サイズ
================================================================================================================================================================================================================
インストール:
 httpd                                             x86_64                                2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                 53 k
依存関係のインストール:
 apr                                               x86_64                                1.7.0-11.el9                                     rhel-9-for-x86_64-appstream-rpms                                127 k
 apr-util                                          x86_64                                1.6.1-20.el9                                     rhel-9-for-x86_64-appstream-rpms                                 98 k
 apr-util-bdb                                      x86_64                                1.6.1-20.el9                                     rhel-9-for-x86_64-appstream-rpms                                 15 k
 httpd-core                                        x86_64                                2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                1.5 M
 httpd-filesystem                                  noarch                                2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                 17 k
 httpd-tools                                       x86_64                                2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                 88 k
 mailcap                                           noarch                                2.1.49-5.el9                                     rhel-9-for-x86_64-baseos-rpms                                    35 k
 redhat-logos-httpd                                noarch                                90.4-1.el9                                       rhel-9-for-x86_64-appstream-rpms                                 18 k
弱い依存関係のインストール:
 apr-util-openssl                                  x86_64                                1.6.1-20.el9                                     rhel-9-for-x86_64-appstream-rpms                                 17 k
 mod_http2                                         x86_64                                1.15.19-3.el9_1.5                                rhel-9-for-x86_64-appstream-rpms                                153 k
 mod_lua                                           x86_64                                2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                 63 k

トランザクションの概要
================================================================================================================================================================================================================
インストール  12 パッケージ

ダウンロードサイズの合計: 2.2 M
インストール後のサイズ: 6.0 M
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
(1/12): apr-util-openssl-1.6.1-20.el9.x86_64.rpm                                                                                                                                 40 kB/s |  17 kB     00:00    
(2/12): apr-util-1.6.1-20.el9.x86_64.rpm                                                                                                                                        216 kB/s |  98 kB     00:00    
(3/12): apr-1.7.0-11.el9.x86_64.rpm                                                                                                                                             254 kB/s | 127 kB     00:00    
(4/12): redhat-logos-httpd-90.4-1.el9.noarch.rpm                                                                                                                                 90 kB/s |  18 kB     00:00    
(5/12): apr-util-bdb-1.6.1-20.el9.x86_64.rpm                                                                                                                                     77 kB/s |  15 kB     00:00    
(6/12): httpd-tools-2.4.53-7.el9_1.5.x86_64.rpm                                                                                                                                 502 kB/s |  88 kB     00:00    
(7/12): httpd-2.4.53-7.el9_1.5.x86_64.rpm                                                                                                                                       281 kB/s |  53 kB     00:00    
(8/12): mod_lua-2.4.53-7.el9_1.5.x86_64.rpm                                                                                                                                     356 kB/s |  63 kB     00:00    
(9/12): httpd-filesystem-2.4.53-7.el9_1.5.noarch.rpm                                                                                                                             84 kB/s |  17 kB     00:00    
(10/12): mod_http2-1.15.19-3.el9_1.5.x86_64.rpm                                                                                                                                 707 kB/s | 153 kB     00:00    
(11/12): httpd-core-2.4.53-7.el9_1.5.x86_64.rpm                                                                                                                                 6.3 MB/s | 1.5 MB     00:00    
(12/12): mailcap-2.1.49-5.el9.noarch.rpm                                                                                                                                        152 kB/s |  35 kB     00:00    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                                                                                            1.9 MB/s | 2.2 MB     00:01     
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  準備             :                                                                                                                                                                                        1/1 
  インストール中   : apr-1.7.0-11.el9.x86_64                                                                                                                                                               1/12 
  インストール中   : apr-util-bdb-1.6.1-20.el9.x86_64                                                                                                                                                      2/12 
  インストール中   : apr-util-1.6.1-20.el9.x86_64                                                                                                                                                          3/12 
  インストール中   : apr-util-openssl-1.6.1-20.el9.x86_64                                                                                                                                                  4/12 
  インストール中   : httpd-tools-2.4.53-7.el9_1.5.x86_64                                                                                                                                                   5/12 
  インストール中   : mailcap-2.1.49-5.el9.noarch                                                                                                                                                           6/12 
  scriptletの実行中: httpd-filesystem-2.4.53-7.el9_1.5.noarch                                                                                                                                              7/12 
  インストール中   : httpd-filesystem-2.4.53-7.el9_1.5.noarch                                                                                                                                              7/12 
  インストール中   : httpd-core-2.4.53-7.el9_1.5.x86_64                                                                                                                                                    8/12 
  インストール中   : mod_lua-2.4.53-7.el9_1.5.x86_64                                                                                                                                                       9/12 
  インストール中   : mod_http2-1.15.19-3.el9_1.5.x86_64                                                                                                                                                   10/12 
  インストール中   : redhat-logos-httpd-90.4-1.el9.noarch                                                                                                                                                 11/12 
  インストール中   : httpd-2.4.53-7.el9_1.5.x86_64                                                                                                                                                        12/12 
  scriptletの実行中: httpd-2.4.53-7.el9_1.5.x86_64                                                                                                                                                        12/12 
  検証             : apr-util-openssl-1.6.1-20.el9.x86_64                                                                                                                                                  1/12 
  検証             : apr-util-1.6.1-20.el9.x86_64                                                                                                                                                          2/12 
  検証             : apr-1.7.0-11.el9.x86_64                                                                                                                                                               3/12 
  検証             : redhat-logos-httpd-90.4-1.el9.noarch                                                                                                                                                  4/12 
  検証             : apr-util-bdb-1.6.1-20.el9.x86_64                                                                                                                                                      5/12 
  検証             : httpd-tools-2.4.53-7.el9_1.5.x86_64                                                                                                                                                   6/12 
  検証             : httpd-2.4.53-7.el9_1.5.x86_64                                                                                                                                                         7/12 
  検証             : mod_lua-2.4.53-7.el9_1.5.x86_64                                                                                                                                                       8/12 
  検証             : httpd-filesystem-2.4.53-7.el9_1.5.noarch                                                                                                                                              9/12 
  検証             : mod_http2-1.15.19-3.el9_1.5.x86_64                                                                                                                                                   10/12 
  検証             : httpd-core-2.4.53-7.el9_1.5.x86_64                                                                                                                                                   11/12 
  検証             : mailcap-2.1.49-5.el9.noarch                                                                                                                                                          12/12 
インストール済みの製品が更新されています。

インストール済み:
  apr-1.7.0-11.el9.x86_64                apr-util-1.6.1-20.el9.x86_64                 apr-util-bdb-1.6.1-20.el9.x86_64        apr-util-openssl-1.6.1-20.el9.x86_64     httpd-2.4.53-7.el9_1.5.x86_64         
  httpd-core-2.4.53-7.el9_1.5.x86_64     httpd-filesystem-2.4.53-7.el9_1.5.noarch     httpd-tools-2.4.53-7.el9_1.5.x86_64     mailcap-2.1.49-5.el9.noarch              mod_http2-1.15.19-3.el9_1.5.x86_64    
  mod_lua-2.4.53-7.el9_1.5.x86_64        redhat-logos-httpd-90.4-1.el9.noarch        

完了しました!
```

# 3. Apacheの設定（主にセキュリティ周り）

---

- サーバ名の設定
    - Servername欄の修正
- ドキュメントルートのインデックスを無効
    - Options Indexes FollowSymLinksからOptions FollowSymLinksに変更
- Apacheバージョン表示の無効化
    - ServerSignature / ServerTokens Prodの追加
- TRACEメソッドを利用したHTTPリクエスヘッダの認証上などを盗み出す手法の無効化
    - TraceEnable offの追加
- X-Frame-Optionsヘッダの設定（クリックジャッキング攻撃の対策）
    - 自サーバのコンテンツを他のサイトに埋め込まれないようにして、意図しないサイトへ強制的に誘導してしまわないようにする
    - 00-base.conf内で、モジュール：LoadModule headers_module modules/mod_headers.soはデフォルトで有効化されている
    - Header append X-FRAME-OPTIONS "SAMEORIGIN”の追加（DENYにするとサイト表示に影響がある可能性あり）
- Header always unset X-Powered-By（PHPを導入している場合、PHPのバージョンを非表示）
- PHPにはモジュール版のCGIがあるため、CGIの機能を利用せずにPHPを実行できるので無効化します。

設定した結果の差分

```bash
# cp -p /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.org
# vi /etc/httpd/conf/httpd.conf
# diff -U0 /etc/httpd/conf/httpd.conf.org /etc/httpd/conf/httpd.conf
--- /etc/httpd/conf/httpd.conf.org      2023-03-30 00:01:45.000000000 +0900
+++ /etc/httpd/conf/httpd.conf  2023-05-01 16:20:55.117742879 +0900
@@ -100 +100 @@
-#ServerName www.example.com:80
+ServerName DomainName
@@ -149 +149 @@
-    Options Indexes FollowSymLinks
+    Options FollowSymLinks
@@ -156 +156 @@
-    AllowOverride None
+    AllowOverride All 
@@ -260,5 +260,5 @@
-<Directory "/var/www/cgi-bin">
-    AllowOverride None
-    Options None
-    Require all granted
-</Directory>
+#<Directory "/var/www/cgi-bin">
+#    AllowOverride None
+#    Options None
+#    Require all granted
+#</Directory>
@@ -358,0 +359,6 @@
+
+ServerSignature Off
+ServerTokens Prod
+TraceEnable off
+Header append X-FRAME-OPTIONS "SAMEORIGIN"
+Header always unset X-Powered-By
```

iconsフォルダはデフォルトコンテンツが利用する画像となり通常利用しないものなので無効化

```bash
# cd /etc/httpd/conf.d/
# mv autoindex.conf autoindex.conf.org
```

WelcomePageの削除

```bash
# mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.org
```

# 4.Apache起動

---

```bash
# systemctl enable --now httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

# systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-04-18 17:08:44 JST; 12s ago
       Docs: man:httpd.service(8)
   Main PID: 2070 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 24752)
     Memory: 41.5M
        CPU: 86ms
     CGroup: /system.slice/httpd.service
             ├─2070 /usr/sbin/httpd -DFOREGROUND
             ├─2072 /usr/sbin/httpd -DFOREGROUND
             ├─2073 /usr/sbin/httpd -DFOREGROUND
             ├─2074 /usr/sbin/httpd -DFOREGROUND
             └─2075 /usr/sbin/httpd -DFOREGROUND

 4月 18 17:08:44 ray-rhel9 systemd[1]: Starting The Apache HTTP Server...
 4月 18 17:08:44 ray-rhel9 httpd[2070]: Server configured, listening on: port 80
 4月 18 17:08:44 ray-rhel9 systemd[1]: Started The Apache HTTP Server.
```

# 5.TestPage確認

---

```bash
# vi /var/www/html/index.html 
<html>
<body>
<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
Web Test Page
</div>
</body>
</html>
```

Webページを開いてアクセスできること、ならびにTestPageが表示されていることを確認

# 6.WebページのSSL化に伴い、mod_sslのインストール

---

mod_sslのインストール

```bash
# dnf install mod_ssl
サブスクリプション管理リポジトリーを更新しています。
メタデータの期限切れの最終確認: 3:59:01 時間前の 2023年04月18日 13時57分01秒 に実施しました。
依存関係が解決しました。
==========================================================================================================================================================================================================
 パッケージ                              アーキテクチャー                       バージョン                                         リポジトリー                                                     サイズ
==========================================================================================================================================================================================================
インストール:
 mod_ssl                                 x86_64                                 1:2.4.53-7.el9_1.5                                 rhel-9-for-x86_64-appstream-rpms                                 114 k
依存関係のインストール:
 sscg                                    x86_64                                 3.0.0-5.el9_0                                      rhel-9-for-x86_64-appstream-rpms                                  49 k

トランザクションの概要
==========================================================================================================================================================================================================
インストール  2 パッケージ

ダウンロードサイズの合計: 163 k
インストール後のサイズ: 366 k
これでよろしいですか? [y/N]: y
パッケージのダウンロード:
(1/2): sscg-3.0.0-5.el9_0.x86_64.rpm                                                                                                                                       91 kB/s |  49 kB     00:00    
(2/2): mod_ssl-2.4.53-7.el9_1.5.x86_64.rpm                                                                                                                                200 kB/s | 114 kB     00:00    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                                                                                      284 kB/s | 163 kB     00:00     
トランザクションの確認を実行中
トランザクションの確認に成功しました。
トランザクションのテストを実行中
トランザクションのテストに成功しました。
トランザクションを実行中
  準備             :                                                                                                                                                                                  1/1 
  インストール中   : sscg-3.0.0-5.el9_0.x86_64                                                                                                                                                        1/2 
  インストール中   : mod_ssl-1:2.4.53-7.el9_1.5.x86_64                                                                                                                                                2/2 
  scriptletの実行中: mod_ssl-1:2.4.53-7.el9_1.5.x86_64                                                                                                                                                2/2 
  検証             : sscg-3.0.0-5.el9_0.x86_64                                                                                                                                                        1/2 
  検証             : mod_ssl-1:2.4.53-7.el9_1.5.x86_64                                                                                                                                                2/2 
インストール済みの製品が更新されています。

インストール済み:
  mod_ssl-1:2.4.53-7.el9_1.5.x86_64                                                                       sscg-3.0.0-5.el9_0.x86_64                                                                      

完了しました!
```

# 7.Apache(mod_ssl)の設定（主にセキュリティ周り）

---

- 安全性の低い暗号スイートのサポートの無効化
    - SSLCipherSuite PROFILE=SYSTEMとなっており、crypto-policiesの仕組みが導入されている（RHEL8から導入されている）
    - openssl ciphers -v 'PROFILE=SYSTEM’で確認することができる
    - LEGACY / DEFAULT / FUTURE / FIPSから指定が可能
    - 試しにFETUREにしたら強度は上がったが、The policy is not supposed to be used for general purpose systemsとRedHatのドキュメントあり
    - ポリシー変更はupdate-crypto-policies —set Policyで可能
    - 基本的にはDefaultをそのまま使うことを推奨しているが、SSLv3が存在している
    - MozillaのGeneratorで設定を作成することもできるみたい
    
    [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
    
    - 踏襲したくないのであれば自分で暗号化スイートを設定をする必要あり（IPAが推奨を定義しているので参考にすると良い）
    
    [TLS暗号設定ガイドライン　安全なウェブサイトのために（暗号設定対策編） | 情報セキュリティ | IPA 独立行政法人 情報処理推進機構](https://www.ipa.go.jp/security/crypto/guideline/ssl_crypt_config.html)
    

```bash
# openssl ciphers -v 'PROFILE=SYSTEM'
TLS_AES_256_GCM_SHA384         TLSv1.3 Kx=any      Au=any   Enc=AESGCM(256)            Mac=AEAD
TLS_CHACHA20_POLY1305_SHA256   TLSv1.3 Kx=any      Au=any   Enc=CHACHA20/POLY1305(256) Mac=AEAD
TLS_AES_128_GCM_SHA256         TLSv1.3 Kx=any      Au=any   Enc=AESGCM(128)            Mac=AEAD
TLS_AES_128_CCM_SHA256         TLSv1.3 Kx=any      Au=any   Enc=AESCCM(128)            Mac=AEAD
ECDHE-ECDSA-AES256-GCM-SHA384  TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESGCM(256)            Mac=AEAD
ECDHE-RSA-AES256-GCM-SHA384    TLSv1.2 Kx=ECDH     Au=RSA   Enc=AESGCM(256)            Mac=AEAD
ECDHE-ECDSA-CHACHA20-POLY1305  TLSv1.2 Kx=ECDH     Au=ECDSA Enc=CHACHA20/POLY1305(256) Mac=AEAD
ECDHE-RSA-CHACHA20-POLY1305    TLSv1.2 Kx=ECDH     Au=RSA   Enc=CHACHA20/POLY1305(256) Mac=AEAD
ECDHE-ECDSA-AES256-CCM         TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESCCM(256)            Mac=AEAD
ECDHE-ECDSA-AES128-GCM-SHA256  TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESGCM(128)            Mac=AEAD
ECDHE-RSA-AES128-GCM-SHA256    TLSv1.2 Kx=ECDH     Au=RSA   Enc=AESGCM(128)            Mac=AEAD
ECDHE-ECDSA-AES128-CCM         TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESCCM(128)            Mac=AEAD
ECDHE-ECDSA-AES128-SHA256      TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AES(128)               Mac=SHA256
ECDHE-RSA-AES128-SHA256        TLSv1.2 Kx=ECDH     Au=RSA   Enc=AES(128)               Mac=SHA256
ECDHE-ECDSA-AES256-SHA         TLSv1   Kx=ECDH     Au=ECDSA Enc=AES(256)               Mac=SHA1
ECDHE-RSA-AES256-SHA           TLSv1   Kx=ECDH     Au=RSA   Enc=AES(256)               Mac=SHA1
ECDHE-ECDSA-AES128-SHA         TLSv1   Kx=ECDH     Au=ECDSA Enc=AES(128)               Mac=SHA1
ECDHE-RSA-AES128-SHA           TLSv1   Kx=ECDH     Au=RSA   Enc=AES(128)               Mac=SHA1
AES256-GCM-SHA384              TLSv1.2 Kx=RSA      Au=RSA   Enc=AESGCM(256)            Mac=AEAD
AES256-CCM                     TLSv1.2 Kx=RSA      Au=RSA   Enc=AESCCM(256)            Mac=AEAD
AES128-GCM-SHA256              TLSv1.2 Kx=RSA      Au=RSA   Enc=AESGCM(128)            Mac=AEAD
AES128-CCM                     TLSv1.2 Kx=RSA      Au=RSA   Enc=AESCCM(128)            Mac=AEAD
AES256-SHA256                  TLSv1.2 Kx=RSA      Au=RSA   Enc=AES(256)               Mac=SHA256
AES128-SHA256                  TLSv1.2 Kx=RSA      Au=RSA   Enc=AES(128)               Mac=SHA256
AES256-SHA                     SSLv3   Kx=RSA      Au=RSA   Enc=AES(256)               Mac=SHA1
AES128-SHA                     SSLv3   Kx=RSA      Au=RSA   Enc=AES(128)               Mac=SHA1
DHE-RSA-AES256-GCM-SHA384      TLSv1.2 Kx=DH       Au=RSA   Enc=AESGCM(256)            Mac=AEAD
DHE-RSA-CHACHA20-POLY1305      TLSv1.2 Kx=DH       Au=RSA   Enc=CHACHA20/POLY1305(256) Mac=AEAD
DHE-RSA-AES256-CCM             TLSv1.2 Kx=DH       Au=RSA   Enc=AESCCM(256)            Mac=AEAD
DHE-RSA-AES128-GCM-SHA256      TLSv1.2 Kx=DH       Au=RSA   Enc=AESGCM(128)            Mac=AEAD
DHE-RSA-AES128-CCM             TLSv1.2 Kx=DH       Au=RSA   Enc=AESCCM(128)            Mac=AEAD
DHE-RSA-AES256-SHA256          TLSv1.2 Kx=DH       Au=RSA   Enc=AES(256)               Mac=SHA256
DHE-RSA-AES128-SHA256          TLSv1.2 Kx=DH       Au=RSA   Enc=AES(128)               Mac=SHA256
DHE-RSA-AES256-SHA             SSLv3   Kx=DH       Au=RSA   Enc=AES(256)               Mac=SHA1
DHE-RSA-AES128-SHA             SSLv3   Kx=DH       Au=RSA   Enc=AES(128)               Mac=SHA1
PSK-AES256-GCM-SHA384          TLSv1.2 Kx=PSK      Au=PSK   Enc=AESGCM(256)            Mac=AEAD
PSK-CHACHA20-POLY1305          TLSv1.2 Kx=PSK      Au=PSK   Enc=CHACHA20/POLY1305(256) Mac=AEAD
PSK-AES256-CCM                 TLSv1.2 Kx=PSK      Au=PSK   Enc=AESCCM(256)            Mac=AEAD
PSK-AES128-GCM-SHA256          TLSv1.2 Kx=PSK      Au=PSK   Enc=AESGCM(128)            Mac=AEAD
PSK-AES128-CCM                 TLSv1.2 Kx=PSK      Au=PSK   Enc=AESCCM(128)            Mac=AEAD
PSK-AES256-CBC-SHA             SSLv3   Kx=PSK      Au=PSK   Enc=AES(256)               Mac=SHA1
PSK-AES128-CBC-SHA256          TLSv1   Kx=PSK      Au=PSK   Enc=AES(128)               Mac=SHA256
PSK-AES128-CBC-SHA             SSLv3   Kx=PSK      Au=PSK   Enc=AES(128)               Mac=SHA1
DHE-PSK-AES256-GCM-SHA384      TLSv1.2 Kx=DHEPSK   Au=PSK   Enc=AESGCM(256)            Mac=AEAD
DHE-PSK-CHACHA20-POLY1305      TLSv1.2 Kx=DHEPSK   Au=PSK   Enc=CHACHA20/POLY1305(256) Mac=AEAD
DHE-PSK-AES256-CCM             TLSv1.2 Kx=DHEPSK   Au=PSK   Enc=AESCCM(256)            Mac=AEAD
DHE-PSK-AES128-GCM-SHA256      TLSv1.2 Kx=DHEPSK   Au=PSK   Enc=AESGCM(128)            Mac=AEAD
DHE-PSK-AES128-CCM             TLSv1.2 Kx=DHEPSK   Au=PSK   Enc=AESCCM(128)            Mac=AEAD
DHE-PSK-AES256-CBC-SHA         SSLv3   Kx=DHEPSK   Au=PSK   Enc=AES(256)               Mac=SHA1
DHE-PSK-AES128-CBC-SHA256      TLSv1   Kx=DHEPSK   Au=PSK   Enc=AES(128)               Mac=SHA256
DHE-PSK-AES128-CBC-SHA         SSLv3   Kx=DHEPSK   Au=PSK   Enc=AES(128)               Mac=SHA1
ECDHE-PSK-CHACHA20-POLY1305    TLSv1.2 Kx=ECDHEPSK Au=PSK   Enc=CHACHA20/POLY1305(256) Mac=AEAD
ECDHE-PSK-AES256-CBC-SHA       TLSv1   Kx=ECDHEPSK Au=PSK   Enc=AES(256)               Mac=SHA1
ECDHE-PSK-AES128-CBC-SHA256    TLSv1   Kx=ECDHEPSK Au=PSK   Enc=AES(128)               Mac=SHA256
ECDHE-PSK-AES128-CBC-SHA       TLSv1   Kx=ECDHEPSK Au=PSK   Enc=AES(128)               Mac=SHA1
RSA-PSK-AES256-GCM-SHA384      TLSv1.2 Kx=RSAPSK   Au=RSA   Enc=AESGCM(256)            Mac=AEAD
RSA-PSK-CHACHA20-POLY1305      TLSv1.2 Kx=RSAPSK   Au=RSA   Enc=CHACHA20/POLY1305(256) Mac=AEAD
RSA-PSK-AES128-GCM-SHA256      TLSv1.2 Kx=RSAPSK   Au=RSA   Enc=AESGCM(128)            Mac=AEAD
RSA-PSK-AES256-CBC-SHA         SSLv3   Kx=RSAPSK   Au=RSA   Enc=AES(256)               Mac=SHA1
RSA-PSK-AES128-CBC-SHA256      TLSv1   Kx=RSAPSK   Au=RSA   Enc=AES(128)               Mac=SHA256
RSA-PSK-AES128-CBC-SHA         SSLv3   Kx=RSAPSK   Au=RSA   Enc=AES(128)               Mac=SHA1
```

SSL_PROTOCOLは、The SSL protocol version (SSLv3, TLSv1, TLSv1.1, TLSv1.2)から選択できる様子

This module provides SSL v3 and TLS v1.x support for the Apache HTTP Server. SSL v2 is no longer supported.

[mod_ssl - Apache HTTP Server Version 2.4](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html)

調べた結果からの結論

```bash
SSLProtocol all -SSLv3
SSLProxyProtocol all -SSLv3　※mod_proxyを利用している時のみ

現代のセキュリティの推奨ステータスは以下となる。
プロトコル名称	バージョン	公開年度	ステータス
SSL	1.0	(非公開)	廃止
SSL 2.0	1995年	廃止
SSL 3.0	1996年	廃止
TLS	1.0	1999年	非推奨 
TLS 1.1	2006年	非推奨
TLS 1.2	2008年	推奨
TLS 1.3	2018年	推奨
ので、推奨に合わせる形にするのであれば、SSLProtocol +TLSv1.2 +TLSv1.3が推奨パラメータになるかもしれない。
ただし公開するWEBサイトの兼ね合いなのでユーザーと要相談事項になるかと思われる。

また暗号化スイートに関しては
SSLCipherSuite PROFILE=SYSTEM
SSLProxyCipherSuite PROFILE=SYSTEM
のままでも良いかもしれない
```

追加で設定すべきと思われる内容

```bash
SSL/TLSセッションキャッシュ：
SSLSessionCacheTimeoutが300秒となっており短いと安全だが、一方で毎回鍵生成が走りパフォーマンスが悪くなる可能性がある。
そのため1時間ぐらいが推奨とも言われているため、3600で設定
※アクセスが多いサイトはキャッシュサイズをSSLSessionCacheを大きめにすることでWebページの改善が見込めるかもしれない

サイドチャネル攻撃（暗号解読攻撃）：
SSLCompressionはデフォルトでoffになっているので変更は必要なし

OCSPステープリング：
クライアントが SSL/TLS 通信を使うWEBサイトにアクセスすると、証明書の失効情報を確認するため、
クライアントが直接 OCSPレスポンダ（一般的に認証局が運営しています）に問い合わせを行うためパフォーマンス的によろしくない。
有効化することでOCSPレスポンダの問い合わせ結果をキャッシュしてくれるので、アクセスするたびにOCSPレスポンダへ問い合わせする必要がなくなる
```

設定した結果の差分

```bash
# cp -p /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.org
# vi /etc/httpd/conf.d/ssl.conf
# diff -U0 /etc/httpd/conf.d/ssl.conf.org /etc/httpd/conf.d/ssl.conf
--- /etc/httpd/conf.d/ssl.conf.org      2023-03-30 00:01:45.000000000 +0900
+++ /etc/httpd/conf.d/ssl.conf  2023-05-01 16:29:58.824610827 +0900
@@ -24 +24 @@
-SSLSessionCacheTimeout  300
+SSLSessionCacheTimeout  3600
@@ -43,2 +45,2 @@
-#DocumentRoot "/var/www/html"
-#ServerName www.example.com:443
+DocumentRoot "/var/www/html"
+ServerName DomainName:443
@@ -59,2 +61,2 @@
-#SSLProtocol all -SSLv3
-#SSLProxyProtocol all -SSLv3
+SSLProtocol +TLSv1.2 +TLSv1.3 
+SSLProxyProtocol +TLSv1.2 +TLSv1.3
@@ -164,3 +170,3 @@
-<Directory "/var/www/cgi-bin">
-    SSLOptions +StdEnvVars
-</Directory>
+#<Directory "/var/www/cgi-bin">
+#    SSLOptions +StdEnvVars
+#</Directory>
@@ -203,0 +210,3 @@
+SSLUseStapling On
+SSLStaplingReturnResponderErrors off
+SSLStaplingCache shmcb:/run/httpd/stapling_cache(128000)

```

他にも

Content-Security-Policy：クロスサイトスクリプティングやデータインジェクション等の攻撃を検知し、影響を軽減するための設定

X-Content-Type-Options：ブラウザが *Content-type* を検査（sniffing）することを無効にします。

X-Frame-Options：iframeタグによるクリックジャッキングを防止するための設定

などの設定するとよりセキュアになるのかもしれない。

# 8.VirtualHost設定

---

/etc/httpd/conf.d/vhost.confを作成して、複数のドメイン名を利用できるようにします。

※この時点ではSSL証明書の適用は実施しておりません。

```bash
# cat /etc/httpd/conf.d/vhost.conf
<VirtualHost *:443>
    DocumentRoot /var/www/test2
    ServerName DomainName
    ErrorLog logs/virtual.host-error_log
    CustomLog logs/virtual.host-access_log combined
</VirtualHost>

<Directory "/var/www/test2">
    Options FollowSymLinks
    AllowOverride All
</Directory>
```

ディレクトリ（/var/www/test2）を作成して、indexファイルを作成して表示ができればOKです。
