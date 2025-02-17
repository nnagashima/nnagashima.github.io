[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# CloudWatchとCloudWatchAgentについて

![](/AWS/CloudWatchとCloudWatchAgentについて/image01.png)

# 目次

- [CloudWatchとCloudWatchAgentについて](#cloudwatchとcloudwatchagentについて)
- [目次](#目次)
- [はじめに](#はじめに)
- [CloudWatchとは](#cloudwatchとは)
- [リソースをモニタリングするCloudWatchとCloudWatchAgent](#リソースをモニタリングするcloudwatchとcloudwatchagent)
- [CloudWatchAgentのパラメータ作成](#cloudwatchagentのパラメータ作成)
- [CloudWatchAgentの起動](#cloudwatchagentの起動)
- [CloudWatchAgentを使ってプロセスモニタリング](#cloudwatchagentを使ってプロセスモニタリング)
- [アラーム設定について](#アラーム設定について)
- [CloudWatchAgentの料金について](#cloudwatchagentの料金について)
- [最後に](#最後に)

---

# はじめに

---

AWS CloudWatchについて公式ドキュメント見れば分かりますが、

 今まで調べてきたものを忘れないようにするために自分の備忘録として記載します。

# CloudWatchとは

---

AWSリソースとAWSで実行されているアプリケーションをリアルタイムでモニタリングします。

 AWSのサービスをデプロイするとメトリクスが自動的に表示されるのでメトリクスをモニタリングして、 

閾値を超えたら通知を実施したり、リソースを自動的に変更するアラームを作ったりすることができます。

CloudWatchには以下の機能があります。 

1. リソースをモニタリングするCloudWatch 
2. ログ集めて監視するCloudWatch Logs 
3. メールなどで通知を行うCloudWatch Alerm 
4. イベントをトリガーとして別のアクションを実行するCloudWatch Events （今はEventBridge）
5. WebページやAPIエンドポイントに対してモニタリングするSynthetic Monitoring 
6. リアルユーザーモニタリング機能のCloudWatch RUM（Real-User Monitoring）

今回は1から3について触れていきたいと思います。

# リソースをモニタリングするCloudWatchとCloudWatchAgent

---

先ほど記載したようにリソースをデプロイすると自動的にCloudWatchにメトリクスが生成されます。 

このメトリクスはAWSにて決まっており、値のモニタリング方法については各メトリクスによって統計が変わってきますので、

[ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)からサービス毎に適切なモニタリングでアラート通知をするようにしましょう。

EC2に関しては標準のメトリクスだとメモリ、ディスク、プロセス、ログといったものは監視することができません。 

そこを補うのがCloudWatch Agentになり、インストールすることで足りない部分のメトリクスを補うことができます。 

CloudWatchAgentはインターネットへアウトバンドでアクセスできる必要もしくはVPCエンドポイント経由でアクセスできる必要があります。

 CloudWatchAgentをインストールする前にEC2にはCloudWatchAgentServerPolicyが割り当てられたIAMロールを適用するようにしてください。

 似たようなポリシーでCloudWatchAgentAdminPolicyがありますが、

Agentの設定をSystems Manager のパラメータストアに格納する際にはこちらを使用します。

インストール方法は2種類あり、コマンドライン経由で実施するかSSM Agentが導入されていればSSM経由でインストールすることができます。

- https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/installing-cloudwatch-agent-commandline.html
- https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/installing-cloudwatch-agent-ssm.html

# CloudWatchAgentのパラメータ作成

---

前述でAgentのインストールが完了したら設定ファイルを作成します。

 実施する方法はCloudWatchAgentをインストールした対象でウィーザードを使って実施するか、 

Systems Mangerのパラメータストアにて直接パラメータを作成するかのどちらかになります。 

今回ははウィーザードを利用したパターンを記載したいと思います。

```bash
# /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

#OSの種別はなんですか？
On which OS are you planning to use the agent?
1. linux
2. windows
3. darwin
default choice: [1]:

#使うのはEC2ですか？オンプレミスですか？
Trying to fetch the default region based on ec2 metadata...
Are you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]:

#Agentを稼働させるユーザーはなんですか？
Which user are you planning to run the agent?
1. root
2. cwagent
3. others
default choice: [1]:

#StatsD daemonは起動させますか？
#https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html
Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:

#StatsDの起動ポートは？
Which port do you want StatsD daemon to listen to?
default choice: [8125]

#StatsDのデータ取得間隔は？
What is the collect interval for StatsD daemon?
1. 10s
2. 30s
3. 60s
default choice: [1]:
3

#StatsDが収集したデータを集計する感覚は？
What is the aggregation interval for metrics collected by StatsD daemon?
1. Do not aggregate
2. 10s
3. 30s
4. 60s
default choice: [4]:

#StatsDが収集したメトリクスを監視する？
Do you want to monitor metrics from CollectD? WARNING: CollectD must be installed or the Agent will fail to start
1. yes
2. no
default choice: [1]:

#CPUやメモリ監視する？
Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:

#CPUをコア当たりで監視する？
Do you want to monitor cpu metrics per core?
1. yes
2. no
default choice: [1]:

#CloudWatchのメトリクスに利用可能なDimensionを追加する？
Do you want to add ec2 dimensions (ImageId, InstanceId, InstanceType, AutoScalingGroupName) into all of your metrics if the info is available?
1. yes
2. no
default choice: [1]:

#EC2のDimansionを集約しますか？
Do you want to aggregate ec2 dimensions (InstanceId)?
1. yes
2. no
default choice: [1]:

#メトリクスを高頻度で取得しますか？
Would you like to collect your metrics at high resolution (sub-minute resolution)? This enables sub-minute resolution for all metrics, but you can customize for specific metrics in the output json file.
1. 1s
2. 10s
3. 30s
4. 60s
default choice: [4]:

#取得するメトリクスの種類
#以下を参照してください
#https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html

Which default metrics config do you want?
1. Basic
2. Standard
3. Advanced
4. None
default choice: [1]:
3

#作成されたConfig
Current config as follows:
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "metrics": {
                "aggregation_dimensions": [
                        [
                                "InstanceId"
                        ]
                ],
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "collectd": {
                                "metrics_aggregation_interval": 60
                        },
                        "cpu": {
                                "measurement": [
                                        "cpu_usage_idle",
                                        "cpu_usage_iowait",
                                        "cpu_usage_user",
                                        "cpu_usage_system"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ],
                                "totalcpu": false
                        },
                        "disk": {
                                "measurement": [
                                        "used_percent",
                                        "inodes_free"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "diskio": {
                                "measurement": [
                                        "io_time",
                                        "write_bytes",
                                        "read_bytes",
                                        "writes",
                                        "reads"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "netstat": {
                                "measurement": [
                                        "tcp_established",
                                        "tcp_time_wait"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 60,
                                "service_address": ":8125"
                        },
                        "swap": {
                                "measurement": [
                                        "swap_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        }
                }
        }
}

#設定内容に問題ありませんか？
Are you satisfied with the above config? Note: it can be manually customized after the wizard completes to add additional items.
1. yes
2. no
default choice: [1]:

#以前の設定ファイルを移行するためにインポートファイルはありますか？
Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?
1. yes
2. no
default choice: [2]:

#ログ監視はしますか？
Do you want to monitor any log files?
1. yes
2. no
default choice: [1]:
1

#監視するログファイルパスを記載してください
Log file path:
/var/log/messages

#CloudWatchLogsのLogGroup名を設定してください
Log group name:
default choice: [messages]

#CloudWatchLogsのStream名を設定してください
Log stream name:
default choice: [{instance_id}]

#ログ保持日数を選択してください
Log Group Retention in days
1. -1
2. 1
3. 3
4. 5
5. 7
6. 14
7. 30
8. 60
9. 90
10. 120
11. 150
12. 180
13. 365
14. 400
15. 545
16. 731
17. 1827
18. 3653
default choice: [1]:
3

#追加するログファイルはありますか？
Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]:
2

#作成されたConfig
Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.
Current config as follows:
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "/var/log/messages",
                                                "log_group_name": "messages",
                                                "log_stream_name": "{instance_id}",
                                                "retention_in_days": 3
                                        }
                                ]
                        }
                }
        },
        "metrics": {
                "aggregation_dimensions": [
                        [
                                "InstanceId"
                        ]
                ],
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "collectd": {
                                "metrics_aggregation_interval": 60
                        },
                        "cpu": {
                                "measurement": [
                                        "cpu_usage_idle",
                                        "cpu_usage_iowait",
                                        "cpu_usage_user",
                                        "cpu_usage_system"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ],
                                "totalcpu": false
                        },
                        "disk": {
                                "measurement": [
                                        "used_percent",
                                        "inodes_free"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "diskio": {
                                "measurement": [
                                        "io_time",
                                        "write_bytes",
                                        "read_bytes",
                                        "writes",
                                        "reads"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "netstat": {
                                "measurement": [
                                        "tcp_established",
                                        "tcp_time_wait"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 60,
                                "service_address": ":8125"
                        },
                        "swap": {
                                "measurement": [
                                        "swap_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        }
                }
        }
}
Please check the above content of the config.
The config file is also located at /opt/aws/amazon-cloudwatch-agent/bin/config.json.
Edit it manually if needed.

#SSMパラメータストアに保存しますか？
Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]:
2
Program exits now.
```

# CloudWatchAgentの起動

---

設定が終わったらCloudWatchAgentが起動するために必要なcollectdをインストールし、CloudWatchAgentを起動します。

```
# amazon-linux-extras install collectd
# /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
# systemctl status amazon-cloudwatch-agent.service
● amazon-cloudwatch-agent.service - Amazon CloudWatch Agent
   Loaded: loaded (/etc/systemd/system/amazon-cloudwatch-agent.service; enabled; vendor preset: disabled)
   Active: active (running) since 金 2022-04-22 13:51:26 UTC; 35min ago
```

起動するとCloudWatchのCWAgentというNamespacesにメトリクスが取り込まれています。 

同じくCloudWatchLogsにも/var/log/messagesのログが取り込まれています。

# CloudWatchAgentを使ってプロセスモニタリング

---

CloudWatchAgentでプロセスモニタリングはprocstatを使います。

 /opt/aws/amazon-cloudwatch-agent/bin/配下でconfig2.jsonを作成します。

config2.jsonの中身 

※今回はDockerプロセスをモニタリングします。

```bash
{
    "metrics": {
        "metrics_collected": {
            "procstat": [
                {
                    "exe": "docker",
                    "measurement": [
                        "pid_count"
                    ],
                    "metrics_collection_interval": 60
                }
            ]
        }
    }
}
```

configが記載できたら以下コマンドでコンフィグを読み込みます。 

最初に起動した時は fetch-configを使っていますが、追加でConfigを読み込みたい場合はappend-configを使ってください。

```bash
# /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a append-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config2.json
```

しばらくするとCloudWatchにメトリクスが生成されます。

# アラーム設定について

---

メトリクスについては適切な統計情報のもとで設定をしていけば良いですが、 

ログについては事前にメトリックフィルターでフィルターパターンを作るようにしてください。 

フィルターパターンを作るとマッチしたログの数のカウント情報が生成されます。

# CloudWatchAgentの料金について

---

無料枠と有料枠があるので注意してください。 

https://aws.amazon.com/jp/cloudwatch/pricing/

# 最後に

---

Windowsの場合は触れていませんが、イベントログをレベルと対象ログごとにモニタリングすることができます。

プロセスも同様にpid_file形式でモニタリングすることができます。 

CloudWatchここまでできるのはいいですが、Apacheのステータスページのモニタリングとかミドルウェア周りが弱いので、

他のツールで補う必要があるかなと思います。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
