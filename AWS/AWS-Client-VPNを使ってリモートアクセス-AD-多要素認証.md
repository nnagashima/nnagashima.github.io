[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# AWS Client VPNを使ってリモートアクセス(AD+多要素認証を使う)

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/image01.jpg)

# 目次
- [AWS Client VPNを使ってリモートアクセス(AD+多要素認証を使う)](#aws-client-vpnを使ってリモートアクセスad多要素認証を使う)
- [目次](#目次)
- [はじめに](#はじめに)
- [AWSでCA認証局について](#awsでca認証局について)
- [認証局の作成とクライアントの証明書とキーを生成](#認証局の作成とクライアントの証明書とキーを生成)
- [クライアント証明書の更新方法](#クライアント証明書の更新方法)
- [サーバー証明書の更新方法（CA更新は含まない）](#サーバー証明書の更新方法ca更新は含まない)
- [サーバー証明書の更新方法（CA更新を含む）](#サーバー証明書の更新方法ca更新を含む)
- [Client VPN Endpoint の作成](#client-vpn-endpoint-の作成)
- [作成したVPNエンドポイントにサブネットを関連付け](#作成したvpnエンドポイントにサブネットを関連付け)
- [作成したVPNエンドポイントに承認ルールを追加](#作成したvpnエンドポイントに承認ルールを追加)
- [作成したVPNエンドポイントに接続元制限](#作成したvpnエンドポイントに接続元制限)
- [クライアント側の設定](#クライアント側の設定)
- [最後に](#最後に)


---

# はじめに

---

AWS Client VPNを利用して、AWSリソースへのアクセス方法について調べてみたのでまとめて、 

実際にOpenVPNを利用してAWSへアクセスも行ってみました。

# AWSでCA認証局について

---

ここのURLに書いてある通り、AWSでプライベート認証機関を用意すると400ドルかかる上に、 

プライベート証明書の数ごとに料金が発生するのでコスト節約の観点オレオレ認証局を立てて実施します。 

[AWS Certificate Manager の料金](https://aws.amazon.com/jp/certificate-manager/pricing/?nc=sn&loc=3)

# 認証局の作成とクライアントの証明書とキーを生成

---

ADによるSimple ADを利用した接続方法もありますが、今回は相互認証を利用した接続を実施したいと思います。 

[Client authentication in AWS Client VPN](https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/client-authentication.html)

OSはAmazonLinux2にて実施しています。 またEC2に対してAWSCertificateManagerFullAccessのロールを割り当てるようにしてください。

※後程AWSコマンドでACM にインポートするため

gitコマンドがインストールしていない場合にはgitコマンドをインストールしてください。 

基本的には上記URLに記載のある通り実施していきます。

```bash
gitのインストール
$ sudo dnf install git

証明書を作成するフォルダに移動
$ cd /usr/local/src

リポジトリをダウンロード
$ sudo git clone https://github.com/OpenVPN/easy-rsa.git

フォルダに移動
$ cd easy-rsa/easyrsa3/
$ pwd
/usr/local/src/easy-rsa/easyrsa3

CAの有効期間を100年にする
$ sudo cp -p vars.example vars
$ sudo vi vars
$ diff vars.example vars
138c138
< #set_var EASYRSA_CA_EXPIRE	3650
---
> set_var EASYRSA_CA_EXPIRE	36500

新しい PKI 環境を初期化
$ sudo ./easyrsa init-pki

新しい認証局 (CA) を構築
$ sudo ./easyrsa build-ca nopass → COMMONネームを記載:ca-actmotech.local

CA情報の確認
$ sudo openssl x509 -text -noout -in pki/ca.crt

サーバー証明書作成(サーバ名は任意)
$ sudo ./easyrsa --days=36500 build-server-full actmotech.local nopass
**※easy-rsa 3.2.0からドメイン名で作成しない場合、server subject alternative nameが削除されたので、alternativeを明示的に指定する必要があります。
例：sudo ./easyrsa --days=36500 --san=DNS:actmotech-local build-server-full actmotech-local nopass**

クライアント証明書の作成(Client名は任意)
$ sudo ./easyrsa --days=365 build-client-full client.actmotech.local nopass

サーバ証明書、クライアント証明書の有効期限を確認します。
$ sudo openssl x509 -noout -subject -dates -in ./pki/issued/actmotech.local.crt
$ sudo openssl x509 -noout -subject -dates -in ./pki/issued/client.actmotech.local.crt

カスタムフォルダ作成
$ sudo mkdir published

認証局のCA証明書、サーバ証明書、キーをカスタムフォルダにコピー
$ sudo cp -p pki/ca.crt published/
$ sudo mv pki/issued/actmotech.local.crt published/
$ sudo mv pki/private/actmotech.local.key published/

作成したクライアント証明書をカスタムフォルダにコピー
$ sudo mv pki/issued/client.actmotech.local.crt published/
$ sudo mv pki/private/client.actmotech.local.key published/

acmの画面にてサーバ証明書とクラインと証明書をアップロードします。
※サーバ証明書はクライアントVPNエンドポイントを作成しているリージョンのACMに存在するようにしてください。

もしくはCLIで実施する場合は。。。

カスタムフォルダに移動して、ACMへインポート
$ aws acm import-certificate --certificate file://actmotechserver.crt --private-key file://actmotechserver.key --certificate-chain file://ca.crt --region ap-northeast-1

カスタムフォルダに移動して、ACMへインポート
$ aws acm import-certificate --certificate file://client1.actmotech.xyz.crt --private-key file://client1.actmotech.xyz.key --certificate-chain file://ca.crt --region ap-northeast-1
```

# クライアント証明書の更新方法

---

証明書は期限を迎えたら使えなくなるので、更新方法についても記載します。

CAは初期作成時と同じCAを利用した場合となります。

以下コマンドを実行するとエラーが返ってきます。これはすでに証明書要求があることによるエラーなので、

ファイル名を変えて作成するか、同じ名前で作成したい場合には「client.actmotech.local.req」を削除します。

```bash
$ sudo ./easyrsa --days=400 build-client-full client.actmotech.local nopass
Using Easy-RSA 'vars' configuration:
* /usr/local/src/easy-rsa/easyrsa3/vars

EasyRSA version ~VER~

Error
-----
Request file already exists. Aborting build to avoid overwriting this file.
If you wish to continue, please use a different name.
Conflicting file found at:
* /usr/local/src/easy-rsa/easyrsa3/pki/reqs/client.actmotech.local.req
```

削除後にもう一度コマンドを実行してみます。途中で回答を求められるところはyもしくはyesで回答します。

そうすると新しい有効期限でクライアント証明書が発行することができます。

```bash
$ sudo ./easyrsa --days=400 build-client-full client.actmotech.local nopass
Using Easy-RSA 'vars' configuration:
* /usr/local/src/easy-rsa/easyrsa3/vars
Warning!

An inline file for name 'client.actmotech.local' already exists:
* /usr/local/src/easy-rsa/easyrsa3/pki/inline/client.actmotech.local.inline

Type the word 'y' to continue, or any other input to abort.
  Confirm OVER-WRITE existing inline file ? y
......
Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
......
Write out database with 1 new entries
Data Base Updated

Notice
------
Certificate created at:
* /usr/local/src/easy-rsa/easyrsa3/pki/issued/client.actmotech.local.crt

Notice
------
Inline file created:
* /usr/local/src/easy-rsa/easyrsa3/pki/inline/client.actmotech.local.inline
```

# サーバー証明書の更新方法（CA更新は含まない）

---

サーバ証明書を100年とか作っていれば更新することはないのですが、

期限を設けてサーバ証明書を作っている場合には期限切れが発生するので更新する必要があります。

以下コマンドを実行するとエラーが返ってきます。これはすでに証明書要求があることによるエラーなので、

ファイル名を変えて作成するか、同じ名前で作成したい場合には「actmotech.local.req」を削除します。

```bash
$ sudo ./easyrsa --days=365000 build-server-full actmotech.local nopass
Using Easy-RSA 'vars' configuration:
* /usr/local/src/easy-rsa/easyrsa3/vars

EasyRSA version ~VER~

Error
-----
Request file already exists. Aborting build to avoid overwriting this file.
If you wish to continue, please use a different name.
Conflicting file found at:
* /usr/local/src/easy-rsa/easyrsa3/pki/reqs/actmotech.local.req
```

削除後にもう一度コマンドを実行してみます。途中で回答を求められるところはyもしくはyesで回答します。

そうすると新しい有効期限でサーバー証明書が発行することができます。

```jsx
$ sudo ./easyrsa --days=365000 build-server-full actmotech.local nopass
Using Easy-RSA 'vars' configuration:
* /usr/local/src/easy-rsa/easyrsa3/vars
Warning!

An inline file for name 'actmotech.local' already exists:
* /usr/local/src/easy-rsa/easyrsa3/pki/inline/actmotech.local.inline

Type the word 'y' to continue, or any other input to abort.
  Confirm OVER-WRITE existing inline file ? y

-----

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /usr/local/src/easy-rsa/easyrsa3/pki/reqs/actmotech.local.req
* key: /usr/local/src/easy-rsa/easyrsa3/pki/private/actmotech.local.key

You are about to sign the following certificate:
Request subject, to be signed as a server certificate
for '365000' days:

subject=
    commonName                = actmotech.local

Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /usr/local/src/easy-rsa/easyrsa3/pki/3d6852c1/temp.4.1
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'actmotech.local'
Certificate is to be certified until Sep  2 13:14:26 3023 GMT (365000 days)

Write out database with 1 new entries
Data Base Updated

Notice
------
Certificate created at:
* /usr/local/src/easy-rsa/easyrsa3/pki/issued/actmotech.local.crt

Notice
------
Inline file created:
* /usr/local/src/easy-rsa/easyrsa3/pki/inline/actmotech.local.inline
```

作成した証明書をACMにインポートしてクライアントVPNエンドポイントのサーバ証明書を切り替えます。

なお、クライアントVPNのConfigはCAが含まれていますが初期構築時のCAを利用している場合には、

Configを再ダウンロードする必要がない為、既存のConfigで接続することができます。

※サーバ証明書の期限が切れる前で実施している為、期限切れ後に証明書を入れ替えた場合はある程度時間が経過しないと接続ができないかもしれません。

# サーバー証明書の更新方法（CA更新を含む）

---

初期構築と同じことを実施します。

作成した証明書をACMを再インポートを実施しクライアントVPNエンドポイントのサーバ証明書を切り替えます。

前述でも書いている通りクライアントVPNのConfigはCAが含まれているのでConfigをダウンロードし直し、

ClientVPNアプリから接続を実施しましたが即時接続できませんでした。

その為、即時で切り替えたい場合には新しくクライアントVPNエンドポイントを作成して接続し、

旧クライアントVPNエンドポイントを削除するやり方になってしまうかと思います。

※サーバ証明書の期限切れの場合も多分同じになるかと思います。（検証は未実施）

※初期構築時にインポートしたACMに新しい証明書を再インポートでも試してみましたが即時で繋げることはできませんでした。

（新しいACMが反映されるまでに最大24時間かかるとのことでした。）

# Client VPN Endpoint の作成

本題に戻って、Client VPN Endpointを作成します。

---

AWS Consoleにて、VPCサービスからClient VPN Endpointを作成します。

名前タグには任意の名前をクライアント IPv4 CIDR*には接続時の NAT で払い出される IP アドレスの範囲を設定しますので、 

クライアントネットワーク、接続するAWSネットワークと被らないようにしてください。 CIDRは/16-/22の範囲で設定が可能です。

サーバ証明書ARNには、先ほどアップロードしたサーバー証明書を選択し、認証オプションには相互認証の使用にチェックを入れます。 

クライアント証明書ARNには、先ほどアップロードしたサーバー証明書を選択し、認証オプションには相互認証の使用にチェックを入れます。 

```bash
AWS公式ドキュメントより：
サーバー証明書は AWS Certificate Manager (ACM) にアップロードし、クライアント VPN エンドポイントの作成時に指定する必要があります。
サーバー証明書を ACM にアップロードするときは、認証局 (CA) も指定します。
クライアント証明書を ACM にアップロードする必要があるのは、クライアント証明書の CA がサーバー証明書の CA と異なる場合だけです。
ACM の詳細については、AWS Certificate Manager ユーザーガイドを参照してください。
```

ユーザーベースの認証も使用するので、事前に作っているActive DirectoryのディレクトリIDを使用しています。

クライアントログを保存したい場合にはCloudWatchLogsに保存ができます。 

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/Untitled.png)

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/Untitled1.png)

クライアント接続ハンドラは接続元IPによるアクセス制御を行いたい場合など有効にします。

```bash
クライアント VPN エンドポイントのクライアント接続ハンドラーを設定できます。
ハンドラーを使用すると、デバイス、ユーザー、接続属性に基づいて新しい接続を許可するカスタムロジックを実行できます。
現在、サポートされているクライアント接続ハンドラーのタイプは AWS Lambda 関数のみです。
クライアント VPN エンドポイントのクライアント接続ハンドラを設定するには、デバイス、ユーザー、接続属性を入力として受け取り、
新しい接続を許可または拒否する決定をクライアント VPN サービスに返す AWS Lambda 関数を作成します。
```

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/Untitled2.png)

プロトコルはUDPとし、スプリットトンネルを有効にします。 

スプリットトンネルとはインターネットをアクセスする際にVPNトンネルを経由せずに直接インターネットにアクセスします。 

最後に接続するVPCとクライアントから接続するために許可されたセキュリティグループを選択して残りはデフォルトのままとして、 

VPCエンドポイントを作成します。また今回ユーザーベース認証にしているのでMSADのDNSを指定します。

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/Untitled3.png)

# 作成したVPNエンドポイントにサブネットを関連付け

---

今のままでは接続できないので、サブネットをVPCエンドポイントに紐付けします。

[AWS Client VPN ターゲットネットワーク](https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/cvpn-working-target.html#cvpn-working-target-associate)

作成したVPCエンドポイントを選択し、関連付けタブより関連付けをクリックして、 関連付けしたいVPCとサブネットを選択して、

関連付けをクリックします。 関連付けした直後は黄色マークで関連受け中と表示され、少し待つと関連付け済みになります。

注意が必要なのは関連付けされているサブネット数が増えると費用も増えます。

[AWS VPN の料金](https://aws.amazon.com/jp/vpn/pricing/)

# 作成したVPNエンドポイントに承認ルールを追加

---

作成したVPCエンドポイントを選択し、承認ルールタブより**認証ルールを追加**をクリックします。 

VPNクライアントからアクセスさせるIPアドレスの範囲をアクセスを有効にする送信先ネットに記載し、 

アクセスを付与する対象を全てのユーザーにアクセスを許可するにします。 

AD認証をする場合にはADがいるネットワークの指定をするのを忘れないようにしてください。

AD認証にて特定のアクセスグループのユーザーへのアクセスに絞ることもできます。

[認証ルールを に追加する AWS Client VPN エンドポイント](https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/cvpn-working-rule-authorize-add.html)


# 作成したVPNエンドポイントに接続元制限

---

接続元制限を行いたい場合には作成したVPCエンドポイントを選択し、 セキュリティグループを選択し、

セキュリティグループの適用をクリックします。 セキュリティグループについては事前に作成をするようにしてください。

※セキュリティグループでインバウンドを絞ることはできません。ソースをどのように設定した場合でもClient VPN エンドポイントは通信を受け付けます。

# クライアント側の設定

---

クライアントソフトウェアはAWS公式のClientVPNを利用しています。

[Client VPN Download](https://aws.amazon.com/jp/vpn/client-vpn-download/)

インストールできたら、AWS Console から作成したVPNエンドポイントを選択して、クライアント設定のダウンロードをします。 

また証明書による相互認証を行うためにクライアントの証明書とキーファイルもダウンロードします。 

ダウンロードしたら.ovpnの拡張子ファイルをテキスト開き以下を修正します。 

[AWS Client VPN エンドポイント設定ファイルのエクスポート](https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/cvpn-working-endpoint-export.html)

クライアント証明書とキーファイルの情報を末尾に記載します。

パスの場合）

```bash
cert /path/*********.crt
key /path/*********.key
```

直接クライアント証明書とキーを記載する場合）

```bash
<cert>
-----BEGIN CERTIFICATE-----
MIIDVjCCAj6gAwIBAgIRANCKzFQMReHHibib8hvEI78wDQYJKoZIhvcNAQELBQAw
~ 省略 ~
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCqbZ3LPi+Wr0pB
~ 省略 ~
-----END PRIVATE KEY-----
</key>
```

修正が完了したらOpenVPNを立ち上げてプロファイルをインポートし、接続が成功すれば問題ありません。

また今回指定したActiveDirectoryは二要素認証もいれてあるので、ADの認証/証明書認証/多要素認証の3段構えになっております。

そのため接続する時には以下のような画面になります。

![](/AWS/AWS-Client-VPNを使ってリモートアクセス-AD-多要素認証を使う/image02.png)

接続できた後には、念の為AWS内にあるEC2などへPINGなど実施して疎通確認を行なってください。

# 最後に

---

今回オレオレ証明書をしていますが証明書の期限は必ずやってくるので更新を忘れずにしましょう。 

更新期限を100年にするなどのやり方もあります。

認証局の設定ファイルを見るとサーバ証明書はデフォルトで825日となっているようです。 

同じくCAの有効期限期限は3650日となっているようです。

```bash
# grep default_days "/home/ec2-user/easy-rsa/easyrsa3/openssl-easyrsa.cnf"
default_days= $ENV::EASYRSA_CERT_EXPIRE# how long to certify for

# grep EASYRSA_CERT_EXPIRE "/home/ec2-user/easy-rsa/easyrsa3/vars.example"
#set_var EASYRSA_CERT_EXPIRE  825

# grep EASYRSA_CA_EXPIRE "/home/ec2-user/easy-rsa/easyrsa3/vars.example"
#set_var EASYRSA_CA_EXPIRE  3650
```

上記を任意の日数に変更して./easyrsa init-pkiを実施することで初期化することができます。 

事前に有効日数を指定したい場合にはこちらを実施した上で進めてください。 

期限を延ばすことで気をつけなきゃいけないのは使わないクライアント証明書は失効させるようにしましょう。 

あとここまでやって最後調べていて気づいたのですが、AWSのハンズオン内容があるんですね。。。 

[AWS Client VPN Basic ハンズオン](https://catalog.us-east-1.prod.workshops.aws/workshops/be2b90c2-06a1-4ae6-84b3-c705049d2b6f/ja-JP)

参考としてACM証明書を利用したやり方も見つけたのでクラスメソッド様のBLOG内容を引用させていただきます。

[ACM で発行したパブリック証明書で Client VPN を構築してみた](https://dev.classmethod.jp/articles/clientvpn-with-acm-public-certificates/)

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

