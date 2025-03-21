[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# WorkSpacesへのアクセスを信頼されたデバイスから接続

# 目次

- [WorkSpacesへのアクセスを信頼されたデバイスから接続](#workspacesへのアクセスを信頼されたデバイスから接続)
- [目次](#目次)
- [はじめに](#はじめに)
- [デバイス証明書の作成](#デバイス証明書の作成)
- [CAの公開鍵(cacert.pem)をインポート](#caの公開鍵cacertpemをインポート)
- [WorkSpces接続端末へクライアント証明書のインストール](#workspces接続端末へクライアント証明書のインストール)

---

# はじめに

---

WorkSpaces接続する際にAD連携や多要素認証によってセキュリティ向上することができますが、

デバイス証明書を用いて信頼されたデバイスからのみしか接続できなくする方法もありますので記載していきたいと思います。

# デバイス証明書の作成

---

OSはAmazonLinux2023を使って作ります。OpenSSLがインストールされていることを確認します。

```jsx
# openssl version
OpenSSL 3.0.8 7 Feb 2023 (Library: OpenSSL 3.0.8 7 Feb 2023)
```

インストールされていない場合にはインストールを実施してください。

```jsx
# yum install -y openssl
```

作業用ディレクトリの作成をします。

```jsx
# mkdir -p /etc/pki/home-workspaces
# mkdir -p /etc/pki/home-workspaces/client/2024
# mkdir -p /etc/pki/home-workspaces/newcerts
```

CAを構成します。

```jsx
# cd /etc/pki/home-workspaces/
# echo "01" | sudo tee serial
# touch index.txt
# cp /etc/pki/tls/openssl.cnf ./openssl_ca.cnf
# vi openssl_ca.cnf
[ CA_default ]
dir = /etc/pki/home-workspaces
 
[ v3_ca ]
# authorityKeyIdentifier=keyid:always,issuer
keyUsage = digitalSignature, cRLSign, keyCertSign
```

CAの公開鍵と秘密鍵を作成します。(100年で作成します。）

```jsx
# openssl req -new -x509 -keyout cakey.pem -out cacert.pem -days 35600 -config openssl_ca.cnf
Enter PEM pass phrase: パスワード入力
Verifying - Enter PEM pass phrase:　パスワード入力
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:都道府県
Locality Name (eg, city) [Default City]:市町村区
Organization Name (eg, company) [Default Company Ltd]:Actmotech
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:home-workspaces
Email Address []:
```

クライアントの公開鍵と秘密鍵を作成します。

```jsx
# cd client/2024
# openssl genrsa -out client.key 2048
# openssl req -new -key client.key -out client.csr -sha256 -days 365
Ignoring -days without -x509; not generating a certificate
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:都道府県
Locality Name (eg, city) [Default City]:市町村区
Organization Name (eg, company) [Default Company Ltd]:Actmotech
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:home-workspaces
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:任意のパスワード
An optional company name []:任意のパスワード
```

OpenSSLの設定ファイルをコピーして編集します。

```jsx
# cp /etc/pki/tls/openssl.cnf ./openssl_client.cnf
# vi openssl_client.cnf
[ CA_default ]
dir = /etc/pki/home-workspaces
private_key = $dir/cakey.pem

[ usr_cert ]
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
extendedKeyUsage = clientAuth
```

作成したクライアント証明書をCAに署名してもらいます。

```jsx
# openssl ca -config openssl_client.cnf -out client.crt -in client.csr -days 365
Using configuration from openssl_client.cnf
Enter pass phrase for /etc/pki/home-workspaces/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: May 16 12:57:39 2024 GMT
            Not After : May 16 12:57:39 2025 GMT
        Subject:
            countryName               = JP
            stateOrProvinceName       = Saitama
            organizationName          = Actmotech
            commonName                = home-workspaces
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Key Usage: critical
                Digital Signature, Non Repudiation, Key Encipherment, Data Encipherment
            X509v3 Subject Key Identifier: 
                B5:25:82:22:93:6E:AD:FD:90:82:A1:DC:AA:F1:4F:AC:62:8F:E9:A5
            X509v3 Authority Key Identifier: 
                B2:8F:1F:96:44:39:34:83:FB:9E:7A:83:AC:7E:2D:D7:42:39:89:A0
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
Certificate is to be certified until May 16 12:57:39 2025 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

クライアントにインストールするPKCS12のファイルを作成します。

```jsx
# openssl pkcs12 -export -in client.crt -inkey client.key -name home-workspaces -certfile /etc/pki/home-workspaces/cacert.pem -out home-workspaces.p12
Enter Export Password:
Verifying - Enter Export Password:
```

# CAの公開鍵(cacert.pem)をインポート

---

WorkSpacesサービスからディレクトリを選択してアクセスコントロールオプションを編集します。

ルート証明書1にcacert.pemの内容を貼り付けます。

私はMAC/Windowsユーザーなので信頼されたデバイスをMacOS/Windowsで選択します。

![](/AWS/WorkSpacesへのアクセスを信頼されたデバイスから接続/Untitled.png)

# WorkSpces接続端末へクライアント証明書のインストール

---

home-workspaces.p12をダブルクリックします。

パスワードを求められるので入力をしてデバイス証明書を端末にインストールします。

その後WorkSpacesアプリを開いて、ログイン画面が表示がされWorkSpacesにログインができればOKです。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
