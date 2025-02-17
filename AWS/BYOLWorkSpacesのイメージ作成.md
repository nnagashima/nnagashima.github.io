[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# BYOL WorkSpacesのイメージ作成

![](/AWS/BYOLWorkSpacesのイメージ作成/icon.png)

# 目次
- [BYOL WorkSpacesのイメージ作成](#byol-workspacesのイメージ作成)
- [目次](#目次)
- [はじめに](#はじめに)
- [仮想マシンの作成](#仮想マシンの作成)
- [WindowsOSのインストール](#windowsosのインストール)
- [OSインストール後の作業](#osインストール後の作業)
- [BYOL Checkerの実施](#byol-checkerの実施)
- [OVFからOVA形式へ変換](#ovfからova形式へ変換)
- [S3へイメージをアップロード](#s3へイメージをアップロード)
- [S3へアップロードしたイメージをVMImportするためのIAMロールとポリシー作成](#s3へアップロードしたイメージをvmimportするためのiamロールとポリシー作成)
- [OVAイメージをAMIに変換](#ovaイメージをamiに変換)
- [最後に](#最後に)

---

# はじめに

---

AWS WorkSpacesでのBYOLを実施するにあたってイメージの作成が必要になります。

イメージ作成までにはなりますが概要をまとめましたので記載します。

Windows10のISOは以下が対応しているようです。詳細要件については以下URLを参照ください。（2023年09月時点）

- Windows 10 バージョン 21H2 (2021 年 12 月更新)
- Windows 10 バージョン 22H2 (2022 年 11 月更新)
- Windows 11 バージョン 22H2 (2022 年 10 月リリース)

https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/byol-windows-images.html

# 仮想マシンの作成

---

今回私はイメージの作成にESXi8を利用しております。新規仮想マシンの作成を実施します。

WindowsのOSイメージを作成するにあたり、言語はEnglishである必要があるのでISOダウンロード時には注意してください。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled.png)

イメージの作成はWindows10で実施します。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled1.png)

仮想マシンのストレージを選択します。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled2.png)

BYOL CheckerではDISKは80GB以下にする必要があるので80GBで作成していますが、

あとから調べると「VM は、最大サイズが 70 GB、空き容量が 10 GB 以上の 1 つのボリューム上にあることが必要です。

また、BYOL イメージの Microsoft Office へのサブスクライブを計画している場合は、

VM は最大サイズが 70 GB で、20 GB 以上の空き容量を持つ 1 つのボリューム上に存在する必要があります。

ルートボリュームがある DISK は 70 GB を超えることはできません。」

と記載があるので70GBで作成するのがよさそうです。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled3.png)

I/Oコントローラタイプの選択では、デフォルト（LSI Logic SAS）にします。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled4.png)

データストアのISOイメージは事前にダウンロードしたISOをマウントしておきます。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled5.png)

WorkSpacesの要件である「1つのシングルボリュームの仮想マシン」を作る必要があるため、

仮想マシンのオプションでボリュームが分割されないようにファームウェアタイプをBIOSにします。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled6.png)

# WindowsOSのインストール

---

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled7.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled8.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled9.png)

今回はWindows10Proで実施しています。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled10.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled11.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled12.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled13.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled14.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled15.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled16.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled17.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled18.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled19.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled20.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled21.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled22.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled23.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled24.png)

# OSインストール後の作業

---

- ログインしてKMS認証を実施
- Administrator有効化とパスワード設定（https://www.partitionwizard.jp/partitionmanager/enable-administrator-account.html）
- Workspaces_BYOLアカウントの作成(Administratorsグループへの所属) とパスワードの設定
- BYOLユーザーでログインして、BYOLアカウント以外のユーザープロファイル削除

```bash
Get-WmiObject win32_userprofile | where { $_.Sid -match "S-1-5-¥d{2}-" -and $_.LocalPath -notmatch "$env:USERNAME`$" }
```

- スタート＞設定＞システム＞詳細情報＞システムの詳細設定をクリックし、システムのプロパティウインドウの詳細設定タブから、
    
    ユーザプロファイルの「設定」をクリックし、「既定のプロファイル」と「WorkSpaces_BYOL」以外のユーザプロファイルがある場合には削除
    
- Winodowsへの自動ログオンの無効化

```bash
netplwizを起動して、ユーザーがこのコンピューターを利用する際、ユーザー名とパスワードの記入が必須にする
```

- RealTimeIsUniversalレジストリキーの追加

```bash
reg add "HKEY_LOCAL_MACHINE¥System¥CurrentControlSet¥Control¥TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

- IPv6無効化（IPアドレスは静的を使用しない）
- RDP有効化（ネットワークレベル認証未設定）
- FW無効化
- WindowsUpdate実施、実施後WindowsUpdateサービス無効化
- VMtoolsをインストールします
- Modern Appx Packageを削除します

```bash
> set-executionpolicy unrestricted
  Do you want to change the execution policy? [A] Yes to All
> A ←入力
> Get-AppxPackage -AllUsers | Remove-AppxPackage
```

- VMtoolsをアンインストールします
- CDDriveからISOイメージを外す

全部実施したらOS再起動をします。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled25.png)

# BYOL Checkerの実施

---

- [https://tools.amazonworkspaces.com/BYOLChecker.zip](https://tools.amazonworkspaces.com/BYOLChecker.zip)をダウンロードして解凍しておきます。
- PowerShellを管理者モードで起動し、BYOL Checkerを解凍したディレクトリに移動し以下コマンドを実施します。
    
    ```bash
    Set-ExecutionPolicy Unrestricted
    ./BYOLChecker.ps1
    ```
    
- BYOL Chckerの画面がでたらBegin Testsを実施して結果を確認します。

以下BYOL Checkerの画面となります。Warning項目はFix All Warningsで解消可能となります。

![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled26.png)
![](/AWS/BYOLWorkSpacesのイメージ作成/Untitled27.png)

# OVFからOVA形式へ変換

---

仮想マシン停止後、VMwareのコンソールから対象の仮想マシンをOVF形式でエクスポートします。

その後、OVFToolを利用してOVF形式からOVA形式にイメージ変換します。

※OVFToolのダウンロードはVMwareアカウントが必要になります。

以下、MACでの実行例です。

```bash
% cd /Applications
% cd ./VMware\ OVF\ Tool
% ./ovftool /Users/username/Downloads/image/windows10-22H2-2.ovf /Users/username/Downloads/windows10-22H2-2.ova
Opening OVF source: /Users/username/Downloads/image/windows10-22H2-2.ovf
The manifest validates
Warning:
 - Line -1: Unsupported value 'cpuid.coresPerSocket' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'tools.guest.desktop.autolock' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'svga.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge0.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge4.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge4.virtualDev' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge4.functions' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge5.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge5.virtualDev' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge5.functions' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge6.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge6.virtualDev' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge6.functions' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge7.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge7.virtualDev' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge7.functions' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'hpet0.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'RemoteDisplay.maxConnections' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'sched.cpu.latencySensitivity' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'disk.EnableUUID' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmxstats.filename' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'numa.autosize.cookie' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'numa.autosize.vcpu.maxPerVirtualNode' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge0.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge4.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge5.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge6.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'pciBridge7.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'scsi0.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'ethernet0.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'usb_xhci.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'sata0.pciSlotNumber' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'scsi0.sasWWID' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmotion.checkpointFBSize' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmotion.checkpointSVGAPrimarySize' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmotion.svga.mobMaxSize' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmotion.svga.graphicsMemoryKB' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'ethernet0.generatedAddressOffset' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'monitor.phys_bits_used' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'softPowerOff' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'toolsInstallManager.lastInstallError' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'svga.guestBackedPrimaryAware' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'tools.remindInstall' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'toolsInstallManager.updateCounter' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'guestInfo.detailed.data' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'sata0:0.autodetect' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'usb_xhci:4.present' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'usb_xhci:4.deviceType' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'usb_xhci:4.port' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'usb_xhci:4.parent' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmware.tools.internalversion' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'vmware.tools.requiredversion' for attribute 'key' on element 'ExtraConfig'.
 - Line -1: Unsupported value 'migrate.hostLog' for attribute 'key' on element 'ExtraConfig'.
Opening OVA target: /Users/username/Downloads/windows10-22H2-2.ova
Writing OVA package: /Users/username/Downloads/windows10-22H2-2.ova
Transfer Completed                    
Warning:
 - No supported manifest(sha1, sha256, sha512) entry found for: 'windows10-22H2-2.ovf'.
 - File is missing from the manifest: 'windows10-22H2-2.ovf'.
 - Wrong file size specified in OVF descriptor for 'windows10-22H2-2-0.vmdk' (specified: 0, actual 17790739968).
 - Wrong file size specified in OVF descriptor for 'windows10-22H2-2.nvram' (specified: 0, actual 8684).
Completed successfully
```

# S3へイメージをアップロード

---

AWSマネジメントコンソールにてWorkSpacesを起動するリージョンでS3バケットを作成して、作成したイメージをアップロードします。

# S3へアップロードしたイメージをVMImportするためのIAMロールとポリシー作成

---

ここからはAWS CLIを利用します。

1. AWS CLIを起動してVMImport用のポリシーを作成するためのファイルを作成します。

```bash
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}
```

1. AWS CLIにて上記で作成したファイルを保存したフォルダに移動してVMImport用のロールを作成します。

```bash
$ aws iam create-role　--role-name vmimport　--assume-role-policy-document file://ファイル名.json
```

1. AWS CLIにてロールにアタッチする権限を記載したポリシーを作成します。（OVAイメージが置いているバケットを指定）

```bash
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket" 
         ],
         "Resource": [
            "arn:aws:s3:::S3バケット名",
            "arn:aws:s3:::S3バケット名/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket",
            "s3:PutObject",
            "s3:GetBucketAcl"
         ],
         "Resource": [
            "arn:aws:s3:::S3バケット名",
            "arn:aws:s3:::S3バケット名/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      }
   ]
}
```

1. AWS CLIにて、1で作成したファイルを保存したフォルダに移動して、2で作成したロールに3で指定した権限を付与します。

```bash
$ aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://ファイル名.json
```

# OVAイメージをAMIに変換

---

OVAイメージをVMImportでAMI化するためにファイルを作成してAWS CLIで保存

```bash
{
  "Description": "Windows10 BYOL Pro",
  "DiskContainers": [
    {
      "Description": "Windows10 Root volume",
      "Format": "ova",
      "UserBucket": {
        "S3Bucket": "S3バケット名",
        "S3Key": "ファイル名.ova"
      }
    }
  ],
  "LicenseType": "BYOL"
}
```

1. 1で配置したフォルダに移動してVMImportを実行します。

```bash
$ aws ec2 import-image --cli-input-json file://ファイル名.json --region ap-northeast-1

>タスク進捗状況の確認
$ aws ec2 describe-import-image-tasks --import-task-ids import-ami-000000000000000000　←タスクIDを指定してインポート進捗状況を確認します。
```

1. EC2サービスのAMIにイメージが表示されていることを確認し、WorkSpacesサービスのイメージからイメージの作成にてWorkSpacesイメージを作成します。

（BYOL申請が通っていれば表示されます）

# 最後に

---

色々と制約が多いので気をつけなければならない点がありますが、要点を押さえておけばそこまで難しいことはないかもしれません。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
