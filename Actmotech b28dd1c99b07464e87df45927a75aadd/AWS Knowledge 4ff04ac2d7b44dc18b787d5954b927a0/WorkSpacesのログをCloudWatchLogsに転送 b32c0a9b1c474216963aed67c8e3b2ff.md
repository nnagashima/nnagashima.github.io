# WorkSpacesのログをCloudWatchLogsに転送

![desktopappstreaming-amazonworkspaces-icon-1783x2048-s67ycgjv.png](WorkSpaces%E3%81%AE%E3%83%AD%E3%82%AF%E3%82%99%E3%82%92CloudWatchLogs%E3%81%AB%E8%BB%A2%E9%80%81%20b32c0a9b1c474216963aed67c8e3b2ff/desktopappstreaming-amazonworkspaces-icon-1783x2048-s67ycgjv.png)

# 目次

---

# はじめに

---

CloudWatchLogsにWorkSpacesにアップロードすることをやったので、備忘録に残します。

# VPCエンドポイントの作成

---

今回説明に利用している環境はプライベートサブネットですが、パブリックサブネットにNATゲートウェイを配置しているため、

NATゲートウェイ経由でCloudWatchLogsへログ転送しています。

セキュアに実施するのが一般的だと思いますが、その際には以下のVPCエンドポイントを作成してください。

- **com.amazonaws.ap-northeast-1.logs**

VPCエンドポイントに紐づけるセキュリティグループはインバウンド443で開放をしてください。

ソースも絞りたい場合にはエンドポイントを利用するサブネットのIPアドレス範囲を記載してください。

エンドポイントポリシーを利用して制限をかけたい場合の例も記載します。（特定のARNに対してCloudWatchLogsへの書き込みを許可）

```bash
{
  "Statement": [
    {
      "Sid": "PutOnly",
      "Principal": "*",
      "Action": [
				"logs:CreateLogGroup",
        "logs:CreateLogStream",
				"logs:PutRetentionPolicy",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:logs:ap-northeast-1:AWSアカウントID:log-group:workspaces-system-eventlog:*"
    }
  ]
}
```

# CloudWatchAgentのインストール

---

SSMでもできますが、今回は事前にダウンロードしてきたCloudWatchAgentのMSIを使ってインストールします。

WorkSpacesで管理者モードでPowershellを実行し、CloudWatchAgentのMSIファイルがある場所に移動してインストールを実行します。

```bash
msiexec /i amazon-cloudwatch-agent.msi
```

# IAMグループ、ユーザー、ポリシーを作成

---

IAMグループを作成します。IAMグループを作成してユーザーをどうして所属させるかというとSecurityHubで検査した時に引っかかるからです。

```bash
グループ名：
wslogs-to-cwlogs-grp01
```

IAMポリシーを作成して、先ほど作成したグループにポリシーをアタッチします。

```bash
ポリシー名：
wslogs-to-cwlogs-pol01

ポリシー内容：
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:ap-northeast-1:AWSアカウントID:log-group:workspaces-system-eventlog:*"
        }
    ]
}
```

IAMユーザーを作成し、ユーザーを先ほど作成したグループに追加します。AWSコンソールへのアクセスできる機能は不要となります。

またIAMユーザー作成後にアクセスキーの発行を実施します。

```bash
ユーザー：
wslogs-to-cwlogs-user01
```

# WorkSpacesの環境変数を編集

---

スタート＞設定＞詳細情報＞システムの詳細設定＞環境変数をクリックします。

システム環境変数に以下のように入力します。

```bash
AWS_REGION : リージョン名を記入（例 : ap-northeast-1
AWS_SECRET_ACCESS_KEY : シークレットアクセスキー
AWS_ACCESS_KEY_ID : アクセスキー
```

# Powershellの実行権限変更

---

以下のように実行権限がなっています。

```bash
> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine      Restricted
```

CloudWatchAgentのスクリプトを実行できないので、以下のコマンドを実行して権限を与えます。

```bash
> Set-ExecutionPolicy RemoteSigned -Scope Process

実行ポリシーの変更
実行ポリシーは、信頼されていないスクリプトからの保護に役立ちます。実行ポリシーを変更すると、about_Execution_Policies
のヘルプ トピック (https://go.microsoft.com/fwlink/?LinkID=135170)
で説明されているセキュリティ上の危険にさらされる可能性があります。実行ポリシーを変更しますか?
[Y] はい(Y)  [A] すべて続行(A)  [N] いいえ(N)  [L] すべて無視(L)  [S] 中断(S)  [?] ヘルプ (既定値は "N"): y
```

Process部分がRemote Signedに変更されていれば実行権限変更完了です。

```bash
> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process    RemoteSigned
  CurrentUser       Undefined
 LocalMachine      Restricted
```

# CloudWatchAgentのConfig作成

---

ウィーザードに従い以下のようにしていきます。

