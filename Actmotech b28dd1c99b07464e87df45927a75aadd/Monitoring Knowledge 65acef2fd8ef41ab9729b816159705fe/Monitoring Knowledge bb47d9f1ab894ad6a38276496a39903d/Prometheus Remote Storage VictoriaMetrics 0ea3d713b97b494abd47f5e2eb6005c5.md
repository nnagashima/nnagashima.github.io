# Prometheus Remote Storage VictoriaMetrics

![b5a73280-7d50-11e9-87e3-b0ff44e0673e.png](Prometheus%20Remote%20Storage%20VictoriaMetrics%200ea3d713b97b494abd47f5e2eb6005c5/b5a73280-7d50-11e9-87e3-b0ff44e0673e.png)

# 目次

---

# はじめに

---

Prometheusは長期データ保存ができない設計となっております。

長期で保存したい場合には別のストレージに書き出す必要があり、その中の一つとなるものです。

# VictoriaMetricsのデプロイ

---

VictoriaMetricsaは公式でHelmが用意されているので、Helmを利用します。

まずはレポジトリをダウンロードして最新化します。

```bash
# helm repo add vm https://victoriametrics.github.io/helm-charts/
# helm repo update
```

最新化したらNamespaceを作成します。

```bash
# kubectl create namespace vm
```

VictoriaMetricsをデプロイするにあたって設定ファイルを確認します。

```bash
# helm show values vm/victoria-metrics-cluster > vm-config.yml
・Incert/SelectのTypeをLoadBalanacerに変更しています。
・PVのAccessModeをReadWriteManyにしています。
```

VictoriaMetricsをデプロイします。

```bash
# helm install vmcluster vm/victoria-metrics-cluster -f vm-config.yml -n vm
NAME: vmcluster
LAST DEPLOYED: Mon May  1 23:28:45 2023
NAMESPACE: vm
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Write API:

The Victoria Metrics write api can be accessed via port 8480 with the following DNS name from within your cluster:
vmcluster-victoria-metrics-cluster-vminsert.vm.svc.cluster.local

Get the Victoria Metrics insert service URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace vm -l "app=vminsert" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace vm port-forward $POD_NAME 8480

You need to update your Prometheus configuration file and add the following lines to it:

prometheus.yml

    remote_write:
      - url: "http://<insert-service>/insert/0/prometheus/"

for example -  inside the Kubernetes cluster:

    remote_write:
      - url: "http://vmcluster-victoria-metrics-cluster-vminsert.vm.svc.cluster.local:8480/insert/0/prometheus/"
Read API:

The VictoriaMetrics read api can be accessed via port 8481 with the following DNS name from within your cluster:
vmcluster-victoria-metrics-cluster-vmselect.vm.svc.cluster.local

Get the VictoriaMetrics select service URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace vm -l "app=vmselect" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace vm port-forward $POD_NAME 8481

You need to specify select service URL into your Grafana:
 NOTE: you need to use the Prometheus Data Source

Input this URL field into Grafana

    http://<select-service>/select/0/prometheus/

for example - inside the Kubernetes cluster:

    http://vmcluster-victoria-metrics-cluster-vmselect.vm.svc.cluster.local:8481/select/0/prometheus/
```

デプロイ後の結果を確認します。

```bash
# kubectl get pods -A | grep 'vminsert\|vmselect\|vmstorage'
vm               vmcluster-victoria-metrics-cluster-vminsert-86c484d8f6-5pkhw      1/1     Running                  0               81m
vm               vmcluster-victoria-metrics-cluster-vminsert-86c484d8f6-k8lqf      1/1     Running                  0               81m
vm               vmcluster-victoria-metrics-cluster-vmselect-6b9f84b8f8-62gsx      1/1     Running                  0               81m
vm               vmcluster-victoria-metrics-cluster-vmselect-6b9f84b8f8-z6bpk      1/1     Running                  0               81m
vm               vmcluster-victoria-metrics-cluster-vmstorage-0                    1/1     Running                  0               81m
vm               vmcluster-victoria-metrics-cluster-vmstorage-1                    1/1     Running                  0               81m
```

