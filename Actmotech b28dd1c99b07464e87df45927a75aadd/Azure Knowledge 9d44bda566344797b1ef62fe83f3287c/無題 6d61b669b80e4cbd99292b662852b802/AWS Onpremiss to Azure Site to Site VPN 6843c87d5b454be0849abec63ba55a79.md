# AWS/Onpremiss to Azure Site to Site VPN

![ダウンロード.png](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/%25E3%2583%2580%25E3%2582%25A6%25E3%2583%25B3%25E3%2583%25AD%25E3%2583%25BC%25E3%2583%2589.png)

# 目次

---

# はじめに

---

ハイブリットクラウドの検討においてIPSecVPNでAzureとAWS / Onpremissを接続しました。 

OnpremissにはUnifi Dream Machine Pro を利用しています。

# 前提

---

- AWSの仮想ネットワークは作成済み
- Azureの仮想ネットワークは作成済み

# 参考にした情報

https://learn.microsoft.com/ja-jp/azure/vpn-gateway/tutorial-site-to-site-portal

# Azure側で仮想ネットワークでゲートウェイサブネット作成

---

サブネットからゲートウェイサブネット作成します。

サブネットアドレス範囲：ゲートウェイサブネット用に作成します。

![Untitled](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/Untitled.png)

# Azure 側で仮想ネットワークゲートウェイ作成

---

仮想ネットワークゲートウェイを作成します。

- **[名前]:** 好きな名前で作成します。
- **[リージョン]:** RGがあるリージョンを選択します。
- **ゲートウェイの種類:** VPN
- **VPN の種類:** ルート ベース
- **SKU:** VpnGw2
- **世代:** 第 2 世代
- **仮想ネットワーク:** 先ほど作成したゲートウェイサブネットが存在する仮想ネットワークを選びます。
- **ゲートウェイ サブネットのアドレス範囲:** 仮想ネットワークを選択するとゲートウェイサブネットが表示されます。
- **[パブリック IP アドレス]** : 新規作成
- **パブリック IP アドレス名:** 好きな名前で作成します。
- **アクティブ/アクティブ モードの有効化:** 有効

→有効にすることでアクティブ/アクティブ モードでのVPNゲートウェイを作成することができます。

　※ゾーン冗長はゾーン指定でもいいですが今回はゾーン冗長で実施しました。（料金は変わらないはず）

- **[Configure BGP](BGP の構成):** 無効

参考情報

SKUについて：https://learn.microsoft.com/ja-jp/azure/vpn-gateway/vpn-gateway-about-vpngateways

VNGの料金について：https://azure.microsoft.com/ja-jp/pricing/details/vpn-gateway/

※SKUのサイズによってデプロイされる時間が結構変わるようです。

![Untitled.png](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/Untitled%201.png)

# AWS側の設定

---

カスタマーゲートウェイ作成（Azureの仮想ネットワークゲートウェイのGIPを入力）

仮想プライベートゲートウェイはすでにあるものを利用、サブネットのルーティング設定を追記

VPN接続設定作成

[Site to Site VPNとFortigateでIPSecVPN](../../AWS%20Knowledge%204ff04ac2d7b44dc18b787d5954b927a0/Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40.md) のAWS側の設定を参照

# オンプレミス側の設定

---

VPN接続設定作成

[Site to Site VPNとUnifi Dream Machine ProでIPSecVPN ](../../AWS%20Knowledge%204ff04ac2d7b44dc18b787d5954b927a0/Site%20to%20Site%20VPN%E3%81%A8Unifi%20Dream%20Machine%20Pro%E3%81%A6%E3%82%99IPSecVPN%209d45eec37c2a44bd8875eb93c6a36220.md) のUnifi Dream Machine Proの設定を参照

# Azure側で ローカルネットワークゲートウェイ作成

---

ローカルネットワークゲートウェイを作成します。

接続したい先の数の分だけ作成します。

- IPアドレス：接続先のIPアドレスを記載します。
- アドレス空間：接続先のLocalIPを記載します。
    
    ![名称未設定.png](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/%25E5%2590%258D%25E7%25A7%25B0%25E6%259C%25AA%25E8%25A8%25AD%25E5%25AE%259A.png)
    

デフォルトのままとして作成をします。

![名称未設定.png](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/%25E5%2590%258D%25E7%25A7%25B0%25E6%259C%25AA%25E8%25A8%25AD%25E5%25AE%259A%201.png)

# VPN接続を作成

---

作成した仮想ネットワークゲートウェイとローカルネットワークゲートウェイを使ってVPN接続を作成します。

作成した仮想ネットワークゲートウェイに移動し、左メニューの接続をクリックして接続を追加します。

接続したい先の数の分だけ作成します。

- 名前：作成する名前
- 接続の種類：サイト対サイト
- 仮想ネットワークゲートウェイ：作成した仮想ネットワークゲートウェイ
- ローカルネットワークゲートウェイ：作成したローカルネットワークゲートウェイ
- 共有キー：入れる値はなんでもいいですが、接続先と一致させる必要があります。

→AWSであれば共有キーが払い出されるので、そちらを利用。オンプレミスだとランダムでキーを作る必要があります。

- Azure プライベート IP アドレスを使用する]：オフのままにします。
- BGP を有効にする：オフのままにします。
- IKEプロトコル：IKEv2 を選択します。
    
    ![名称未設定.png](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/%25E5%2590%258D%25E7%25A7%25B0%25E6%259C%25AA%25E8%25A8%25AD%25E5%25AE%259A%202.png)
    

なお、VPN接続作成後にIPSecおよびIKEポリシーを変更することができます。

変更する際にはVPN接続作成した対象を選択すると左メニューに構成があるのでクリックすることでIPSecおよびIKEポリシーを変更することができます。

https://learn.microsoft.com/ja-jp/azure/vpn-gateway/vpn-gateway-about-compliance-crypto

参考までに設定した内容を載せておきます。

![Untitled](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/Untitled%202.png)

# VPN接続確認

---

Azureで起動しているVMから、AWSやOnPremissで動いているVMに対してPingを実行して応答があれば接続ができています。

他の確認方法としては仮想ネットワークゲートウェイの左メニューの接続から状態の確認をすることができます。

![Untitled](AWS%20Onpremiss%20to%20Azure%20Site%20to%20Site%20VPN%206843c87d5b454be0849abec63ba55a79/Untitled%203.png)

# 最後に

---

IPSecをやるたびにIPSecおよびIKEポリシーを忘れるので、良い復習になりました。