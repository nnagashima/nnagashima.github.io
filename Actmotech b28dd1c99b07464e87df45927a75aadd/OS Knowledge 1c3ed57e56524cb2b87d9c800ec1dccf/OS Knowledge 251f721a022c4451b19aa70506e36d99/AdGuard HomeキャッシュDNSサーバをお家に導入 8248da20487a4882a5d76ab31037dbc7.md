# AdGuard HomeキャッシュDNSサーバをお家に導入

![ubuntu_14143.png](Guacamole%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E7%92%B0%E5%A2%83%E3%82%92%E6%95%B4%E3%81%88%E3%82%8B%2006607b0254a146f2926a6087ee4ab548/ubuntu_14143.png)

目次

# はじめに

---

今まで自宅ではUnboundでDNSキャッシュサーバ運用していましたが、Unboundにさまざまなサイトのブロックリストを喰わせるのに、

スクリプト書いたり、ブロックされたドメインのチェックをするのがUIでできないのでめんどくさいなと思っていたところ、

AdGuardHomeでDNSキャッシュサーバ兼ドメインブロックができることを知り、UnboundからAdGuardに切り替えたことを備忘録として残す。

なお、UnboundはADGuardのアップストリームとして継続して利用しています。

- 以前のキャッシュDNSサーバ

