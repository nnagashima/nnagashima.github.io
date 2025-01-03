[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ ETC Knowledgeへ戻る](/ETC/top)

# NURO光回線の速度改善

# 目次
- [NURO光回線の速度改善](#nuro光回線の速度改善)
- [目次](#目次)
- [はじめに](#はじめに)
- [調べたこと](#調べたこと)
- [購入したもの](#購入したもの)
- [セットアップ](#セットアップ)
- [肝心のインターネット速度テストはどうなったか？](#肝心のインターネット速度テストはどうなったか)
- [最後に](#最後に)

<br>

# はじめに

以前やったNURO光の速度改善を実施した話でも書こうと思います。

 ※環境によっては必ずしも回線スピードが改善するとは言えないので、ご了承ください。

ここ最近NURO光でインターネット速度テストを行っていたところ、 

回線開通後と比べて100Mbpsほど遅くなっていて頑張ってもダウンロード速度が、

 350-400Mbpsしかでていないことに気づき、なんとかならないか検討を開始

# 調べたこと

そもそもNURO光のONUはルーター兼の機能があり、ルーター機能のみをOFFにすることはできないです。

 そのために2重ルーター構造になってしまいます。

インターネット接続に不具合が出る可能性やサポートしていない構成になるので理解した上で実施をしてください。

※公式ページにも記載ありました。

なので実施するための方法として考えたのはNURO光のONU兼ルーターのDMZ機能を有効化し、

 新しく購入するルーターにパケットを転送するようにすればできるのではないか？と考えて調べたところ、

 すでにやっている方の記事を見かけました。

# 購入したもの

速度改善だけでなく購入にあたり以下の要件を考えました。 

- 子供がインターネット使うこと時にセキュリティをしっかりしておきたい
- WiFi6(11ax)に対応していること
- 価格は1万円前後

購入したもの： [ASUS WiFi 無線 ルーター WiFi6 2402+574Mbps v6プラス対応デュアルバンド RT-AX3000](https://www.amazon.co.jp/gp/product/B084K1NY5D/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)

RT-AX3000を選んだ理由としては、 

- トレンドマイクロのAiProtectionを利用している
- WiFi6(11ax)に対応している
- 1万円ちょっとで購入できる
- PPTP、OpenVPN、IPSecVPNに対応
- デュアルWANに対応 です。

# セットアップ

まず購入したRT-AX3000のWANポートととNURO光のONUのLANポートを結線します。 

その後、RT-AX3000マニュアルを見ながら初期セットアップをします。 

RT-AX3000のセットアップ完了したら、NURO光のONUの設定変更します。 

- DMZ機能を有効しRT-AX3000のWANポートに割り当てしたIPアドレスを指定
- 無線LAN機能をOFFにする

最後にRT-AX3000の配下にあるPCでインターネット接続テストを実施して問題なければ完了です。

# 肝心のインターネット速度テストはどうなったか？

以前までは平均してダウンロード速度が350-400Mpbsだったものが、 平均して600Mbpsぐらいまででるようになりました！！ 

※調子がいいと700Mbpsまででるー

# 最後に

速度が速くなったものの、そもそも公式ではサポートしていない構成になっているので、 実施する分には自己責任でお願いします。 

また初めに書きましたが必ずしも速度が改善するとは限らならいので注意してください。

なお、今回別ルーターを設置したことにより管理が複雑にはなるけれども 色々なことができるようになったので、

これはこれでよかったなと思います。

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ ETC Knowledgeへ戻る](/ETC/top)