# LinuxのApacheログをS3に転送(fluentd編)

![amazon-s3-simple-storage-service.svg](Windows%E3%81%AE%E3%82%A4%E3%83%98%E3%82%99%E3%83%B3%E3%83%88%E3%83%AD%E3%82%AF%E3%82%99%E3%82%92S3%E3%81%AB%E8%BB%A2%E9%80%81%20120dbcb1ccbb4f4991cc69233f4e7d83/amazon-s3-simple-storage-service.svg)

# 目次

# はじめに

---

Linuxのログをfluentdを利用して、ログをS3に退避させます。

OSはAmazonLinux2023をベースに記載しています。

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
                "s3:ListAllMyBuckets",
                "s3:PutBucketLogging",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::バケット名"
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
            "Resource": "arn:aws:s3:::バケット名/*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:Describe*",
            "Resource": "*"
        }
    ]
}
```

# fluentdのインストール

---

今回はAmazonLinux2023になるのでAamzonLinux2023の内容でインストールを実施しています。

```jsx
# curl -fsSL https://toolbelt.treasuredata.com/sh/install-amazon2023-fluent-package5-lts.sh | sh
```

# fluentdの実行ユーザーを変更

---

デフォルトの起動ユーザーがtd-agentとなってしまうためログファイルごとに毎回権限変更が必要になるため、

起動ユーザーをrootで起動するようにserviceファイルを修正します。

```jsx
# cp -p /usr/lib/systemd/system/fluentd.service /user/lib/systemd/system/fluentd.service.org
# vi /usr/lib/systemd/system/fluentd.service
# diff -u /usr/lib/systemd/system/td-agent.service.org /usr/lib/systemd/system/fluentd.service
[Service]
-User=fluentd
-Group=fluentd
+User=root
+Group=root
# systemctl daemon-reload
```

# fluentdの追加プラグインをインストール

---

S3プラグインは同梱されていますが、metadataを取り扱うプラグインは入っていないので追加でインストールします。

```jsx
# yum install ruby ruby-devel
# gem update --system 3.5.7
# gem install fluent-plugin-ec2-metadata
```

# fluentdの設定ファイルを作成

---

## S3へ保存するログファイルの設定ファイル作成

---

fluentdを利用してS3へ保存したログファイルの設定ファイルを作成します。

設定ファイルは/etc/fluent/conf.d配下に作ります。

### Apacheのログファイルの場合

---

```jsx
<source>
    @type tail
    path /var/log/httpd/access_log
    pos_file /var/log/td-agent/pos/access_log.pos
    tag log.httpd.access_log
    format apache2
    <parse>
        @type none
    </parse>
</source>
```

## S3へ保存する設定ファイル作成

---

```jsx
#######################
# Input Plugin
#######################
## OS logs (messages)
@include /etc/fluent/conf.d/apache.conf

#######################
# EC2 metadata
#######################
<match log.**>
    @type ec2_metadata
    @label @S3
    output_tag ${tagset_name}.${instance_id}.${tag}
</match>

#######################
# Output Plugin
#######################
<label @S3>
    <match **>
        @type s3
	      s3_bucket 【バケット名】
        s3_region ap-northeast-1
        path ${tag[1]}/%Y/%m/%d
        time_slice_format %Y%m%d_%H%M%S
        s3_object_key_format %{path}/${tag[3]}-${tag[4]}-%{time_slice}-%{index}.%{file_extension}
        <format>
            @type single_value
        </format>
        <buffer tag,time>
            @type file
            path /var/log/td-agent/buffer/s3
            timekey 5m #ログ保存を実行する間隔
            timekey_wait 1m
            timekey_use_utc false
            timekey_zone Asia/Tokyo
            chunk_limit_size 1g
            flush_at_shutdown true
        </buffer>
    </match>
</label>
```

# それぞれのファイルの設定ファイルチェック

---

それぞれ[info]: finished dry run modeと表示され、エラーがでないことを確認します。

```jsx
# fluentd --dry-run -c /etc/fluent/conf.d/os.conf
# fluentd --dry-run -c /etc/fluent/conf.d/apache.conf
# fluentd --dry-run -c /etc/fluent/fluent.conf
```

# Fluentdの起動とエラーが出ていないか確認

---

```jsx
# systemctl enable fluentd
# systemctl restart fluentd
# systemctl status fluentd
# less /var/log/fluent/fluent.log
```

# S3バケットにログが保管されていることを確認

---

以下のようにログが保管されていることを確認します。

![FireShot Capture 992 - nnagashima-s3-ec2log - S3 バケット - S3 - Global - s3.console.aws.amazon.com.png](Linux%E3%81%AEApache%E3%83%AD%E3%82%AF%E3%82%99%E3%82%92S3%E3%81%AB%E8%BB%A2%E9%80%81(fluentd%E7%B7%A8)%20b47c606a57434f7f861a862c88819c15/FireShot_Capture_992_-_nnagashima-s3-ec2log_-_S3_%25E3%2583%258F%25E3%2582%2599%25E3%2582%25B1%25E3%2583%2583%25E3%2583%2588_-_S3_-_Global_-_s3.console.aws.amazon.com.png)

こんな感じでログをリアルタイムでも1日に1回という形でもCloudWatchLogsに転送しなくてもログをリアルタイム保管することができます。