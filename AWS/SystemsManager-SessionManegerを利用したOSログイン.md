[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# Systems ManagerのSession Manegerを利用したOSログイン

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/AWS-Systems-Manager.png)

# 目次
- [Systems ManagerのSession Manegerを利用したOSログイン](#systems-managerのsession-manegerを利用したosログイン)
- [目次](#目次)
- [はじめに](#はじめに)
- [AWS Systems Manager とは](#aws-systems-manager-とは)
- [AWS Systems Managerの利用方法](#aws-systems-managerの利用方法)
- [AWS Systems Manager Session Manger利用のための準備（PublicSubnetの場合）](#aws-systems-manager-session-manger利用のための準備publicsubnetの場合)
- [AWS Systems Manager Session Manger利用のための準備（PrivateSubnetの場合）](#aws-systems-manager-session-manger利用のための準備privatesubnetの場合)
- [AWS Systems Manager Session Manger利用](#aws-systems-manager-session-manger利用)
- [たまにある質問の補足](#たまにある質問の補足)
- [Linuxのファイル転送の場合](#linuxのファイル転送の場合)
- [RDP接続の場合](#rdp接続の場合)

---

# はじめに

---

AWS Systems Manager Session Mangerについて公式ドキュメント見れば分かりますが、

今まで調べてきたものを忘れないようにするために自分の備忘録として記載します。

# AWS Systems Manager とは

---

AWSの構成管理ツールのサービスでAWS Systems Managerを利用するために

SSM AgentをEC2インスタンスへインストールする必要があります。 

尚、一部のOSには既にデフォルトでインストールされているのでインストールする必要はありません。 

[Linux 用 EC2 インスタンスに SSM Agent を手動でインストールおよびアンインストールする](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/sysman-manual-agent-install.html)

[Windows Server 用 EC2 インスタンスで SSM Agent を使用する](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/sysman-install-ssm-win.html)

またオンプレミス環境にもインストールできるようです。

[Systems Manager を利用したハイブリッドおよびマルチクラウド環境でのサーバーの管理](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/systems-manager-hybrid-multicloud.html)


# AWS Systems Managerの利用方法

---

AWS Systems Managerが使えるようにするためには以下を実施します。

- SSM Agentのインストール
- SSM AgentからSSM APIへの経路
- IAMロールの付与

SSM Agentが導入されていないものについては上記にリンクを貼っているのでそちらを参考にしてください。

SSM Agentはインターネット経由もしくはVPCエンドポイント経由でアウトバンドでの経路が必要になります。 

インターネット経由でやる場合にはパブリックサブネット上に配置するかNATゲートウェイを経由します。 

VPCエンドポイント経由であればプライベートに配置されていてもアクセスが可能になります。

IAMロール作成してEC2に適用する必要がありIAMポリシーについては、AmazonSSMManagedInstanceCoreを付与する必要があります。

AWS Systems Manager Session Mangerとは

Session Mangerを利用することでEC2へのアクセスのためにインバウンドポート解放や踏み台サーバの用意などすることなく、 

ブラウザからShellを操作することができ、かつSSHでアクセスからIAM経由でのアクセスとなり操作履歴も残すことができるようになります。

なお利用するにあたりVPCの「DNS ホスト名とDNS 解決は有効」になっていることを確認してください。

[AWS Systems Manager と IAM の連携方法](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/security_iam_service-with-iam.html)

# AWS Systems Manager Session Manger利用のための準備（PublicSubnetの場合）

---

先ほど述べたように3ステップ踏む必要がありますが、 今回はデフォルトでインストールされているAmazonLinux2を利用し、

パブリックネットワークにおいてます。 またEC2を作成する前に事前にIAMロールを用意しておくほうが良いと思うので

IAM画面からロール=>ロール作成へと進み、 以下の画面のように選択して次へ進みます。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421233838.png)

次に許可ポリシーを選択する画面がでてくるので検索画面にて、

 AmazonSSMManagedInstanceCoreと入力してEnterを押すと、

対象のポリシーが表示されるのでチェックを入れて次へ進みます。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421234010.png)

最後にロール名と必要に応じてタグを付与してロール作成をします。

 EC2を作成する際には今回作成したIAMロールを選択忘れしないようにロールを適用してください。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421234329.png)

またSessionManger経由でアクセスのみとするのであればセキュリティグループのInboundの解放やキーペアの作成も不要になるので、

不要な場合は作成しないようにしましょう。

尚、本内容を実施する際に、Outboudを制限かけている場合は、443ポートが外にでられるようにSecurityGroupを設定してください。

# AWS Systems Manager Session Manger利用のための準備（PrivateSubnetの場合）

---

ロール作成からEC2への適用まではPublicSubnetのやり方と同じです。

今回の場合はPublicSubnetにNAT Gatewayがない場合で記載をしています。

PublicSubnetにNAT Gatewayがある場合にはPublicSubnetと同様のやり方で問題ありません。

PrivateSubnetの場合にはインターネットにでることができないので、SSM Endpointを作る必要があります。

作成するエンドポイントは以下になります。

- ssmmessages
- ssm
- ec2messages

VPCエンドポイントを作成する上で気をつけなければならないことは、紐づけるサブネットが多くなった場合にはその分料金がかかるので、

エンドポイントを冗長化するかシングルにするかはお金と要相談してください。

エンドポイントの作成場所はVPC画面メニューのエンドポイントから作成できます。

VPCエンドポイントを作成したらエンドポイントインターフェースに対して、443ポートのInbound許可のSecurityGroupを作成して適用します。

# AWS Systems Manager Session Manger利用

---

AWS Systems Managerから左メニューのセッションマネージャーをクリックすると以下の画面が表示されるので、 

セッションの開始をクリックします。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421234449.png)

接続するターゲットインスタンスの一覧が表示されるので、接続する対象を選択しセッションの開始をクリックすると、

ブラウザShellが起動します。LinuxならShell、WindowsならPowerShellになります。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421234643.png)

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421235045.png)

また事前に実行ログなどをCloudWatchLogやS3にも保存しておくことができます。

![](/AWS/SystemsManager-SessionManegerを利用したOSログイン/20220421235243.png)

# たまにある質問の補足

---

ブラウザ経由でアクセスできるのは便利なんだけどLinuxのファイル転送やRDP接続してGUI操作したい時はどうすればいいの？ 

とたまに聞かれることがありますので補足します。

# Linuxのファイル転送の場合

---

接続元にAWS CLIとSession ManagerのPluginをインストールします。 

※ここからはMACで実施しています。 

[Install the Session Manager plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

その後SSHするための設定を.ssh/configに追記します。

```bash
# SSH over Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

IAMに必要なポリシーは以下を参考にしました。 

[ステップ 8: (オプション) Session Manager を通して SSH 接続のアクセス許可を付与および制御する](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html)

`"ssm:SessionDocumentAccessCheck":"true"` を指定することで、`Resource` で指定したすべての条件が存在しないとエラーになります。

つまり、Document の AWS-StartSSHSessionやAWS-StartPortForwardingSessionを指定して

 `start-session` コマンドを実行しないとダメということになり、特定の Document のみが実行できる環境が出来上がります。

document/***は SystemsMangerのドキュメントで管理されており、AWS-StartSSHSessionとAWS-StartPortForwardingSessionはAmazon所有となっています。

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:StartSession",
                "ssm:SendCommand"
            ],
            "Resource": [
                "arn:aws:ec2:ap-northeast-1:AWSアカウントID:instance/*",
                "arn:aws:ssm:ap-northeast-1::document/AWS-StartPortForwardingSession",
                "arn:aws:ssm:ap-northeast-1:AWSアカウントID:document/SSM-SessionManagerRunShell",
                "arn:aws:ssm:ap-northeast-1::document/AWS-StartSSHSession"
            ],
            "Condition": {
                "BoolIfExists": {
                    "ssm:SessionDocumentAccessCheck": "ture"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:DescribeSessions",
                "ssm:GetConnectionStatus",
                "ssm:DescribeInstanceInformation",
                "ssm:DescribeInstanceProperties",
                "ec2:DescribeInstances",
                "ssm-guiconnect:StartConnection",
                "ssm-guiconnect:GetConnection",
                "ssm-guiconnect:CancelConnection"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:TerminateSession",
                "ssm:ResumeSession"
            ],
            "Resource": [
                "arn:aws:ssm:ap-northeast-1:AWSアカウントID:session/${aws:username}-*"
            ]
        }
    ]
}
```

接続元を絞るためにGIP制限もかけました。

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Condition": {
                "NotIpAddress": {
                    "aws:SourceIp": [
                        "GlobalIPAddress/32",
                        "GlobalIPAddress/32"
                    ]
                }
            },
            "Action": "*",
            "Resource": "*",
            "Effect": "Deny"
        }
    ]
}
```

作成したポリシーをアタッチするためにIAMグループを作成し、IAMユーザを作成してグループに所属させます。

アクセスキーとシークレットキーを発行して、 [aws configure --profile profilename]を実行して認証情報をクライアント側に登録しておきます。

最後にクライアント側からSSH接続すると接続できるので、同じ要領でSCPコマンドでファイル転送することが可能になります。

```bash
% ssh ec2-user@インスタンスID -i ./鍵ファイルの名前
Last login: Thu Apr 21 15:33:00 2022 from localhost

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
16 package(s) needed for security, out of 25 available
Run "sudo yum update" to apply all updates.
[ec2-user@xxx-xxx-xxx-xxx ~]$

% scp -i <秘密鍵> <ローカルファイル> ec2-user@<インスタンスID>:＜転送先のディレクトリ>
```

# RDP接続の場合

---

接続元にAWS CLIとSession ManagerのPluginをインストールします。 

IAMに必要なポリシーはLinuxのファイル転送の場合と同じポリシーを利用します。

先ほどは記載していませんがssm:TerminateSession では自分自身のsessionだけを対象とするため、${aws:username}-* とし、

IPでの制限をかけたい場合には絞りたいIPアドレスにIPアドレスを記載することで制限をつけることもできます。

アクセスキーとシークレットキーを発行して、 aws configureを実行して認証情報をクライアント側に登録しておきます。

最後にクライアント側からAWS CLIを使って接続することでRDP接続が可能になります。

尚、RDPで接続している間は、以下コマンドを実行したウィンドウは閉じないようにしてください。

```bash
aws ssm start-session --target EC2のインスタンスID --document-name AWS-StartPortForwardingSession --parameters "portNumber=3389, localPortNumber=13389"
```

ここではローカルポート番号を13389にしているので、RDP接続する際にはlocalhost:13389で接続します。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
