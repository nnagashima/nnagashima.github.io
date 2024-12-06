# Prometheusとnode_exporterの導入

![prometheus-icon-511x512-1vmxbcxr.png](Prometheus%E3%81%A8node_exporter%E3%81%AE%E5%B0%8E%E5%85%A5%202f963ebfdbdb4f6f948e37adbf830e84/prometheus-icon-511x512-1vmxbcxr.png)

# 目次

---

# Prometheusとは

---

Prometheusは監視ツール本体で、GO言語で書かれたオープンソースになります。

関連するものでExporter、Alertmanagerが存在します。

※Prometheus2.0からはTSDBが利用されることになりパフォーマンスが改善されるようになりました。

ExporterはAgentでさまざまなものが存在しており代表的なのはnode_exporterがあります。

node_exporterは名前の通りノードのCPUメトリクスやMEMメトリクスなどの、

コンピュータリソースのメトリクスを取得します。

- 他にも利用用途が多そうなものを以下に記載しておきます。
    - node_exporter：[https://github.com/prometheus/node_exporter](https://github.com/prometheus/node_exporter)
    - blackbox_exporter：[https://github.com/prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
    - azure-metrics-exporter：[https://github.com/webdevops/azure-metrics-exporter](https://github.com/webdevops/azure-metrics-exporter)
    - cloudwatch-exporter：[https://github.com/prometheus/cloudwatch_exporter](https://github.com/prometheus/cloudwatch_exporter)
    - gcp-exporter：[https://github.com/DazWilkin/gcp-exporter](https://github.com/DazWilkin/gcp-exporter)
    - sakuracloud_exporter：[https://github.com/sacloud/sakuracloud_exporter](https://github.com/sacloud/sakuracloud_exporter)
    - apache_exporter：[https://github.com/Lusitaniae/apache_exporter](https://github.com/Lusitaniae/apache_exporter)
    - nginx_exporter：[https://github.com/nginxinc/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)
    - mysql_exporter：[https://github.com/prometheus/mysqld_exporter](https://github.com/prometheus/mysqld_exporter)
    - postgres_exporter：[https://github.com/prometheus-community/postgres_exporter](https://github.com/prometheus-community/postgres_exporter)

AlertmanagerはPrometheusで取得している内容をアラートルールに従って通知する機能や

重複除外やグルーピング機能を持ってます。

なお取得したデータ保存期間はデフォルトで15日となっているため、

延長したい場合にはRemoteStorageを用意する必要があります。

RemoteSotrageとは取得したメトリクスをローカルに持つストレージだけでなく、別のストレージに保管する機能になります。

# Prometheusの動作環境について

---

OSはOpenBSD / NetBSD / FreeBSD / DragonFlyBSD / Darwin / Linux / Windowsに対応しております。

# Prometheusとnode_exporterインストール

---

インストールする環境はRokcyLinux9で実施します。

リポジトリファイルにPrometheusのリポジトリを追加します。

```bash
# cat << EOF > /etc/yum.repos.d/prometheus.repo
[prometheus]
name=prometheus
baseurl=https://packagecloud.io/prometheus-rpm/release/el/$releasever/$basearch
repo_gpgcheck=1
enabled=1
gpgkey=https://packagecloud.io/prometheus-rpm/release/gpgkey
       https://raw.githubusercontent.com/lest/prometheus-rpm/master/RPM-GPG-KEY-prometheus-rpm
gpgcheck=1
metadata_expire=300
EOF
```

なお、RockyLinux9ではSHA-1アルゴリズムの仕様をポリシーで制限しています。

暗号署名を検証するためにSHA-1を仕様する場合は

以下コマンドでSHA-1のアルゴリズムを有効化します。

```bash
# update-crypto-policies --set DEFAULT:SHA1
```

Prometheusとnode_exporterをインストールします。

```bash
# dnf -y install prometheus2 node_exporter
```

# prometheus.ymlで監視設定とサービス起動

---

/etc/prometheus/promethus.ymlを編集します。

編集箇所は23行目と31行目になります。

23行目：ここでは監視する対象のホスト名を記載しています。

29-31行目：監視する対象で受付しているホスト名もしくはIPアドレス+Port番号を記載します。

これらを監視したい対象に対して追記する形になります。

※9090はPrometheus自身のポート番号、9100はnode_exporterのポート番号になります。

```bash
1  # my global config
2  global:
3    scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
4    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
5    # scrape_timeout is set to the global default (10s).
6
7  # Alertmanager configuration
8  alerting:
9    alertmanagers:
10     - static_configs:
11         - targets:
12            # - alertmanager:9093
13
14  # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
15  rule_files:
16    # - "first_rules.yml"
17    # - "second_rules.yml"
18
19  # A scrape configuration containing exactly one endpoint to scrape:
20  # Here it's Prometheus itself.
21  scrape_configs:
22    # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
23    - job_name: "ray-prometheus"
24
25      # metrics_path defaults to '/metrics'
26      # scheme defaults to 'http'.
27
28      static_configs:
29        - targets: 
30          - 'localhost:9090'
31          - 'localhost:9100'
```

prometheus.ymlの設定が誤っていないか確認します。

確認するツールはpromtoolを利用し、SUCCESSが表示されればOKです。

```bash
# promtool check config /etc/prometheus/prometheus.yml 
Checking /etc/prometheus/prometheus.yml
 SUCCESS: /etc/prometheus/prometheus.yml is valid prometheus config file syntax
```

Prometheusとnode_exporterの起動をします。

```bash
## サービスの起動
# systemctl start prometheus
# systemctl start node_exporter

## サービスが起動したか確認
# systemctl is-active node_exporter
active
# systemctl is-active prometheus
active
```

# 監視ができているか確認

---

http://IPアドレス:9090にURLでアクセスし、ターゲットがUPとなっていることを確認します。

画面は上のメニューのStatus→Targetsをクリックします。

![Untitled](Prometheus%E3%81%A8node_exporter%E3%81%AE%E5%B0%8E%E5%85%A5%202f963ebfdbdb4f6f948e37adbf830e84/Untitled.png)

あとは必要な情報をPromQLというPrometheus独自のQueryでデータを抽出することが可能です。

画面は上のメニューGraphから検索バーでPromQLを記載することで出力できます。

Prometheusの公式ページでもSampleのQueryを用意しているので参考にしてみてください。

https://prometheus.io/docs/prometheus/latest/querying/examples/

![Untitled](Prometheus%E3%81%A8node_exporter%E3%81%AE%E5%B0%8E%E5%85%A5%202f963ebfdbdb4f6f948e37adbf830e84/Untitled%201.png)

![Untitled](Prometheus%E3%81%A8node_exporter%E3%81%AE%E5%B0%8E%E5%85%A5%202f963ebfdbdb4f6f948e37adbf830e84/Untitled%202.png)