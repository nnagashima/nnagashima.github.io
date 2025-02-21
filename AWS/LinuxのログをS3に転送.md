[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# LinuxのログをS3に転送

# 目次
- [LinuxのログをS3に転送](#linuxのログをs3に転送)
- [目次](#目次)
- [はじめに](#はじめに)
- [EC2に適用するIAMロールとポリシーの作成](#ec2に適用するiamロールとポリシーの作成)
- [EC2へAWS CLIをインストール](#ec2へaws-cliをインストール)
- [EC2のログをローテションしつつ、S3へアップロード](#ec2のログをローテションしつつs3へアップロード)

---

# はじめに

---

Linuxのログをローテーションして、ログをS3に退避する方法を記載します。

OSはRHELをベースに記載しています。

# EC2に適用するIAMロールとポリシーの作成

---

以下のようにIAMポリシーを作成して、EC2用のロールを作成しポリシーをアタッチします。

```jsx
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutBucketLogging",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:GetObjectVersionTorrent",
                "s3:PutObject",
                "s3:GetObjectAcl",
                "s3:GetObject",
                "s3:GetObjectVersionTagging",
                "s3:GetObjectVersionAcl",
                "s3:PutObjectVersionTagging",
                "s3:GetObjectTagging",
                "s3:PutObjectTagging",
                "s3:GetObjectVersionForReplication"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        }
    ]
}
```

# EC2へAWS CLIをインストール

---

以下ドキュメントに従い、インストールを実施します。

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html

# EC2のログをローテションしつつ、S3へアップロード

---

例として/var/logにある拡張子のlogとつく名前のファイルをローテションします。

保存したいファイルの日付を増やしたい場合にはrotateの日数を伸ばしてください。

```jsx
/var/log/*.log {
    dateext
    missingok
    notifempty
    daily
    rotate 30
    sharedscripts
    compress
    su root root
    lastaction
        find /var/log/ -name "*log-`date +\"%Y%m%d.gz\"`" |xargs -L 1 -I % aws --region=ap-northeast-1 s3 cp % s3://【バケット名】/`curl http://169.254.169.254/latest/meta-data/instance-id/`/【ログファイル名】/
    endscript
} 
```

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
