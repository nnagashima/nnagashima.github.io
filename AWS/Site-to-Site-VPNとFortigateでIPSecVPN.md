[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# Site to Site VPNとFortigateでIPSecVPN

# 目次
- [Site to Site VPNとFortigateでIPSecVPN](#site-to-site-vpnとfortigateでipsecvpn)
- [目次](#目次)
- [はじめに](#はじめに)
- [実施手順概要](#実施手順概要)
- [仮想プライベートゲートウェイの作成](#仮想プライベートゲートウェイの作成)
- [カスタマーゲートウェイの作成](#カスタマーゲートウェイの作成)
- [サイト間のVPN接続の設定](#サイト間のvpn接続の設定)
- [Fortigateの設定](#fortigateの設定)
- [VPCのルートテーブルの設定](#vpcのルートテーブルの設定)
- [IPSecVPNステータスをチェック](#ipsecvpnステータスをチェック)

---

# はじめに

---

ハイブリットクラウドの検討においてIPSecVPNでオンプレミスとAWSを接続しました。 

Fortigateは60D を利用しており、ファームウェアはv6.0.11 build0387 (GA)です。

実施するにあたっての前提としてオンプレ側の外部アドレスは固定IPであること、 AWS側のVPCはサブネットやIGWの設定が完了しているものとします。

# 実施手順概要

---

1. 仮想プライベートゲートウェイの作成
2. カスタマーゲートウェイの作成
3. サイト間のVPN接続の設定
4. Fortigateの設定
5. VPCのルートテーブルの設定

の順で実施していきます。

料金は以下のような感じです。 

[AWS サイト間 VPN および Accelerated サイト間 VPN への接続料金](https://aws.amazon.com/jp/vpn/pricing/)


# 仮想プライベートゲートウェイの作成

---

AWS ConsoleのVPCから仮想プライベートゲートウェイを選択して、仮想プライベートゲートウェイの作成をクリックします。 

名前は任意としてASNはAmazonデフォルトASNにして、仮想プライベートゲートウェイを作成します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413012543.jpg)

作成ができたら作成した仮想プライベートゲートウェイを選択し、右上のアクションからVPCへのアタッチを実施します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413013016.jpg)

# カスタマーゲートウェイの作成

---

AWS ConsoleのVPCからカスタマーゲートウェイを選択して、カスタマーゲートウェイの作成をクリックします。 

名前は任意、BGPのASNはデフォルト、IPアドレスはオンプレミス側のGIPを指定します。 

ARN 証明書はIKE 認証のために事前に共有されたキーではなくデジタル証明書を使用する際に利用します。

![AWS Site-to-Site VPN を使用して証明書ベースの VPN を作成する方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/vpn-certificate-based-site-to-site/)

今回はIKE認証で実施するので何も選択しません。 デバイスも今回は未入力とし、カスタマーゲートウェイを作成します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413013116.jpg)

# サイト間のVPN接続の設定

---

AWS ConsoleのVPCからサイト間のVPN接続を選択して、VPN接続を作成するをクリックします。 

名前タブには任意の名前を入れ、仮想プライベートゲートウェイとカスタマーゲートウェイIDは先ほど作成したものを選択し、

ルーティングオプションは静的を選択します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413014024.jpg)

トンネルオプションはデフォルトのままでVPN接続の作成をクリックします。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413014044.jpg)

トンネルオプションで選択できるものを一応貼っておきます。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413014424.jpg)

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413014447.jpg)

作成できたら作成したVPN接続を選択して設定のダウンロードを選択します。

ベンダーは色々選べます（CISCO、Pfsense、Yamaha、Fortinet、F5、Juniperなど）

今回接続する対象はFortigateなので以下のような形で設定情報をダウンロードしています。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413014823.jpg)

設定ファイルをダウンロードするとトンネルIPや事前共有鍵の記載があるので確認しておきましょう。

# Fortigateの設定

---

AWS側は接続元のトンネルを２つ用意してくれているので２つ設定することを忘れないようにしてください。 

またルーティング設定でプライオリティの設定をどちらかは下げるようにしてください。

FortigateにログインしてメニューのVPNからIPSecウィーザードを選択します。 

任意の名前を入れ、テンプレートタイプはカスタムを選択します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413020106.jpg)

ネットワークの欄ではIPアドレスのところに先ほど設定ダウンロードしたAWS側のTunnelIPを記載します。
インターフェースにはWANを選択します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413020037.jpg)

認証方式は事前共有鍵を選択し、事前共有鍵に先ほど設定ダウンロードした記載のPre-Shared Keyを入力します。

 残りはダウンロードした設定ファイルに従ってFortigate側の設定を行います。 

暗号化強度は最低要件を満たして実施しているので強度を上げる場合には適時暗号化強度を変更してください。 

尚、Ciscoの設定をダウンロードしたら以下のように記載がありました。

```bash
! A policy is established for the supported ISAKMP encryption,
! authentication, Diffie-Hellman, lifetime, and key parameters.
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! Category "VPN" connections in the GovCloud region have a minimum requirement of AES128, SHA2, and DH Group 14.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! NOTE: If you customized tunnel options when creating or modifying your VPN connection, you may need to modify these sample configurations to match the custom settings for your tunnels.
```

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413021452.jpg)

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413022934.jpg)

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413023206.jpg)

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413023238.jpg)

次にルーティング設定を行います。

ネットワークからスタティクルートを選択し、新規作成をクリックします。

宛先にはVPCのCIDRを入力し、インターフェースには先ほど作ったVPNをプルダウンから選択します。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413023626.jpg)

次にAWSとの通信許可をポリシー&オブジェクトのIPv4ポリシーから新規作成をクリックします。

名前は任意で入力します。

入力インターフェースはLAN側のインターフェースを選択し、出力インターフェースは先ほど作ったVPNをプルダウンから選択します。

 送信元や宛先、サービスなどは環境ごとに適時設定してください。今回はALLで設定しています。

 画像は省略しますが逆側の通信も同様に作るようにしてポリシーが両方向で設定できたら完了です。

 通信ができていない間は警告の黄色▲マークが表示されますがIPSec接続できるようになれば消えます。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413023940.jpg)

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413024730.jpg)

ここまででFortigateの設定は完了です。

# VPCのルートテーブルの設定

---

AWS ConsoleのVPCから接続したVPCのルートテーブルを選択し、アクションからルート編集をクリックします。 

ルートの追加をクリックして、FortigateのLAN(internal)側のネットワークアドレスを入力し、

ターゲットを作成した仮想プライベートゲートウェイを指定します。 最後にルートの保存をクリックします。

![](/AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413025401.jpg)

# IPSecVPNステータスをチェック

---

AWS ConsoleのVPCからサイト間のVPN接続を選択して、作成したVPNを選択します。 

タブの中にトンネルの詳細があるのでクリックするとステータスがUPになっています。 

もしUPなっていない場合にはFortigate側のモニタからIPSecモニタを選択し、対象のVPN接続を選択しアップをクリックしてください。

![](AWS/Site-to-Site-VPNとFortigateでIPSecVPN/20220413025644.jpg)

以上でFortigateとAWS VPCとのVPN接続設定が完了です。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)