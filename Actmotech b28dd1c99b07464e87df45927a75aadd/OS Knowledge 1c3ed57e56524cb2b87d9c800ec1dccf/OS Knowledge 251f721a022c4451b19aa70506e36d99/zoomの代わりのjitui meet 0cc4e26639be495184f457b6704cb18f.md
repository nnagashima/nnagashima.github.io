# zoomの代わりのjitui meet

タグ: Ubuntu

![ubuntu_14143.png](Guacamole%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E7%92%B0%E5%A2%83%E3%82%92%E6%95%B4%E3%81%88%E3%82%8B%2006607b0254a146f2926a6087ee4ab548/ubuntu_14143.png)

# Zoomの代わりに検討

---

ZOOMもいいんですけどライセンス料金などがあるので、無償の代わりのものないかを探していたところ、

jitui-meetというものを見つけましたので、構築して利用してみました。

# 前提

---

- OSはUbuntu22.04
- DNSはCloudflareで管理
- 443と80ポートがLetsEncyptにてアクセスできる

ような状態からスタートします。

# Jitui Meetインストールする前のセットアップ

---

OSを最新化して、必要なPKGをインストールします。

なお、JitsiはUbuntuのuniverseパッケージリポジトリからの依存関係を必要としますのでリポジトリを追加します。

```bash
# apt update
# apt install apt-transport-https
# apt-add-repository universe
# apt update
```

DNSレコードでセットしたDomain Nameでホスト名を変更します。

```bash
# hostnamectl set-hostname ドメイン名
```

次にホスト名とドメイン名を紐付けします。

```bash
# cat /etc/hosts
GlobalIP ドメイン名
```

Prosodyパッケージのリポジトリを追加します。

※Prosodyは近代的なXMPP通信サーバー

```bash
# echo deb [http://packages.prosody.im/debian](http://packages.prosody.im/debian) $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
# wget [https://prosody.im/files/prosody-debian-packages.key](https://prosody.im/files/prosody-debian-packages.key) -O- | sudo apt-key add 
```

jitsiリポジトリがパッケージソースに追加され、JitsiMeetパッケージが利用可能になります。

```bash
# curl [https://download.jitsi.org/jitsi-key.gpg.key](https://download.jitsi.org/jitsi-key.gpg.key) | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
# echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] [https://download.jitsi.org](https://download.jitsi.org/) stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
# apt update
```

Jitui Meetをインストールします。

以下コマンドを実行すると、ドメインを入力する画面とLet's Encryptで証明書取得までできます。

```bash
# apt install jitsi-meet
```

ドメインでアクセスをすると、jitui meetのUIが表示されますが今のままだと誰でもMTGが開催できる状態となるため制限をかけます。

```bash
# cd /etc/prosody/conf.avail/
# vi meet.actmotech.xyz.cfg.lua
authentication = "internal_hashed" -- do not delete me
→anonymousをinternal_hashedに変更
```

また以下コマンドでユーザーとパスワードを作成します。

```bash
prosodyctl register [username] [domain名] [password]
```

上記の設定変更が終わったら、サービスの再起動を行います。

```bash
# systemctl restart jicofo.service prosody.service nginx.service
```

再度ドメインでアクセスしてMTGを開始するとユーザー名とパスワードを求められるので、

登録したユーザ名でログインします。

![Untitled](zoom%E3%81%AE%E4%BB%A3%E3%82%8F%E3%82%8A%E3%81%AEjitui%20meet%200cc4e26639be495184f457b6704cb18f/Untitled.png)