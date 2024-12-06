# Amazon Managed Prometheus Serviceについて

![Managed Service for Prometheus.png](Amazon%20Managed%20Prometheus%20Service%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%2075a32b2bd4c240c8be8aa8a9dfc5ed89/Managed_Service_for_Prometheus.png)

# 目次

---

# Amazon Managed Prometheus Service(AMP)について

---

Prometheusは長期データ保存ができない設計となっております。

長期で保存したい場合には別のストレージに書き出す必要があり、その中の一つとなるものです。

他にもストレージにはVictoriaMetricsが有名どころであるがナレッジがまだ少ないですね。

選択できるストレージは以下に記載がありますので参考にしてみてください。

https://prometheus.io/docs/prometheus/latest/storage/

自身で調べた限りのAMPSの特徴は以下です。

- 150日分のデータがデフォルトで保存可能だが申請することで150日以上保存が可能
- SLAは99.9%
- オートスケールが可能で、3AZにまたがって冗長化されている
- 格納データからクエリを発行してSNS経由でアラート通知が可能（Alertmangerの代わりが可能かもしれない）
- 採用されている技術はCoretex

https://github.com/cortexproject/cortex

- AWSが提供しているマネージドサービスのためストレージの運用管理が不要

これらの機能を踏まえて自宅にあるKubernetes環境で動いているPrometheusのデータをAMPSにリモートで書き込むには、

1. 自宅k8sのPrometheusをソース元を絞って外部公開し、AWS上に立てたPrometheusにフェデレーションする
2. 自宅ルータのFortigateとAWS Site to Site VPNを使ってIPSec接続し、AWS上に立てたPrometheusにフェデレーションする
3. SSM Agentを利用してRemoteWriteを実現する

この中で簡単に実現かつセキュリティ的に安心できるのは2なのかなと思いますので、2で実現してみたいと思います。

なお、金額についてですがAMPを東京リージョンで利用した場合、

PrometheusサーバとPrometheusサーバに仕込んだnode_exportのデータ1日の料金を、

AWS Cost Exporterで確認したところ$0.73でした。

これを1ヶ月を30日と考えた際に$21.9となり、1USDを130円とした時に約3000円になります。

# PrometheusをAWS上に作成

---

前提FortigateとSite to Site VPNの接続ができていること

[Site to Site VPNとFortigateでIPSecVPN](../../AWS%20Knowledge%204ff04ac2d7b44dc18b787d5954b927a0/Site%20to%20Site%20VPN%E3%81%A8Fortigate%E3%81%A6%E3%82%99IPSecVPN%206a143fc57a95452092712545087a7b40.md)

Prometheus作成方法はOS直インストールする方法になりますが、以下を参照してPrometheusを作成してください。

[Prometheusとnode_exporterの導入](Prometheus%E3%81%A8node_exporter%E3%81%AE%E5%B0%8E%E5%85%A5%202f963ebfdbdb4f6f948e37adbf830e84.md)

EC2からAMPにRemoteWriteに接続するには以下を参考にしながら設定しました。

https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-remote-write-EC2.html

# AMPの設定

---

AWSコンソールにログインしてAMPのワークスペースを作成します。

作成ができたらリモート書き込みURLが発行されるので、URLを控えておきます。

# AMPに書き込み許可するIAM Policyを作成

---

必要なポリシーは「AmazonPrometheusRemoteWriteAccess」になります。

ポリシーを作成したら、先ほどAWS上に作成したPrometheusにIAMロールを割り当てし、OS再起動します。

# AMP RemoteStorageの設定

---

/etc/prometheus/prometheus.ymlに以下を記載します。

他にも設定できるパラメータは以下があります。

https://prometheus.io/docs/practices/remote_write/

- max_samples_per_send：
    
    一度に纏めて送信するサンプルの数
    
    キューに10000個サンプルがあり、設定値を1000にするとサンプルを1000個ずつに5回に分けて送信します。
    
    送信されるタイミングはサンプルが1000個溜まった時点で外部ストレージに送信されます。
    
- max_shards：
    
    remote writeの並列実行数の上限
    
    非常に遅いエンドポイントへのリモート書き込みでない限り、デフォルトを超えて増加する必要はほとんどないそうです。
    
- capacity：
    
    サンプルを格納できる最大容量
    
    スクレイピングで取得したサンプル数がcapacityを超えた場合、超えた分のサンプルは破棄されます。
    
    例えば100のcapacityに対してスクレイピングしたサンプル数が130個となると、30個のサンプルが送信されず消失します。
    

　　max_samples_per_sendの3から10倍に設定が推奨のようですが容量が多すぎるとメモリが過剰に消費されます。

```bash
remote_write:
  -
    url: https://aps-workspaces.ワークスペースを作成したリージョン.amazonaws.com/workspaces/ワークスペースID/api/v1/remote_write
    queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500
    sigv4:
         region: ワークスペースを作成したリージョン
```

Prometheusのサービス再起動をします。

```bash
# systemctl restart prometheus
# systemctl status prometheus
→AMPのワークスペースと正常に接続できれていればログに表示されます。
```

# AWS上に作成したPrometheusから自宅k8sのPrometheusへフェデレーション

---

k8s上のPrometheusをNodePortで外部公開できるようにします。

その後AWS上で作成したPrometheusから、k8s上のPrometheusの情報を全て取得するために、

/etc/prometheus/prometheus.ymlに以下のように追記します。

```bash
- job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job=".+"}'
        - '{__name__=~".+"}'

    static_configs:
      - targets:
        - 'IPアドレス:30090'
```

https://prometheus.io/docs/prometheus/latest/federation/

# AMPにPrometheusのデータを送信する際に、PrometheusをActive-Active構成にした場合

---

Prometheus自体にHA機能はありません。

またHAの構成自体をPrometheusは否定をしておりActive-Activeで作るように展開している

ただActive-Activeで作るとメトリクスが重複してしまうので以下の機能を利用します。

Send high-availability data to Amazon Managed Service for Prometheus with Prometheus

https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-ingest-dedupe.html

/etc/prometheus/prometheus.ymlに以下を記載します。

```bash
## Prometheus01
global:
  external_labels:
      cluster: prometheus-cluster
      __replica__: ray-prometheus01

## Prometheus02
global:
  external_labels:
      cluster: prometheus-cluster
      __replica__: ray-prometheus02
```

Prometheusのサービス再起動をします。

```bash
# systemctl restart prometheus
```