[自宅にキャッシュDNSサーバ](https://blog.actmotech.xyz/d21b533870cc41d9ad4f6f12dc24c1bf)

# UnboudとADGuardを組み合わせてやりたこと

---

- Unboudは軽量でありパラメータチューニングが可能なため、リゾルバとしての動作を高速化させる。
- ADGuardは機能通り、トラッキングリクエストや広告をフィルタリングすることでマルウェアからの保護を向上させる。

# 構築環境

---

- 自宅ハイパーバイザー：Proxmox8.2
- OS：RockyLinux9の冗長構成
- 何かしらのドメインを取得していること（ADGuardでDoHをやりたい場合）

# OSセットアップ

---

以下を参考にしてください。

[UbuntuOSの初期設定](https://blog.actmotech.xyz/f8933b6457e64efcbc802962379dca6c)

# AdGuard Homeインストール

---

1号機(192.168.21.5)と2号機(192.168.21.6)で同じことを実施します。

```bash
# apt install curl
# curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
starting AdGuard Home installation script
channel: release
operating system: linux
cpu type: amd64
AdGuard Home will be installed into /opt/AdGuardHome
checking tar
script is executed with root privileges
no need to uninstall
downloading package from https://static.adtidy.org/adguardhome/release/AdGuardHome_linux_amd64.tar.gz -> AdGuardHome_linux_amd64.tar.gz
successfully downloaded AdGuardHome_linux_amd64.tar.gz
unpacking package from AdGuardHome_linux_amd64.tar.gz into /opt
successfully unpacked, contents: 
合計 31580
-rwxrwxrwx 1 root root 32149656  6月  6 23:36 AdGuardHome
-rw-rw-rw- 1 root root      566  6月  6 23:36 AdGuardHome.sig
-rw-r--r-- 1 root root   115641  6月  6 23:36 CHANGELOG.md
-rw-r--r-- 1 root root    35149  6月  6 23:36 LICENSE.txt
-rw-r--r-- 1 root root    22128  6月  6 23:36 README.md
2024/06/19 01:58:14 [info] AdGuard Home, version v0.107.51
2024/06/19 01:58:14 [info] service: control action: install
2024/06/19 01:58:15 [info] service: started
2024/06/19 01:58:15 [info] Almost ready!
AdGuard Home is successfully installed and will automatically start on boot.
There are a few more things that must be configured before you can use it.
Click on the link below and follow the Installation Wizard steps to finish setup.
AdGuard Home is now available at the following addresses:
2024/06/19 01:58:15 [info] go to http://127.0.0.1:3000
2024/06/19 01:58:15 [info] go to http://[::1]:3000
2024/06/19 01:58:15 [info] go to http://192.168.21.5:3000
2024/06/19 01:58:15 [info] service: action install has been done successfully on linux-systemd
AdGuard Home is now installed and running
you can control the service status with the following commands:
sudo /opt/AdGuardHome/AdGuardHome -s start|stop|restart|status|install|uninstall
```

インストール完了したらhttp://IPアドレス:3000でアクセスを実施し、ウィザードに従って入力していきます。

表示された内容は以下です。（スクショ撮り忘れました）

- 管理用待ち受けインターフェースの設定と待ち受けポート番号
- DNSサーバ待ち受けインターフェースの設定と待ち受けポート番号
- 管理用UIにログインする際のユーザー名とパスワード

# ADGuardのDoH設定に関わる補足情報

---

このあと同期設定を実施しますが本設定については同期はされないのでDoH及びDoTを有効化するのに証明書が必要となるためLet`s Encryptを利用します。

※ DNS over TLS（DoT）とは、DNSにTLSを組み合わせ、ドメイン名やIPアドレスなどの問い合わせや応答を暗号化する通信規約です。

1号機(192.168.21.5)と2号機(192.168.21.6)で同じことを実施します。

なお、Let`s Encryptで証明書を更新する際にCronに記載していた記憶があるのですが、

今はCertbotに組み込まれたタイマーを使って簡単にSSL証明書の更新をしてくれるらしいです。（systemctl status certbot.timer）

```bash
# apt -y install certbot
# certbot certonly --manual --preferred-challenges=dns --preferred-chain="ISRG Root X1"
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): 登録するドメイン名を記載
Requesting a certificate for 登録するドメイン名

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

_acme-challenge.登録するドメイン名

with the following value:

【DNS TXT record】

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.登録するドメイン名.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

※ここでTXTレコードの登録を促されているので、ドメインを管理画面でTXTレコードを登録してくさい。
※登録後しばらくすると配信されるので待ってからEnterを押してください。

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/登録するドメイン名/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/登録するドメイン名/privkey.pem
This certificate expires on 2024-09-17.
These files will be updated when the certificate renews.

NEXT STEPS:
- This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

# systemctl status certbot.timer
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/usr/lib/systemd/system/certbot.timer; enabled; preset: enabled)
     Active: active (waiting) since Wed 2024-06-19 22:23:42 JST; 9min ago
    Trigger: Thu 2024-06-20 09:23:33 JST; 10h left
   Triggers: ● certbot.service

 6月 19 22:23:42 adguard02 systemd[1]: Started certbot.timer - Run certbot twice daily.
```

# AdGuard Homeの設定 - **アップストリームDNSサーバー設定**

---

1号機(192.168.21.5)で実施します。

AdGuard Homeの問い合わせ先として使用するDNSサーバを設定します。

設定＞DNS設定＞アップストリームDNSサーバで以下の内容で反映します。

```bash
127.0.0.1:8953
```

※リクエストモードは「並列リクエスト」にすることで解決スピードを向上させます。

# AdGuard Homeの設定 - 暗号化設定をする場合の情報

---

1号機(192.168.21.5)で実施します。

設定＞暗号化設定で設定をしていきます。

- 暗号化を有効にする（HTTPS、DNS-over-HTTPS、DNS-over-TLS）にチェックオンにします。
- サーバ名はSSL証明書を取得する際に利用したドメイン名にします。
- 証明書では
    - 証明書のパスを設定するを選択し、fullcahin.pemのファイルパスを記載します。
    - 秘密鍵のパスを設定するを選択し、privkey.pemのファイルパスを記載します。

```bash
/etc/letsencrypt/live/登録するドメイン名/fullchain.pem
/etc/letsencrypt/live/登録するドメイン名/privkey.pem
```

# AdGuard Homeの設定 - DNSブロックリストの追加

---

1号機(192.168.21.5)で実施します。

インストール直後のリストは以下となっていました。

```bash
・AdGuard DNS filterが有効
https://adguardteam.github.io/HostlistsRegistry/assets/filter_1.txt
・AdAway Default Blocklistが無効
https://adguardteam.github.io/HostlistsRegistry/assets/filter_2.txt
```

上記に以下内容を加えました。

```bash
・Steven Black's List（広告やトラッキング、および悪意のあるドメインをブロックするためのホストファイルリスト）
https://adguardteam.github.io/HostlistsRegistry/assets/filter_33.txt
・豆腐フィルタ（有名ですよね）
https://raw.githubusercontent.com/tofukko/filter/master/Adblock_Plus_list.txt
```

リストについてはAdGuardHomeにて定期的に最新化したものをダウンロードしてくれます。

# AdGuard Homeの設定 - 同期設定（AdguardHome-sync）

---

1号機(192.168.21.5)で設定を実施します。

```tsx
# mkdir adguardhome-sync && cd adguardhome-sync
# curl -OL https://github.com/bakito/adguardhome-sync/releases/download/v0.6.11/adguardhome-sync_0.6.11_linux_amd64.tar.gz
# tar zxvf adguardhome-sync_0.6.11_linux_amd64.tar.gz
# vi ~/.bash_profile
export LOG_LEVEL=info
export ORIGIN_URL=http://192.168.21.5 #1号機
export ORIGIN_USERNAME=ray-adm #1号機で作成したアカウント
export ORIGIN_PASSWORD=****** #1号機で作成したパスワード
export REPLICA1_URL=http://192.168.21.6 #2号機
export REPLICA1_USERNAME=ray-adm #2号機で作成したアカウント
export REPLICA1_PASSWORD=****** #2号機で作成したパスワード
# source .bash_profile
```

手動でテスト実行し同期が実施できるか確認します。

```bash
# ./adguardhome-sync run
2024-06-19T21:23:44.587+0900    INFO    sync    sync/sync.go:38 AdGuardHome sync        {"version": "0.6.11", "build": "2024-05-25T06:15:56Z", "os": "linux", "arch": "amd64"}
2024-06-19T21:23:44.588+0900    INFO    sync    sync/http.go:68 Starting API server     {"port": 8080}
2024-06-19T21:23:44.588+0900    INFO    sync    sync/sync.go:75 Running sync on startup
2024-06-19T21:23:44.639+0900    INFO    sync    sync/sync.go:174        Connected to origin     {"from": "192.168.21.5", "version": "v0.107.51"}
2024-06-19T21:23:45.352+0900    INFO    sync    sync/sync.go:267        Start sync      {"from": "192.168.21.5", "to": "192.168.21.6"}
2024-06-19T21:23:45.404+0900    INFO    sync    sync/sync.go:275        Connected to replica    {"from": "192.168.21.5", "to": "192.168.21.6", "version": "v0.107.51"}
2024-06-19T21:23:45.505+0900    INFO    client  client/client.go:437    Set profile     {"host": "192.168.21.6", "language": "ja", "theme": "auto"}
2024-06-19T21:23:45.813+0900    INFO    client  client/client.go:338    Set stats config        {"host": "192.168.21.6", "interval": 7776000000}
2024-06-19T21:23:45.969+0900    INFO    client  client/client.go:241    Add filter      {"host": "192.168.21.6", "url": "https://adguardteam.github.io/HostlistsRegistry/assets/filter_33.txt", "whitelist": false, "enabled": true}
2024-06-19T21:23:46.291+0900    INFO    client  client/client.go:241    Add filter      {"host": "192.168.21.6", "url": "https://raw.githubusercontent.com/tofukko/filter/master/Adblock_Plus_list.txt", "whitelist": false, "enabled": true}
2024-06-19T21:23:46.397+0900    INFO    client  client/client.go:262    Refresh filter  {"host": "192.168.21.6", "whitelist": false}
2024-06-19T21:23:47.196+0900    INFO    client  client/client.go:276    Set user rules  {"host": "192.168.21.6", "rules": 0}
2024-06-19T21:23:47.424+0900    INFO    client  client/client.go:375    Set access list {"host": "192.168.21.6"}
2024-06-19T21:23:47.593+0900    INFO    sync    sync/sync.go:303        Sync done       {"from": "192.168.21.5", "to": "192.168.21.6"}
```

手動実行ができればあとはCrontabに記載するのみです。（毎日0時に1号機から2号機へ同期実行）

```bash
# crontab -e
LOG_LEVEL=info
ORIGIN_URL=http://192.168.21.5
ORIGIN_USERNAME=ray-adm
ORIGIN_PASSWORD=******
REPLICA1_URL=http://192.168.21.6
REPLICA1_USERNAME=ray-adm
REPLICA1_PASSWORD=******
00 00 * * * /root/adguardhome-sync/adguardhome-sync run

※ctrl+sで保存、ctrl+xで終了
```

# 最後に管理画面の画像

---

日本語対応しているし、どのドメインをブロックしているのかや、どのIPからどういったリクエストがあったのかをUIで確認できるのは非常に便利です。

統計保持で最大365日保持することが可能となります。（設定は時間単位になるので8760時間）

なにより端末毎にいれなくていいのがいいですよね！その代わりサーバが増えることになりますが。。。

![Untitled](AdGuard%20Home%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5DNS%E3%82%B5%E3%83%BC%E3%83%8F%E3%82%99%E3%82%92%E3%81%8A%E5%AE%B6%E3%81%AB%E5%B0%8E%E5%85%A5%208248da20487a4882a5d76ab31037dbc7/Untitled.png)

# Unboudの設定

---

ADGuardのサーバ上で設定をしていきます。（1号機と2号機両方で実施ください。）

設定内容については初期インストールを想定して記載しています。

Unboudとdigコマンドをインストールします。

```jsx
# dnf install unbound bind-utils
```

ベースとなる設定ファイルを作成します。

```jsx

# vi /etc/unbound/conf.d/base.conf
server:
    interface: 127.0.0.1@8953
    interface: 127.0.0.1@5353

    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.11.0/24 allow
    access-control: 192.168.12.0/24 allow
    access-control: 192.168.21.0/24 allow

    interface-automatic: no 

    module-config: "respip validator iterator"

    val-permissive-mode: yes

    hide-version: yes
    hide-identity: yes

    root-hints: "/var/lib/unbound/root.hints"

    do-ip6: no 

    use-syslog: no
    logfile: /var/log/unbound/unbound.log
    log-replies: yes
    log-local-actions: yes
    log-queries: yes
    log-time-ascii: yes
    extended-statistics: yes 

forward-zone:
    name: "."
    forward-addr: 1.1.1.1
    forward-addr: 8.8.8.8 

remote-control:
    control-interface: "/var/run/unbound-control.sock" 

```

所有者を変更します。

```jsx
# chown root:unbound base.conf
```

保管先ログフォルダを作成します。

```jsx
# mkdir /var/log/unbound
# chown unbound:unbound /var/log/unbound
```

DNSSEC（DNS Security Extensions）で使用される鍵ファイルの所有者を変更します。

```jsx
# chown unbound:unbound /etc/unbound/root.key 
```

元の設定ファイルの後ろにorgをつけます。

```jsx
# pwd
/etc/unbound/conf.d
# ls
base.conf  example.com.conf.org  remote-control.conf.org  tuning.conf
```

チューニング設定をします。

```jsx
# cat tuning.conf 
server:
    msg-cache-slabs: 8
    rrset-cache-slabs: 8
    infra-cache-slabs: 8
    key-cache-slabs: 8

    rrset-cache-size: 512m
    msg-cache-size: 128m

    outgoing-num-tcp: 1000
    incoming-num-tcp: 1000

    outgoing-range: 8192
    num-queries-per-thread: 4096

    so-rcvbuf: 4m
    so-sndbuf: 4m

    minimal-responses: yes
    qname-minimisation: yes

    infra-cache-numhosts: 1000000
```

各パラメータの解説を記載しておきます。

- msg-cache-slabs: 8:

メッセージキャッシュに使用するスラブの数を指定します。スラブはメモリ管理手法の一つで、特定サイズのオブジェクトを効率的に管理します。この設定では、8つのスラブを使用します。

- rrset-cache-slabs: 8:

リソースレコードセット（RRset）のキャッシュに使用するスラブの数を指定します。

- infra-cache-slabs: 8:

インフラキャッシュに使用するスラブの数を指定します。インフラキャッシュは、特定のホスト情報をキャッシュして、名前解決を迅速に行うために使用されます。

- key-cache-slabs: 8:

鍵キャッシュに使用するスラブの数を指定します。

- rrset-cache-size: 512m:

リソースレコードセットのキャッシュサイズを512MBに設定します。キャッシュサイズが大きいほど、DNSクエリの応答が高速になりますが、使用するメモリも増加します。

- msg-cache-size: 128m:

メッセージキャッシュのサイズを128MBに設定します。

- outgoing-num-tcp: 1000:

TCP接続でのアウトゴーイングクエリの最大数を1000に設定します。

- incoming-num-tcp: 1000:

TCP接続でのインカミングクエリの最大数を1000に設定します。

- outgoing-range: 8192:

アウトゴーイングトラフィックに使用できるポート範囲を8192に設定します。これにより、同時に処理できる接続の数が増加します。

- num-queries-per-thread: 4096:

各スレッドが処理できる最大クエリ数を4096に設定します。これにより、スレッドの負荷を制御できます。

- so-rcvbuf: 4m:

ソケット受信バッファのサイズを4MBに設定します。受信したデータをバッファリCOんｆングするためのメモリ領域です。

- so-sndbuf: 4m:

ソケット送信バッファのサイズを4MBに設定します。送信するデータをバッファリングするためのメモリ領域です。

- minimal-responses: yes:

クエリに対する応答を最小限にするオプションを有効にします。これにより、不要な情報を含まない応答が返され、プライバシーとパフォーマンスが向上します。

- qname-minimisation: yes:

名前解決の際に、クエリの名前を最小限に抑える設定を有効にします。これにより、プライバシーが向上します。具体的には、DNSクライアントが必要な情報だけをクエリするようになります。

- infra-cache-numhosts: 1000000:

インフラキャッシュに保存できるホストの最大数を100万に設定します。この設定により、大規模なネットワーク環境でのキャッシュの効率が向上します。

所有者を変更します。

```jsx
# chown root:unbound tuning.conf
```

最後に残りのunboudのパラメータを作成します。

```jsx
# pwd
/etc/unbound

# vi unbound.conf
server:
        verbosity: 2 
        statistics-interval: 0
        statistics-cumulative: no
        extended-statistics: yes
        num-threads: 4
        interface-automatic: no
        outgoing-port-permit: 32768-60999
        outgoing-port-avoid: 0-32767
        so-reuseport: yes
        ip-transparent: yes
        max-udp-size: 3072
        chroot: ""
        username: "unbound"
        directory: "/var/lib/unbound"
        log-time-ascii: yes
        harden-glue: yes
        harden-dnssec-stripped: yes
        harden-below-nxdomain: yes
        harden-referral-path: yes
        qname-minimisation: yes
        aggressive-nsec: yes
        unwanted-reply-threshold: 10000000
        prefetch: yes
        prefetch-key: yes
        rrset-roundrobin: yes
        minimal-responses: yes
        trust-anchor-signaling: yes
        root-key-sentinel: yes
        val-clean-additional: yes
        serve-expired: yes
        val-log-level: 1
        ipsecmod-enabled: no
        ipsecmod-hook:/usr/libexec/ipsec/_unbound-hook
        auto-trust-anchor-file: "/var/lib/unbound/root.key"
        include: /etc/unbound/conf.d/*.conf

remote-control:
        control-enable: yes
        server-key-file: "/etc/unbound/unbound_server.key"
        server-cert-file: "/etc/unbound/unbound_server.pem"
        control-key-file: "/etc/unbound/unbound_control.key"
        control-cert-file: "/etc/unbound/unbound_control.pem"
```

Unboundで秘密鍵と公開鍵を作成します。

```jsx
# systemctl start unbound-keygen
```

root.hintをダウンロードします。

```jsx
# curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
```

UnboundのConfigをチェックします。

```jsx
# unbound-checkconf 
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

Unboudのulimitを変更して、Unboundサービスを起動します。

```jsx

# vi /lib/systemd/system/unbound.service
[Unit]
Description=Unbound recursive Domain Name Server
After=network.target
After=unbound-keygen.service
Wants=unbound-keygen.service
Wants=unbound-anchor.timer
Before=nss-lookup.target
Wants=nss-lookup.target

[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/unbound
**ExecStartPre=/bin/ulimit -HSn 65536**
ExecStartPre=/usr/sbin/unbound-checkconf
ExecStartPre=/bin/bash -c 'if [ ! "$DISABLE_UNBOUND_ANCHOR" == "yes" ]; then /usr/sbin/unbound-anchor -a /var/lib/unbound/root.key -c /etc/unbound/icannbundle.pem -f /etc/resolv.conf -R; else echo "Updates of root keys with unbound-anchor is disabled"; fi'
ExecStart=/usr/sbin/unbound -d $UNBOUND_OPTIONS
ExecReload=/usr/sbin/unbound-control reload

[Install]
WantedBy=multi-user.target

# systemctl deamon-reload
# systemctl --now enable unbound
```