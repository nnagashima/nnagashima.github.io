[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# WindowsのイベントログをS3に転送

![](/AWS/WindowsのイベントログをS3に転送/icon.svg)

# 目次
- [WindowsのイベントログをS3に転送](#windowsのイベントログをs3に転送)
- [目次](#目次)
- [はじめに](#はじめに)
- [全体の流れ](#全体の流れ)
    - [前提の補足](#前提の補足)
- [作ったPowerShellのスクリプト](#作ったpowershellのスクリプト)

---

# はじめに

---

元々AWS CLIやAWS SDKを使ってプログラムを書いていたので、 
Windowsのイベントログを月に1回 S3に保管して管理できないかと思ってやってみた結果を記載

# 全体の流れ

---

1. AWS上にWindowsサーバを作成して、IAMのポリシーにてS3とSNSの適切な権限のロールを作成してWindwsサーバにIAMロールを割り当てています。 
2. S3に転送するにあたってS3のゲートウェイエンドポイントを作成しています。
3. S3バケットを作成します。
4. 作成したWinodwsサーバには、AWS CLIをインストールしています。インストール手順については[こちら](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-windows.html#cliv2-windows-install)を参考にして下さい。
5. S3にイベントログを保存できなかった時にエラー処理として、SNSで通知する仕様にしております。
6. SNSを利用するにあたってSNSのVPCエンドポイントとVPCエンドポイント用のセキュリティグループ（インバウンドHTTPS）を作成しています。

### 前提の補足

EC2がS3にファイルをアップロードするためのIAMポリシー

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
            "Resource": "arn:aws:s3:::S3バケット名"
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
            "Resource": "arn:aws:s3:::S3バケット名/eventlog/*"
        }
    ]
}
```

EC2がSNSトピックを利用して発報するためのIAMポリシー

```jsx
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sns:Publish",
            "Resource": "arn:aws:sns:ap-northeast-1:AWSアカウントID:SNSTOPIC名"
        }
    ]
}
```

# 作ったPowerShellのスクリプト

---

特に何か記載することはなく、各プログラムの動作を書いているので、そこまで難しくないと思います。

```bash
# 保存先のバケット名を記載
$BucketName = "********"

# 取得対象のイベントログを指定
$LogNames = @("********")

# SNS送信先のARNを指定
$Arn = "********"

# 先月時点のyyyyMM文字列を取得し作成するファイル名につける
$YYYYMM = (Get-Date).AddMonths(-1).ToString("yyyyMM")

# 出力先として「C:\temp」フォルダを指定
$DstFolder = "c:\temp\"

# イベントログを取得する対象の期間を先月1日から末日までの1ヶ月間と指定
$StartJTime = (Get-Date -Day 1 -hour 0 -minute 0 -second 0).AddMonths(-1)
$EndJTime = (Get-Date -Day 1 -hour 0 -minute 0 -second 0)

# イベントログを取得する際の日付をUTCに変換（UTCに変換しないとフィルタリング条件として使えない）
$StartUtcTime = [System.TimeZoneInfo]::ConvertTimeToUtc($StartJTime).ToString("yyyy-MM-ddTHH:mm:ssZ")
$EndUtcTime = [System.TimeZoneInfo]::ConvertTimeToUtc($EndJTime).ToString("yyyy-MM-ddTHH:mm:ssZ")

# 日付指定でフィルタリング条件作成
$Filter = @"
  Event/System/TimeCreated[@SystemTime>='$StartUtcTime'] and
  Event/System/TimeCreated[@SystemTime<'$EndUtcTime']
"@

# イベントログ出力用のオブジェクトを作成
$EvSession = New-Object -TypeName System.Diagnostics.Eventing.Reader.EventLogSession

# イベントログをevtx形式で出力
foreach($LogName in $LogNames){
  $OutFile = $DstFolder + "\" + $Env:COMPUTERNAME + "_" + $LogName + "_" + $YYYYMM + ".csv"
  $Locale = [System.Globalization.CultureInfo]::CreateSpecificCulture("ja-JP")
  $EvSession.ExportLogAndMessages($LogName,"LogName",$Filter,$OutFile,$True,$Locale)
}

# エクスポートしたイベントログファイルを圧縮
$ZipFile = $OutFile + ".zip"
Compress-Archive -Path $OutFile -DestinationPath $ZipFile

# S3バケットに保存
aws s3 cp $ZipFile s3://$BucketName/

# AWS CLIのステータスコードを取得して0以外のステータスの場合にSNSメッセージを送信
# リターンコードの詳細：https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-usage-returncodes.html
if ($lastexitcode -ne 0){
    aws sns publish --topic-arn $Arn --message "Windows EventLog S3 Upload Fail"
}Else{
    #ファイルを削除
    Remove-Item $OutFile
    Remove-Item $ZipFile

    # エクスポートした時のMetaFileが作成されるので削除（-Recurse -Forceで、rmの-rfと同じ機能となる）
    Remove-Item -Path C:\temp\LocaleMetaData -Recurse -Force
}
```

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
