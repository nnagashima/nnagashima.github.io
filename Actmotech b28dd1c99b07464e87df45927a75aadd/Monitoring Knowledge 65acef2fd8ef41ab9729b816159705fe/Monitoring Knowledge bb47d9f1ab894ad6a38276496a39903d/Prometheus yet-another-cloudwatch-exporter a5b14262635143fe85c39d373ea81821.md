# Prometheus yet-another-cloudwatch-exporter

![prometheus-icon-511x512-1vmxbcxr (1).png](Prometheus%20yet-another-cloudwatch-exporter%20a5b14262635143fe85c39d373ea81821/prometheus-icon-511x512-1vmxbcxr_(1).png)

# 目次

---

# **yet-another-cloudwatch-exporterとは**

---

PrometheusでAWSのCloudWatchからメトリクスを収集するにはオフィシャルのcloudwatch_exporterがあります。

このexporterではGetMetricStatisticsを収集しているため、1度に1つのメトリクスしか取得することができません。

これをGetMetricDataで取得するこtができれば、1回で最大500個のメトリクスを取得することができます。

なお、API料金については取得するために叩いたAPIリクエスト数で月額の料金がきまるので、

料金を抑えるのであればGetMetricDataがもちろん良いです。

これを実現してくれるexporterがyet-another-cloudwatch-exporterになります。

[https://github.com/nerdswords/yet-another-cloudwatch-exporter](https://github.com/nerdswords/yet-another-cloudwatch-exporter)

# 利用前にAWS CLIのインストール

---

利用するOSはRokcyLinux9になります。

```bash
# dnf install unzip
# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
# unzip awscliv2.zip
# ./aws/install
# aws --version
aws-cli/2.9.18 Python/3.9.11 Linux/5.14.0-162.6.1.el9_1.0.1.x86_64 exe/x86_64.rocky.9 prompt/off
```

aws configureでAccessKeyとSecretKeyを登録します。

IAMポリシーは以下があれば良いと思います。

このポリシーを利用してIAMユーザを作成しAccessKeyとSecretKeyを作成してaws configureコマンドを用いて登録します。

※IAMロールを用いてAWS EC2に直接割り当てるでも可能です。

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudWatchExporter",
            "Effect": "Allow",
            "Action": [
                "tag:GetResources",
                "cloudwatch:ListTagsForResource",
                "cloudwatch:GetMetricData",
                "cloudwatch:ListMetrics"
            ],
            "Resource": "*"
        }
    ]
}
```

# **yet-another-cloudwatch-exporterのセットアップ**

利用するOSはRokcyLinux9になります。

上記Githubからクローンしてexporterをコンパイルします。※GO言語が取り扱える状態にしといてください。

```bash
# git clone https://github.com/nerdswords/yet-another-cloudwatch-exporter.git
# cd yet-another-cloudwatch-exporter/
# make build
# ./yace --help
NAME:
   Yet Another CloudWatch Exporter - YACE configured to retrieve CloudWatch metrics through the AWS API

USAGE:
   Yet Another CloudWatch Exporter [global options] command [command options] [arguments...]

VERSION:
   custom-build

AUTHOR:
   

COMMANDS:
   verify-config, vc  Loads and attempts to parse config file, then exits. Useful for CI/CD validation
   version, v         prints current yace version.
   help, h            Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --listen-address value          The address to listen on. (default: ":5000") [$listen-address]
   --config.file value             Path to configuration file. (default: "config.yml") [$config.file]
   --debug                         Add verbose logging. (default: false) [$debug]
   --fips                          Use FIPS compliant aws api. (default: false)
   --cloudwatch-concurrency value  Maximum number of concurrent requests to CloudWatch API. (default: 5)
   --tag-concurrency value         Maximum number of concurrent requests to Resource Tagging API. (default: 5)
   --scraping-interval value       Seconds to wait between scraping the AWS metrics (default: 300) [$scraping-interval]
   --metrics-per-query value       Number of metrics made in a single GetMetricsData request (default: 500) [$metrics-per-query]
   --labels-snake-case             If labels should be output in snake case instead of camel case (default: false)
   --help, -h                      show help
   --version, -v                   print the version
```

HELPを見て分かる通り、configが必要になりますので/etc/prometheus/cloudwatch-exporter-config.ymlを作成します。

※searchTagsの設定を入れて試していますが、どうにもうまく働いてくれないので引き続き調査していきます。

```bash
discovery:
  exportedTagsOnMetrics:
    AWS/EC2:
      - Name
      - ENV
      - Owner
  jobs:
  - type: AWS/EC2
    regions:
      - ap-northeast-1 
      - ap-northeast-3
    period: 60
    length: 300
    delay: 60
    enableMetricData: true
#    searchTags:
#      - Key: Name
#        Value: .*
    metrics:
      - name: CPUUtilization
        statistics:
        - Maximum
      - name: DiskReadBytes
        statistics:
        - Maximum
      - name: DiskWriteBytes
        statistics:
        - Maximum
      - name: NetworkIn
        statistics:
        - Sum
      - name: NetworkOut
        statistics:
        - Sum
```

以下コマンドでexporterを実行します。

http://IPアドレス:5000/metricsでアクセスすることでさまざまな値が取れるようになっています。

```bash
./yace --config.file /etc/prometheus/cloudwatch-exporter-config.yml
```

# yaceサービス化

---

yet-another-cloudwatch-exporterがプログラムで動いているので、サービス登録して動かします。

```bash
# vi /etc/systemd/system/yace.service
[Unit]
Description = prometheus yet-another-cloudwatch-exporter

[Service]
ExecStart = /usr/bin/yace —config.file /etc/prometheus/cloudwatch-exporter-config.yml
Restart = always
Type = simple

[Install]
WantedBy = multi-user.target
```

サービスファイルを作成したらdaemon-reloadをしてサービスを起動します。

```bash
# systemctl daemon-reload
# systemctl start yace
# systemctl enable yace
```

# Prometheusへの取り込み

---

最後に/etc/prometheus/prometheus.ymlに上記メトリクスを取り込むための設定を追記します。

```bash
- job_name: 'cloudwatch'
    scrape_interval: 5m
    metrics_path: /metrics
    static_configs:
      - targets:
          - 'IPアドレス:5000'
```