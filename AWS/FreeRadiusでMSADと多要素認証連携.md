[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# FreeRadiusでMSADと多要素認証連携

# 目次
- [FreeRadiusでMSADと多要素認証連携](#freeradiusでmsadと多要素認証連携)
- [目次](#目次)
- [はじめに](#はじめに)
- [FreeRadiusのパッケージのインストール](#freeradiusのパッケージのインストール)
- [FreeRadiusの起動テスト](#freeradiusの起動テスト)
- [Google Authenticatorをインストール](#google-authenticatorをインストール)
- [ホスト名の設定](#ホスト名の設定)
- [FreeRadiusの設定](#freeradiusの設定)
- [RadiusがGoogoleAuthenticatorを利用するようにする](#radiusがgoogoleauthenticatorを利用するようにする)
- [Radiusのクライアント設定](#radiusのクライアント設定)
- [PAMモジュールの有効化](#pamモジュールの有効化)
- [FreeRadiusの再起動](#freeradiusの再起動)
- [WorkSpacesにログインするユーザーを追加](#workspacesにログインするユーザーを追加)
- [MSADにて多要素認証を有効化](#msadにて多要素認証を有効化)
- [WorkSpacesへのログイン](#workspacesへのログイン)
- [ここまでを振り返って](#ここまでを振り返って)
- [ADとFreeRadiusでユーザー管理が別になるを解消](#adとfreeradiusでユーザー管理が別になるを解消)
- [シングル構成を冗長構成にしたいを解消](#シングル構成を冗長構成にしたいを解消)

---

# はじめに

---

MSADだとWorkSpacesやClientVPNと接続する際にAD認証のみとなります。

AD認証以外ですと

- WorkSpacesの場合はデバイス証明書による認証
- ClientVPNの場合はクライアント証明書で認証

をすることできます。

それ以外には多要素認証をサポートしております。

Passlogicを利用したやり方は以下リンクで記載していますが、今回はFreeRadiusを利用したやり方を記載していきます。

[PassLogicを利用したAWS WorkSpacesの多要素認証化](https://www.notion.so/PassLogic-AWS-WorkSpaces-3046605e29164d968e9b91fce1f8e3a2?pvs=21)

# FreeRadiusのパッケージのインストール

---

OSはAmazonLinux2023を利用しています。

EPELレポジトリをまずは追加したいところですが、AmazonLinux2023にはEPELリポジトリはサポートされていません。

なのでソースからRPMを作るというかなりイレギュラーなことを最初からします。

※RHELを選んでいればこんなことにはなりませんよ。

freeradiusのソースRPMからRPMを作成するためにソースのダウンロードとRPMビルトするためのパッケージを追加します。

また最後にQRコードを表示させるためのパッケージも追加します。

```jsx
# wget https://ftp.riken.jp/Linux/fedora/releases/40/Everything/source/tree/Packages/f/freeradius-3.2.3-4.fc40.src.rpm
# yum install rpm-build qrencode-libs
```

freeradiusのRPMを作成します。

```jsx
# rpmbuild --rebuild freeradius-3.2.3-4.fc40.src.rpm 
Installing freeradius-3.2.3-4.fc40.src.rpm
warning: freeradius-3.2.3-4.fc40.src.rpm: Header V4 RSA/SHA256 Signature, key ID a15b79cc: NOKEY
setting SOURCE_DATE_EPOCH=1706054400
error: Failed build dependencies:
        autoconf is needed by freeradius-3.2.3-4.amzn2023.x86_64
        chrpath is needed by freeradius-3.2.3-4.amzn2023.x86_64
        gcc is needed by freeradius-3.2.3-4.amzn2023.x86_64
        gdbm-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        json-c-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        krb5-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        libcurl-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        libpcap-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        libpq-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        libtalloc-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        libyubikey-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        make is needed by freeradius-3.2.3-4.amzn2023.x86_64
        mariadb-connector-c-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        net-snmp-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        net-snmp-utils is needed by freeradius-3.2.3-4.amzn2023.x86_64
        openldap-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        openssl-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        pam-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        perl(ExtUtils::Embed) is needed by freeradius-3.2.3-4.amzn2023.x86_64
        perl-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        perl-generators is needed by freeradius-3.2.3-4.amzn2023.x86_64
        python3-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        readline-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        sqlite-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        unixODBC-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        ykclient-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
        zlib-devel is needed by freeradius-3.2.3-4.amzn2023.x86_64
```

失敗しました。前提となるパッケージが足りないのエラー内容からインストールしていきます。

```jsx
# yum install -y autoconf chrpath gcc gdbm-devel json-c-devel krb5-devel libcurl-devel libpcap-devel libpq-devel libtalloc-devel libyubikey-devel make mariadb-connector-c-devel net-snmp-devel net-snmp-utils openldap-devel openssl-devel pam-devel perl-ExtUtils-Embed perl-devel perl-generators python3-devel readline-devel sqlite-devel unixODBC-devel ykclient-devel zlib-devel 
No match for argument: libyubikey-devel
No match for argument: ykclient-devel
Error: Unable to find a match: libyubikey-devel ykclient-devel
```

libyubikey-develとykclient-develはyumにないみたいなので、それ以外をインストールしてソースRPMをダウンロードしてビルドしてインストールします。

▪️libyubikey-devel

```jsx
# wget https://ftp.riken.jp/Linux/fedora/releases/40/Everything/source/tree/Packages/l/libyubikey-1.13-22.fc40.src.rpm
# rpmbuild --rebuild libyubikey-1.13-22.fc40.src.rpm
# rpm -ivh ./rpmbuild/RPMS/x86_64/libyubikey-1.13-22.amzn2023.x86_64.rpm 
# rpm -ivh ./rpmbuild/RPMS/x86_64/libyubikey-devel-1.13-22.amzn2023.x86_64.rpm 
```

▪️ykclient-devel（help2manパッケージが必要でした）

```jsx
# wget https://ftp.riken.jp/Linux/fedora/releases/40/Everything/source/tree/Packages/y/ykclient-2.15-18.fc40.src.rpm
# rpmbuild --rebuild ./ykclient-2.15-18.fc40.src.rpm 
Installing ./ykclient-2.15-18.fc40.src.rpm
warning: ./ykclient-2.15-18.fc40.src.rpm: Header V4 RSA/SHA256 Signature, key ID a15b79cc: NOKEY
setting SOURCE_DATE_EPOCH=1706313600
error: Failed build dependencies:
        help2man is needed by ykclient-2.15-18.amzn2023.x86_64
        
# yum instal -y help2man
# rpmbuild --rebuild ./ykclient-2.15-18.fc40.src.rpm 
# rpm -ivh ./rpmbuild/RPMS/x86_64/ykclient-2.15-18.amzn2023.x86_64.rpm
# rpm -ivh ./rpmbuild/RPMS/x86_64/ykclient-devel-2.15-18.amzn2023.x86_64.rpm 
```

今度こそfreeradiusのRPMを作成してインストールしますが、perl-DBI、perl-GDBM_File、perl-Net-IPが必要でした。。。

```jsx
# rpmbuild --rebuild freeradius-3.2.3-4.fc40.src.rpm 
# rpm -ivh ./rpmbuild/RPMS/x86_64/freeradius-3.2.3-4.amzn2023.x86_64.rpm
# rpm -ivh ./rpmbuild/RPMS/x86_64/freeradius-utils-3.2.3-4.amzn2023.x86_64.rpm 
error: Failed dependencies:
        perl(DBI) is needed by freeradius-utils-3.2.3-4.amzn2023.x86_64
        perl(GDBM_File) is needed by freeradius-utils-3.2.3-4.amzn2023.x86_64
        perl(Net::IP) is needed by freeradius-utils-3.2.3-4.amzn2023.x86_64
```

perl-DBI、perl-GDBM_File、perl-Net-IPをyumでインストールしましたがperl-Net-IPだけ失敗したので、ソースからビルドしてインストールします。

```jsx
# yum install -y perl-DBI perl-GDBM_File perl-Net-IP
No match for argument: perl-Net-IP
Error: Unable to find a match: perl-Net-IP

# wget https://ftp.riken.jp/Linux/fedora/releases/40/Everything/source/tree/Packages/p/perl-Net-IP-1.26-33.fc40.src.rpm
# rpmbuild --rebuild perl-Net-IP-1.26-33.fc40.src.rpm
# rpm -ivh ./rpmbuild/RPMS/noarch/perl-Net-IP-1.26-33.amzn2023.noarch.rpm
```

今度こそfreeradiusをインストールしてインストールできました。ふぅ。

```jsx
# rpm -ivh ./rpmbuild/RPMS/x86_64/freeradius-3.2.3-4.amzn2023.x86_64.rpm
# rpm -ivh ./rpmbuild/RPMS/x86_64/freeradius-utils-3.2.3-4.amzn2023.x86_64.rpm 
```

# FreeRadiusの起動テスト

---

rlm_eap_tls というEAP-TLS 認証をするための証明書を作成します。

※マニュアルは/etc/raddb/certs/README.mdに記載されています。

証明書の期限が60日となっているので100年に修正します。

```jsx
 # cd /etc/raddb/certs
 # vi ca.cnf
 # vi client.cnf
 # vi server.cnf
default_days            = 36500
default_crl_days        = 36500
```

証明書を作成します。

```jsx
# /etc/raddb/certs/bootstrap
```

FreeRadiusを起動します。

```jsx
# systemctl start radiusd
# systemctl status radiusd
● radiusd.service - FreeRADIUS high performance RADIUS server.
     Loaded: loaded (/usr/lib/systemd/system/radiusd.service; disabled; preset: disabled)
     Active: active (running) since Wed 2024-05-15 22:06:54 JST; 4s ago
    Process: 84381 ExecStartPre=/bin/chown -R radiusd.radiusd /var/run/radiusd (code=exited, status=0/SUCCESS)
    Process: 84382 ExecStartPre=/usr/sbin/radiusd -C (code=exited, status=0/SUCCESS)
    Process: 84384 ExecStart=/usr/sbin/radiusd -d /etc/raddb (code=exited, status=0/SUCCESS)
   Main PID: 84386 (radiusd)
      Tasks: 6 (limit: 2251)
     Memory: 41.0M
        CPU: 93ms
     CGroup: /system.slice/radiusd.service
             └─84386 /usr/sbin/radiusd -d /etc/raddb

May 15 22:06:54 ip-10-1-13-83.ap-northeast-1.compute.internal systemd[1]: Starting radiusd.service - FreeRADIUS high performance RADIUS server....
May 15 22:06:54 ip-10-1-13-83.ap-northeast-1.compute.internal systemd[1]: Started radiusd.service - FreeRADIUS high performance RADIUS server..
```

# Google Authenticatorをインストール

---

google-authenticatorをインストールした後にFreeRadiusを再起動します。

```jsx
# yum install -y google-authenticator
# systemctl restart radiusd
```

# ホスト名の設定

---

ホスト名はGoogleAuthenticatorに登録した登録名に利用されるのでホスト名を任意の名前に変更します。

```jsx
# hostnamectl set-hostname radius.ドメイン名
```

hostsファイルにも記載します。

```jsx
# vi /etc/hosts
IPアドレス  radius.ドメイン名
```

# FreeRadiusの設定

---

radiusd.confを修正します。

まずrootユーザーで起動させる必要があるため、以下のように変更します。

```jsx
# vi /etc/raddb/radiusd.conf 
512         user = root
513         group = root
```

続いてradius-enabledのユーザーがradius認証を経由するように設定します。

```jsx
# vi /etc/raddb/users
DEFAULT    Group != "radius-enabled", Auth-Type := Reject,Reply-Message = "Your account has been disabled."
DEFAULT    Auth-Type := PAM
```

最後にdegaultファイルでradius認証の際に利用するpamを有効化します。

```jsx
# vi /etc/raddb/sites-available/default
564         #  Pluggable Authentication Modules.
565         pam
```

# RadiusがGoogoleAuthenticatorを利用するようにする

---

GoogoleAuthenticatorが使われるように設定します。

```jsx
# vi /etc/pam.d/radiusd
#%PAM-1.0
#auth       include     password-auth
#account    required    pam_nologin.so
#account    include     password-auth
#password   include     password-auth
#session    include     password-auth
auth requisite pam_google_authenticator.so
account required pam_permit.so
session required pam_permit.so
```

# Radiusのクライアント設定

---

workspacesやMicrosoft ADが所属するVPCからのアクセスを許可します。

ipaddrはAWS VPC CIDRを記載、secretはDirectory Serviceに入れる設定になります。

```jsx
# vi /etc/raddb/clients.conf
client vpc {
        ipaddr = 10.1.0.0/16
        secret = jat2gcVXLFPKEZt 
}
```

# PAMモジュールの有効化

---

 PAMモジュールのリンクを登録します。

```jsx
# ln -s /etc/raddb/mods-available/pam /etc/raddb/mods-enabled/pam
```

# FreeRadiusの再起動

---

FreeRadiusを再起動と自動起動を有効化します。

```jsx
# systemctl restart radiusd
# systemctl enable radiusd
```

# WorkSpacesにログインするユーザーを追加

---

ADでnnagashimaというユーザーを作成しているので、Radiusに登録しま。

```jsx
# groupadd radius-enabled
# useradd -g radius-enabled nnagashima
# sudo -u nnagashima /usr/bin/google-authenticator
対話式表示されますが、基本yで進んでください。
QRコードが表示されるのでスマホでGoogle Authenticatorアプリを起動してスキャンします。
```

# MSADにて多要素認証を有効化

---

対象のディレクトリIDに移動して、多要素認証から有効にして以下の図のように設定します。

DirectoryServiceのセキュリティグループのアウトバウンドを開放していない場合は忘れずに開放してください。

シークレットコードはRadiusのクライアント設定で設定したsecretの内容を記載します。

![](/AWS/FreeRadiusでMSADと多要素認証連携/Untitled.png)

![](/AWS/FreeRadiusでMSADと多要素認証連携/Untitled.png)

# WorkSpacesへのログイン

---

AD認証に加えて、MFAコードを入力する欄が出ているのでGoogleAuthenticatorを起動して入力して、

WorkSpaceへログインができればOKです。（ADアカウント名@ドメイン名）

# ここまでを振り返って

---

- AmazonLinux2023で作るにはちとめんどくさいですね。
- 複数ドメイン管理ができないのが辛いですね。（有償ソフトウェアとかが選択肢？）
- ADとFreeRadiusでユーザー管理が別になるので忘れそうですね。
- シングル構成を冗長構成にしたいですね。

# ADとFreeRadiusでユーザー管理が別になるを解消

---

AmazonLinux2023をADドメインへ参加させるためにDNSの向き先をMSADにします。

```jsx
# vi /etc/systemd/resolved.conf
DNS=MSADのIPアドレス1 MSADのIPアドレス2
# systemctl restart systemd-resolved.service
```

AD参加に必要なパッケージを追加してドメイン参加します。

```jsx
# yum install -y sssd realmd krb5-workstation
# yum install -y oddjob oddjob-mkhomedir sssd adcli
# realm join -U Admin@ドメイン名(大文字) ドメイン名(大文字) --verbose
* Successfully enrolled machine in realm
```

ADに結合されたことを確認

```jsx
# id nnagashima@aws-actmotech.local
uid=644201146(nnagashima@aws-actmotech.local) gid=644200513(domain users@aws-actmotech.local) groups=644200513(domain users@aws-actmotech.local),644201139(aws delegated add workstations to domain users@aws-actmotech.local)
```

ADユーザーのQRコード発行

```jsx
# sudo -u nnagashima@aws-actmotech.local /usr/bin/google-authenticator
```

/home配下にnnagashima@aws-actmotech.localがいることを確認

```jsx
# ls -la /home/nnagashima@aws-actmotech.local
```

WorkSpacesへのログインできれば完了です。

これでユーザー作成は省略することができ、QRコード発行のみで処理可能です。

```jsx
WorkSpacesログイン時のユーザー名はドメイン名付きで入力することを忘れないようにしてください。
```

# シングル構成を冗長構成にしたいを解消

---

EC2にEFSをマウントします。(/mnt配下とします。)

GoogleAuthenticatorの出力先を変更します。

```jsx
# vi /etc/pam.d/radiusd
auth requisite pam_google_authenticator.so secret=/mnt/home/%u/.google_authenticator
account required pam_permit.so
session required pam_permit.so
```

sssd.confファイルを修正します。

```jsx
# vi /etc/sssd/sssd.conf
#fallback_homedir = /home/%u@%d
fallback_homedir = /mnt/home/%u@%d
```

sssdサービスを再起動します。

```jsx
# systemctl restart sssd
```

これを2台目のFreeRadiusを作成するには、EFSをマウントすることで完了となります。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