```bash
PS C:\> cd 'C:\Program Files\Amazon\AmazonCloudWatchAgent\'
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-config-wizard.exe
================================================================
= Welcome to the Amazon CloudWatch Agent Configuration Manager =
=                                                              =
= CloudWatch Agent allows you to collect metrics and logs from =
= your host and send them to CloudWatch. Additional CloudWatch =
= charges may apply.                                           =
================================================================
On which OS are you planning to use the agent?
1. linux
2. windows
3. darwin
default choice: [2]:
2
Trying to fetch the default region based on ec2 metadata...
I! imds retry client will retry 1 timesAre you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]:　#ここではOn-Premisesを選択してください。
2
Please make sure the credentials and region set correctly on your hosts.
Refer to http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html
Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:
2
Do you have any existing CloudWatch Log Agent configuration file to import for migration?
1. yes
2. no
default choice: [2]:
2
Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:
2
Do you want to monitor any customized log files?
1. yes
2. no
default choice: [1]:
2
Do you want to monitor any Windows event log?
1. yes
2. no
default choice: [1]: #ログをアップロードするのでyesを選択します。
1
Windows event log name:
default choice: [System]　#SystemのイベントログをアップロードするのでSystemと入力します。アップロード対象はERRORとCRITICALのみにしてます。
System
Do you want to monitor VERBOSE level events for Windows event log System ?
1. yes
2. no
default choice: [1]:
2
Do you want to monitor INFORMATION level events for Windows event log System ?
1. yes
2. no
default choice: [1]:
2
Do you want to monitor WARNING level events for Windows event log System ?
1. yes
2. no
default choice: [1]:
2
Do you want to monitor ERROR level events for Windows event log System ?
1. yes
2. no
default choice: [1]:
1
Do you want to monitor CRITICAL level events for Windows event log System ?
1. yes
2. no
default choice: [1]:
1
Log group name:
default choice: [System] #ログのグループ名を任意で決めます。
workspaces-system-eventlog
Log stream name:
default choice: [{hostname}] #ログのストリーム名を任意で決めます。ここではメタデータの内容を利用できます。（http://169.254.169.254/latest/meta-data/で表示される内容）
{local_hostname}
Which log group class would you like to have for this log group? #ログクラスはSTANDARDにします。
1. STANDARD
2. INFREQUENT_ACCESS
default choice: [1]:
1
In which format do you want to store windows event to CloudWatch Logs? #ログのアップロード形式はテキストにします。
1. XML: XML format in Windows Event Viewer
2. Plain Text: Legacy CloudWatch Windows Agent (SSM Plugin) Format
default choice: [1]:
2
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
17. 1096
18. 1827
19. 2192
20. 2557
21. 2922
22. 3288
23. 3653
default choice: [1]: #ログの保存期間は14日にします。
6
Do you want to specify any additional Windows event log to monitor?
1. yes
2. no
default choice: [1]:
2
Do you want the CloudWatch agent to also retrieve X-ray traces?
1. yes
2. no
default choice: [1]:
2
Existing config JSON identified and copied to:  D:\Users\nnagashima\AppData\Roaming\Amazon\CloudWatchAgent\etc\backup-configs
Saved config file to config.json successfully.
Current config as follows:
{
        "logs": {
                "logs_collected": {
                        "windows_events": {
                                "collect_list": [
                                        {
                                                "event_format": "text",
                                                "event_levels": [
                                                        "ERROR",
                                                        "CRITICAL"
                                                ],
                                                "event_name": "System",
                                                "log_group_class": "STANDARD",
                                                "log_group_name": "workspaces-system-eventlog",
                                                "log_stream_name": "{local_hostname}",
                                                "retention_in_days": 14
                                        }
                                ]
                        }
                }
        }
}
Please check the above content of the config.
The config file is also located at config.json.
Edit it manually if needed.
Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]:
2
Please press Enter to exit...

Program exits now.
```

途中ででてきた以下について補足で低頻度アクセス用のInfrequent Accessが作られました。

Stanadardと比較すると機能制限はありますが、料金が安くなるため料金の最適化が可能になりました。

Infrequent Accessでできること

- ログの取り込みと保存
- クロスアカウントのログ転送
- KMSによる暗号化

上記以外はできませんが、転送料金が1GB単位あたりStandardの半額になります。

```jsx
Which log group class would you like to have for this log group? #ログクラスはSTANDARDにします。
1. STANDARD
2. INFREQUENT_ACCESS
```

作成したConfigをアタッチします。なお追加でConfigをアタッチしたい場合にはappend-configを使います。

```jsx
> .\amazon-cloudwatch-agent-ctl.ps1 -a fetch-config -c file:config.json
****** processing amazon-cloudwatch-agent ******
I! Trying to detect region from ec2
D! [EC2] Found active network interface
I! imds retry client will retry 1 timesSuccessfully fetched the config and saved in C:\ProgramData\Amazon\AmazonCloudWatchAgent\Configs\file_config.json.tmp
Start configuration validation...
2024/02/27 23:20:43 Reading json config file path: C:\ProgramData\Amazon\AmazonCloudWatchAgent\Configs\file_config.json.tmp ...
2024/02/27 23:20:43 I! Valid Json input schema.
I! Trying to detect region from ec2
D! [EC2] Found active network interface
I! imds retry client will retry 1 times2024/02/27 23:20:43 Configuration validation first phase succeeded
Configuration validation second phase succeeded
Configuration validation succeeded
```

# CloudWatchAgentの実行

---

CloudWatchAgentを実行します。

```bash
> .\amazon-cloudwatch-agent-ctl.ps1 -a status
{
  "status": "stopped",
  "starttime": "",
  "configstatus": "configured",
  "version": "1.300033.0b462"
}

> .\amazon-cloudwatch-agent-ctl.ps1 -a start

****** Processing amazon-cloudwatch-agent ******
AmazonCloudWatchAgent has been started

> .\amazon-cloudwatch-agent-ctl.ps1 -a status
{
  "status": "running",
  "starttime": "2024-02-27T23:22:44",
  "configstatus": "configured",
  "version": "1.300033.0b462"
}
```

実行できたらCloudWatchLogsの画面に行き作成したログがCloudWatchLogsに流れていることを確認します。

ログが流れていない場合には以下にCloudWatchAgentのログがあるのでデバックの参考にしてください。

```bash
C:\ProgramData\Amazon\AmazonCloudWatchAgent\Logsvamazon-cloudwatch-agent.log
```