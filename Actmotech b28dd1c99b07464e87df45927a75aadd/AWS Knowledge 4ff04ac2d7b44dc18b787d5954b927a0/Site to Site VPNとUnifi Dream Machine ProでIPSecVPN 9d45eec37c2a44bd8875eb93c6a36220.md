# Site to Site VPNとUnifi Dream Machine ProでIPSecVPN

![20200821131638.png](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/20200821131638.png)

# 目次

---

# はじめに

---

ハイブリットクラウドの検討においてIPSecVPNでオンプレミスとAWSを接続しました。 

Unifi Dream Machine Pro を利用しています。

実施するにあたっての前提としてオンプレ側の外部アドレスは固定IPであること、 AWS側のVPCはサブネットやIGWの設定が完了しているものとします。

# 実施手順概要

---

1. 仮想プライベートゲートウェイの作成
2. カスタマーゲートウェイの作成
3. サイト間のVPN接続の設定
4. Unifi Dream Machine Proの設定
5. VPCのルートテーブルの設定

の順で実施していきます。

 料金は以下のような感じです。 

- https://aws.amazon.com/jp/vpn/pricing/

# 仮想プライベートゲートウェイの作成

---

AWS ConsoleのVPCから仮想プライベートゲートウェイを選択して、仮想プライベートゲートウェイの作成をクリックします。 

名前は任意としてASNはAmazonデフォルトASNにして、仮想プライベートゲートウェイを作成します。

![20220413012543.png](Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40/20220413012543.png)

作成ができたら作成した仮想プライベートゲートウェイを選択し、右上のアクションからVPCへのアタッチを実施します。

![20220413013016.png](Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40/20220413013016.png)

# カスタマーゲートウェイの作成

---

AWS ConsoleのVPCからカスタマーゲートウェイを選択して、カスタマーゲートウェイの作成をクリックします。 

名前は任意、BGPのASNはデフォルト、IPアドレスはオンプレミス側のGIPを指定します。 

ARN 証明書はIKE 認証のために事前に共有されたキーではなくデジタル証明書を使用する際に利用します。

https://aws.amazon.com/jp/premiumsupport/knowledge-center/vpn-certificate-based-site-to-site/

今回はIKE認証で実施するので何も選択しません。 デバイスも今回は未入力とし、カスタマーゲートウェイを作成します。

![20220413013116.png](Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40/20220413013116.png)

# サイト間のVPN接続の設定

---

AWS ConsoleのVPCからサイト間のVPN接続を選択して、VPN接続を作成するをクリックします。 

名前タブには任意の名前を入れ、仮想プライベートゲートウェイとカスタマーゲートウェイIDは先ほど作成したものを選択し、

 ルーティングオプションは静的を選択します。

![20220413014024.png](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/20220413014024.png)

トンネルオプションはデフォルトのままでVPN接続の作成をクリックします。

![20220413014044.png](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/20220413014044.png)

トンネルオプションで選択できるものを一応貼っておきます。

![20220413014424.png](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/20220413014424.png)

![20220413014447.png](Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40/20220413014447.png)

作成できたら作成したVPN接続を選択して設定のダウンロードを選択します。

ベンダーは色々選べます（CISCO、Pfsense、Yamaha、Fortinet、F5、Juniperなど）

今回接続する対象はFortigateなので以下のような形で設定情報をダウンロードしています。

設定ファイルをダウンロードするとトンネルIPや事前共有鍵の記載があるので確認しておきましょう。

![Untitled](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/Untitled.png)

# Unifi Dream Machine Proの設定

---

Unifi Dream Machine Pro のUIへログインし、

Network>Teleport&VPNの画面にてSite-to-Site VPNからCreate Site-to-site VPNをクリックします。

1. VPN Protocolは、Manual IPsecを選択
2. Pre-shared Keyにはダウンロードしたvpn-xxxxxxxxxxxxxxxxxx.txtというファイルの36行目にある 、
IPSec Tunnel #1 の Pre-Shared Key に記載されている32文字を入力
3. UniFi Gateway IP は、使用するWANを選択
4. Shared Remote Subnet にVPCのプライベートIPとサブネットを入力
5. Remote IP に、vpn-xxxxxxxxxxxxxxxxxx.txtの94行目にある IPSec Tunnel #1 の 
Virtual Private Gateway に記載されているグローバルIPアドレスを入力
6. AdvanedをManualに変更
    - Key Exchange Version でIKEv2を選択
    - EncrytpionでAES-256を選択
    - HashでSHA256を選択
    - IKE DH Group で24を選択
    - ESP DH Group で18を選択

![Untitled](Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220/Untitled%201.png)

最後にAdd New VPN Network ボタンをクリックします。

なお、AWS側では2個のトンネルを用意されていますが、UDM-Proが同一宛先のものは1個ずつしか有効にすることができないため、

もう片側を設定を入れても有効化することができませんので注意が必要です。

# VPCのルートテーブルの設定

---

AWS ConsoleのVPCから接続したVPCのルートテーブルを選択し、アクションからルート編集をクリックします。 

ルートの追加をクリックして、Unifi Dream Machine ProのLAN(internal)側のネットワークアドレスを入力し、

ターゲットを作成した仮想プライベートゲートウェイを指定します。 最後にルートの保存をクリックします。

![20220413025401.png](Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40/20220413025401.png)

# IPSecVPNステータスをチェック

---

AWS ConsoleのVPCからサイト間のVPN接続を選択して、作成したVPNを選択します。 

タブの中にトンネルの詳細があるのでクリックするとステータスがUPになっています。 

# 最後に

---

自宅にUnifi Dream Machine Proがある人はあまりいないように思いますが、これを自宅でやろうとしている人が誤家庭ですね。