# データ収集用のPrometheusの準備

---

VMでOSはUbuntu22.04でPrometheusを用意しています。

本Prometheusサーバで集約してVictoriaMetricsに監視データを投げる形を取りたいと思います。

```bash
# apt install -y prometheus prometheus-node-exporter
# systemctl --now enable prometheus prometheus-node-exporter

# prometheus --version
prometheus, version 2.31.2+ds1 (branch: debian/sid, revision: 2.31.2+ds1-1ubuntu1)
  build user:       team+pkg-go@tracker.debian.org
  build date:       20220317-16:26:29
  go version:       go1.17.3
  platform:         linux/amd64

# prometheus-node-exporter --version
node_exporter, version 1.3.1 (branch: debian/sid, revision: 1.3.1-1)
  build user:       team+pkg-go@tracker.debian.org
  build date:       20220114-23:26:34
  go version:       go1.17.3
  platform:         linux/amd64
```

Prometheusの設定ファイルを以下のようにします。

```bash
# cat /etc/prometheus/prometheus.yml 
# Sample config for Prometheus.

global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

#PrometheusサーバのデータをVictoriaMetricsへの書き込み
remote_write:
  -  
    url: "http://VictoriaMetricsのLBIP:8480/insert/0/prometheus/"
    queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  # Prometheusサーバ自身の監視 
  - job_name: 'ray-prometheus01'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['PrometheusサーバのIP:9090']
      - targets: ['PrometheusサーバのIP:9100']

#kubernetesで動作しているPrometheusのデータをフェデレーションします。
  - job_name: 'k8s-prometheus'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job=".+"}'
        - '{__name__=~".+"}'

    static_configs:
      - targets:
        - 'k8s-prometheusのLBIP:9090'
```

# Prometheusの再起動

---

PrometheusのConfigが書き終わったら設定反映のために再起動します。

```bash
# promtool check config /etc/prometheus/prometheus.yml
・Configのチェックを実施します。

# systemctl restart prometheus
# systemctl status prometheus
※抜粋していますが、ログからもRemoteStorageに対して書き込みができているようです。
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.613Z caller=head.go:598 level=info component=tsdb msg="WAL replay completed" checkpoint_replay_duration=121.523µs wal_replay_duration=5.733468949s total_replay_duration=5.880153109s
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.762Z caller=main.go:850 level=info fs_type=EXT4_SUPER_MAGIC
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.762Z caller=main.go:853 level=info msg="TSDB started"
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.762Z caller=main.go:980 level=info msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.763Z caller=dedupe.go:112 component=remote level=info remote_name=f085ec url=http://192.168.11.156:8480/insert/0/prometheus/ msg="Starting WAL watcher" queue=f085ec
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.763Z caller=dedupe.go:112 component=remote level=info remote_name=f085ec url=http://192.168.11.156:8480/insert/0/prometheus/ msg="Starting scraped metadata watcher"
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.763Z caller=dedupe.go:112 component=remote level=info remote_name=f085ec url=http://192.168.11.156:8480/insert/0/prometheus/ msg="Replaying WAL" queue=f085ec
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.763Z caller=main.go:1017 level=info msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml totalDuration=856.46µs db_storage=2.374µs remote_storage=355.63µs web_handler=1.397µs query_engine=908ns scrape=124.945µs scrape_sd=23.466µs notify=14.178µs notify_sd=8.172µs rules=1.816µs
May 02 08:00:59 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:00:59.763Z caller=main.go:795 level=info msg="Server is ready to receive web requests."
May 02 08:01:06 ray-prometheus01 prometheus[2939]: ts=2023-05-02T08:01:06.496Z caller=dedupe.go:112 component=remote level=info remote_name=f085ec url=http://192.168.11.156:8480/insert/0/prometheus/ msg="Done replaying WAL" duration=6.733250561s
```

# GrafanaへDetaSource登録

---

DataSource登録する際にはSeletctのURLになるので、http://VictoriaMetricsのLBIP:8481/select/0/prometheus/になります。

登録が完了しましたら、VictoriaMetrics経由でグラフが閲覧できるかGrafanaで確認し、閲覧ができていれば問題なく完了です。