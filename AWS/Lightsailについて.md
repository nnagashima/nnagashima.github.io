[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# Lightsailについて

# 目次
- [Lightsailについて](#lightsailについて)
- [目次](#目次)
- [はじめに](#はじめに)
- [Lightsailとは](#lightsailとは)
- [Lightsailのインスタンスについて](#lightsailのインスタンスについて)
- [Lightsailのコンテナについて](#lightsailのコンテナについて)
- [Lightsailのデータベースについて](#lightsailのデータベースについて)
- [Lightsailのロードバランサーについて](#lightsailのロードバランサーについて)
- [LightsailのDNSについて](#lightsailのdnsについて)
- [Lightsailのストレージについて](#lightsailのストレージについて)
- [Lightsail CDNとは](#lightsail-cdnとは)

---

# はじめに

---

LightsailとAWSの各サービスと比較してみました。
(2022/08/01現在)

# Lightsailとは

---

- インスタンス/コンテナ/データベース/ストレージサービス/ロードバランサー/DNS/CDNを提供
- 料金が基本月額固定のため予想可能な低価格の料金を支払うだけ*
- スタートアップスクリプトが用意されているのですぐに開始できる
- LightsailサービスとAWS主要サービスをVPC Peeringを用いて連携することができる
- スモールスタートをAWS主要サービスからではなくLightsailから始めるのはあり
- SLA保証はなし


https://aws.amazon.com/jp/lightsail/faq/

https://aws.amazon.com/jp/lightsail/pricing/

# Lightsailのインスタンスについて

---

- インスタンスはLinuxとWindowsの両方を提供
- 月額固定でVMを利用することが可能(停止中も課金対象)
- 料金の中には転送料金も含まれている(Overした分は料金加算、東京リージョンの場合は0.14 USD/GB)
- スナップショット機能を提供しており、取得したスナップショットからEC2へアップグレード可能
- 静的GIPアドレスをLightsailインスタンスに紐づけることが可能（Redigon内で5つまで無料）
- CPUメトリクスの監視とアラーム設定が可能であり、それ以外は監視不可
- [Lightsail FAQ](https://aws.amazon.com/jp/premiumsupport/knowledge-center/lightsail-considerations-for-use/)
- [LightsailとEC2の違い](https://aws.amazon.com/jp/premiumsupport/knowledge-center/lightsail-differences-from-ec2/)

LightsailとEC2の比較

| 項目 | Lightsail仮想マシン | EC2 |
| --- | --- | --- |
| 複数IP | 複数IPアドレスをアタッチすることが不可能 | 複数IPアドレスをアタッチすることが可能 |
| コンソール出力 | 実践編 | 確認は可能 |
| 終了保護 | 終了保護機能なし | 終了保護機能あり |

# Lightsailのコンテナについて

---

- 手軽にコンテナサービスを始めることができる
- DockerHub or ECRからリポジトリを指定してサービスを起動（パブリックのみ）
- パブリックエンドポイントはHTTPSのみ提供
- コンテナのスケールアウト
- CPUとメモリの監視とアラーム設定が可能

# Lightsailのデータベースについて

---

- MySQL：5.7.38 or 8.0.28のみ
- PostgreSQL:10.20 or 11.15 or 12.10のみ
- 固定の月額料金(+snapshot料金)
- 高可用性オプションあり（別のAZで提供）
- スナップショット機能を提供（スタンバイから作成）
- パブリックかプライベートを選択することができる
- DBパラメータやメンテナンス設定はCLI経由での設定
- 6項目の監視とアラーム設定が可能

LightsailとRDSの比較

| 項目 | Lightsailデータベース | RDS |
| --- | --- | --- |
| 利用料金 | 月額固定 | 利用した分だけ課金 |
| 提供OS | MySQL：5.7.38 or 8.0.28のみ/PostgreSQL:10.20 or 11.15 or 12.10のみ | MySQL / PostgreSQL / MSSQL / MariaDB / Aurora MySQL / Aurora PostgreSQL |
| インスタンスタイプ | 固定で決まっているプラン変更不可 | 色々と選べる、プラン変更可能 |
| サービス連携 | VPC Peeringを用いてAWSサービスと連携可能 | VPC Peeringなど意識することなく利用可能 |
| ネットワーク | パブリック、プライベート環境に配置可能 | パブリック、プライベート環境に配置可能 |
| 監視 | CPU使用率、DB接続、DISKキュー深度、DISK空き容量、ネットワークの送信/受信スループット | CloudWatchにて様々なメトリック監視提供が可能 |
| DBパラメータ | CLI経由でないと更新不可 | UI上で更新可能 |
| メンテナンス | CLI経由で設定が可能 | UI上でスケジューリング設定可能 |
| ログ | LightsailのUI上で閲覧可能 | CloudWatchLogsで閲覧とアラーム設定含めて可能 |
| バックアップ | スナップショット、5分単位で7日分保管 | スナップショット、5分単位でトランザクションログをS3へ保管、最大で35日分を保管 |

# Lightsailのロードバランサーについて

---

- 月額固定料金
- SSL通信も利用可能（証明書の取得も可能）
- 取得したものはRoute53で管理可能、Lightsail DNSでも管理可能
- 仮想マシンは80番ポートのListenのみをヘルスチェック
- 正常なホスト数の監視とアラーム設定が可能
- 機能が足りなければELBを利用することも視野に入れましょう

LightsailロードバランサーとELBの比較

| 項目 | Lightsailロードバランサー | ELB |
| --- | --- | --- |
| 利用料金 | 18USD | Application Load Balancer 時間あたり 0.0243USD、LCU 時間あたり 0.008USD |
| 利用可能ポート | 80 or 443 | 80/443以外も可能 |
| ターゲットポート | 80ポートのみ ※CLI経由でカスタマイズ可能 | 80ポート以外も可能 |
| サービス連携 | VPC Peeringを用いてAWSサービスと連携可能 | VPC Peeringなど意識することなく利用可能 |
| 証明書 | LBサービスの中で取得可能、Route53 or Lightsail DNSで管理可能 | ACMで取得してRoute53で管理し自動更新可能 |
| ロギング | なし | S3バケットにログ保存可能 |
| HelthCheck | HealthCheckPath 設定のみ | HealthCheckPort、HealthCheckProtocol、Matcherなど選択可 |
| 監視 | 正常なホスト数のみ | CloudWatchにて様々なメトリック監視提供が可能 |

# LightsailのDNSについて

---

- DNSゾーンは3つまで作成可能 ※4つ以上作りたい場合はRoute53を利用
- ルーティングポリシーなし
- サポートされないレコードタイプ：CAA、DS、NAPTR、PTR、SOA、SPF
- ヘルスチェック機能なし
- プライベートDNSゾーンサポートなし
- 機能が足りなければRoute53を利用することも視野に入れましょう

LightsailロードバランサーとELBの比較

| 項目 | Lightsail DNS | Route53 |
| --- | --- | --- |
| ゾーン管理 | 3つまで管理可能 | 500ゾーンまで管理可能 |
| ルーティングポリシー | 3つまで管理可能 | 500ゾーンまで管理可能 |
| ヘルスチェック | 機能なし | 機能あり |
| プライベートDNS | プライベートDNS管理不可能 | プライベートDNS管理可能 |

# Lightsailのストレージについて

---

- 仮想マシンにアタッチできるDISKとオブジェクトストレージの両方を提供
- 仮想マシンにアタッチできるDISK
    - サイズはカスタマイズで必要な量だけを選択可能
    - DISKタイプはSSDのみを提供
    - 最小8GBから作成でき最大16TBまで可能
    - リージョン内で20,000GBまで
- オブジェクトストレージ
    - 事前に必要量を確保した月次サービス（容量or転送量超えると追加課金）
    - 毎月AWS請求サイクル内で1回だけLightsailオブジェクトストレージのプランを変更可能
    - 総使用容量やバケット内のオブジェクト数の監視とアラーム設定が可能

Lightsail DISKとEBSの比較

| 項目 | Lightsail Disk | EBS |
| --- | --- | --- |
| ディスクタイプ | SSDのみ | SSDの他にgp3/gp2/IOPS SSDなど使用可能 |
| 暗号化 | デフォルトで暗号化 | 暗号化の選択はユーザーにて実施 |
| 最大容量 | 16TB | 64TB |
| 設定サイズ | 任意で指定可能 | 任意で指定可能 |

Lightsail オブジェクトストレージとS3の比較

- S3の機能をLightsailは踏襲しているため基本機能の違いはほとんどなし

| 項目 | Lightsail オブジェクトストレージ | S3 |
| --- | --- | --- |
| 料金 | 容量と転送料金の固定、※容量もしくは転送料金超えたら課金 | 従量課金 |

# Lightsail CDNとは

---

- 3つのプランがあり、1ヶ月当たりの転送量が決まった定額制
- 超過した分は別途料金が発生
- キャッシュの設定が予めプリセットで用意されており、独自の設定を追加することも可能
- SSL 証明書の発行やメトリクスの確認など、全操作がLightsailのコンソールから操作可能
- LightsailのDNSゾーンでCloudFrontのディストリビューションをAliasレコードで指定可能
- 一部CloudFront で出来た設定が出来ない
    - インスタンスへの直接アクセスをブロックできない
    - エラーの場合、カスタムエラーページを表示することができない
    - オリジンのフェイルオーバー機能がない

Lightsail CDNとCloudFrontの比較

| 項目 | Lightsail CDN | CloudFront |
| --- | --- | --- |
| 料金 | 容量と転送料金の固定、※容量もしくは転送料金超えたら課金 | 1TB のデータ転送 (アウト)、1,000 万件の HTTP または HTTPS リクエスト、2,000,000 件の CloudFront Function 呼び出しが毎月無料 |
| オリジン対象 | Lightsail インスタンス、コンテナ、ロードバランサーバケット | S3バケット、EC2、ELB、MediaStoreコンテナ、MediaPackageエンドポイント |
| オリジン | 複数のオリジンは不可能 | 複数のオリジン可能 |
| 向き不向き | インスタンスやロードバランサーなどのLightsailリソースでウェブサイトまたはウェブアプリケーションをホストしているユーザー向け | AWS で別のサービスを使用してウェブサイトやアプリをホストしている場合、複雑な設定が必要な場合、または1秒あたりのリクエスト数が多いワークロードや大量の動画ストリーミングで配信向け |

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
