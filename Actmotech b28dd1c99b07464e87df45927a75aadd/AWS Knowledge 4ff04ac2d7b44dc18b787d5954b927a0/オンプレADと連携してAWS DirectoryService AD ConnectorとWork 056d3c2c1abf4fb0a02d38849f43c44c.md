# オンプレADと連携してAWS DirectoryService AD ConnectorとWorkSpacesを作成

![2022-12-12_18h13_37.png](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/2022-12-12_18h13_37.png)

# 目次

---

# はじめに

---

自宅にあるADサーバとAWSに用意したWindowsのEC2インスタンスやWorkSpacesを参加させる手段として

AWSのAD Connectorを利用して、Domain参加させる方法を記載します。

オンプレミスのAD作成は以下を参考にしてください。

[Onpremiss Active Directoryを利用してAWS ADConnectorと接続](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2.md)

# 前提

---

- オンプレミスにADサーバが用意されている
- オンプレミス環境とAWS間をSite to Site VPN通信もしくはDX通信が確立している
- AD Connectorが接続するため準備が整っている（AD Connectorを作成するためのサブネットをAWS側に作成することも忘れずに用意してください）

https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/prereq_connector.html#connect_delegate_privileges

- ADやRouter側でFWで制御している場合にはAD通信に必要なポート解放がされていること
    
    AD Connectorが接続するには、[ここのサイト](https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/prereq_connector.html)にも記載がありますが以下の通りです。
    
    - TCP/UDP 53 - DNS
    - TCP/UDP 88 - Kerberos 認証
    - TCP/UDP 389 - LDAP
    
    ですが最終的にWorkSpacesやEC2をドメイン参加させたい場合に必要なのは[ここのサイト](https://social.technet.microsoft.com/Forums/ja-JP/f6126a91-6d96-41d2-ab2a-0b75bf03fdd8/12489125131245212531298722265912391203512999212373124281242712?forum=Wcsupportja)にも記載がありますが以下の通りとなりますので、
    
    制限かけたい場合には以下を参考にしてみてください。　※図は引用させて頂いております。
    

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled.png)

# AD Connectorのデプロイ

---

Directory ServiceからAD Connectorを選択します。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%201.png)

ディレクトリのサイズを選択します。

ディレクトリは一度作成した後に後からサイズ変更ができないので注意してください。

スモールはAD Connectorのユーザー数を500まで、ラージはユーザー数を5000を推奨としています。

※調べてみるとサポートケースにリクエストをすることでアップグレードもできる事例があったので、もしかしたらできるかもしれません。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%202.png)

次にAD Connectorをデプロイするサブネットを選択します。サブネットはAZ毎に一つずつ選択します。

ここを間違えると再度AD Connectorを再作成、もしくはWorkSpacesまで作成していたらWorkSpacesの再作成になるので要注意です。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%203.png)

ADに接続するための情報を登録します。

- ディレクトリのDNS名にはオンプレミスで作成したADのドメイン名
- ディレクトリのNetBIOS名はオンプレミスで作成したADのNetBIOS名
- DNS IPアドレスはオンプレミスで作成したADのIPアドレス
- サービスアカウントのユーザー名とパスワードはAD ConnectorがADに接続するためのユーザー情報

※接続に利用するユーザーは管理者権限をつけないとドメイン参加できる回数に制限がかかってしまい、ドメイン参加できなくなるので注意してください。

https://repost.aws/ja/knowledge-center/workspaces-domain-join-error

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%204.png)

最後に登録した情報の確認が出てくるので問題なければディレクトリの作成をクリックします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%205.png)

ディレクトリのステータスがアクティブになるまでしばし待ちます。

エラーが出た場合にはFWやオンプレミスとAWS間の通信、オンプレミスで作成したADのAD Connectorが接続するユーザーに間違いがないかなど確認をしてください。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%206.png)

作成ができたらディレクトリの詳細を開きます。ここからはAD Connectorを使ってWorkSpacesを利用したのでWorkSpacesをクリックします。

![Untitled.png](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%207.png)

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%208.png)

WorkSpacesの画面ができたら、ディレクトリの登録を行います。

対象のディレクトリを選択して登録をクリックします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%209.png)

WorkSpacesをデプロイする先のサブネットを選択します。

なお、AD ConnectorとWorkSpacesがお互い通信ができるような形にしてください。

一般的な構成としてはAD Connectorをデプロイした先と同じ場所にWorkSpacesをデプロイする構成が一般的のようです。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2010.png)

セルフサービス許可を有効化は有効化にしてください。

機能を有効することによって、ユーザー自身がWorkSpaceを再起動したり、スペック変更や再構築もできるというものです。

WorkDocsを利用する場合は有効化してください。

後から変更したい場合はCloudShellを使うことで変更できます。コマンドは以下の通りです。

以下コマンドを実行後、WorkDocsの画面で対象ディレクトリを選択した上でWorkDocsを作成してください。

```bash
aws workspaces modify-workspace-creation-properties --resource-id {ディレクトリ ID} --workspace-creation-properties EnableWorkDocs=true
```

WorkDocsの料金については[以下の通り](https://aws.amazon.com/jp/workdocs/pricing/)です。

Amazon WorkSpaces のユーザーには、追加料金なしで WorkDocs へのアクセス権限が付与されます。

これには、WorkSpaces の 1 ユーザーにつき 50 GB のストレージが含まれます。 

WorkSpaces ユーザーは、割引料金で 1 TB のストレージを使用できるようにアップグレードすることができます。

登録できたら登録済みがYesになります。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2011.png)

# 登録したディレクトリの中身確認

---

SMAL認証や、Active Directory ドメインで証明書ベースの認証を設定することができます。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2012.png)

WorkSpacesが所属するOUを指定することができます。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2013.png)

デフォルトで作成されるセキュリティグループとは別に設定したいセキュリティグループがあれば選択することができます。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2014.png)

WorkSpacesにパブリックIPを割り当てることができます。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2015.png)

どのデバイスからWorkSpacesをアクセスさせるかを制限させると同時に、

クライアント証明書を利用してWorkSpaces は証明書ベースの認証を使用して、デバイスが信頼できるかどうかを判断できます。

※Client証明書を利用した接続方法は別記事にて記載

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2016.png)

上記のWindows、MacOS、Android以外のOSから接続させたいかどうかを登録できます。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2017.png)

WorkSpacesのユーザーをローカル管理者としたい場合には有効にします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2018.png)

WorkSpacesにアクセスできるIPアドレスを制御できます。

AWSアカウントあたり最大100のIPアクセスコントロールグループをリージョン毎に作成できますが、

1つのディレクトリに関連付けすることができるのは最大25のIPアクセスコントロールグループのみです。

[WorkSpaces の IP アクセスコントロールグループ](https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/amazon-workspaces-ip-access-control-groups.html)

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2019.png)

Windowsの更新を自動で制御させたい場合には有効にしておくと良いと思います。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2020.png)

ここは利用料金に関わるところになります。デフォルトは全て有効になります。

- このアカウントを記憶する
    - WorkSpaces接続時に毎回アカウント名を訊かれるようになります。パスワードは有効にしても記憶されません。
- クライアントからWorkSpaceを再起動する
    - WorkSpacesClientからWorkSpacesの再起動を行うことができます。
- ボリュームサイズを増やす
    - WorkSpacesClientからWorkSpacesのボリュームサイズを増やせるようになります。
        
        ※WorkSpaces構築後やボリュームサイズを変更した場合、6時間経過しないと変更できません。
        
        ※増やす際にも段階的に（最小で10GB、最大で50GBずつ）しか増やすことしかできません。
        
- コンピューティングタイプを変更する
    - 6時間経過しないと変更できません。
    - スペックを下げるのは30日に1回だけのようです。
- 実行モードを切り替える
    - AlwaysOnにすると、その時点で月額料金が発生するので注意
- クライアントからWorkSpaceを再構築する
    - バンドルイメージから WorkSpacesを再構築することができる機能になります。
    - イメージから再構築の為、リビルド前に設定していたシステム設定やアプリケーションは、イメージ取得時点のデータに戻ります。
    - Cドライブはイメージから再構築が行われます。WorkSpaces作成後の Cドライブのデータは引き継がれません。
    - Dドライブはイメージから再構築が行われません。
        
        WorkSpacesの標準機能である自動バックアップのデータから復旧が行われます。
        
        自動バックアップの取得は 12時間おきに実施され、取得のタイミングは WorkSpaces毎に異なります。
        
        バックアップに関する処理は全てバックグラウンドで制御されるため、ユーザ側で制御できる部分はありません。（世代管理不可）
        

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2021.png)

ADとAD Connectorを繋ぐためのアカウント情報を登録します、後から変更したい場合にはここで変更をします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2022.png)

ADの認証だけでなく多要素認証をつけたい場合には登録することができます。

筆者が知っている中ではPassLogic、Duo Security、ReeRadius/GoogleAuthenticator、Authentication Proxyを使ったやり方があります。

※それぞれ検証した結果は後日どこかで載せる予定です。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2023.png)

# WorkSpaces作成

---

- ActiveDirectoryRightsManagementサービスを追加します。
- 作成する前にWorkSpacesのOUを作成し、OU配下にユーザーを作成しています。
- GPO作成する
    - ADサーバ内のコントロールパネルから管理ツールを選択し、グループポリシーの管理をクリック
    - グループポリシーエディターからグループポリシーを作成
    - 作成したポリシーを右クリックして編集
    - 例えばパスワードポリシーを変えたい場合
        
        コンピュータの構成→ポリシー→Windowsの設定→セキュリティの接敵→アカウントポリシー→パスワードのポリシー
        
    - 作成したグループポリシーを割り当てしたいOUへドラック＆ドロップする
    - ユーザーでグループポリシーが当たっているかどうかを確認実施する

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2024.png)

WorkSpacesを作成します。次へをクリックします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2025.png)

次へをクリックします。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2026.png)

オンプレミスで作成したADのユーザーが表示されるので、ユーザーを選択します。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2027.png)

次にバンドルを選択します。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2028.png)

AlwaysOnとAutoStopが選べます。AutoStopを選択すると利用した時間だけ課金されます。

最小の時間は1からしか選択できません。常時利用したい場合にはAlwaysOnにしますが月額課金になるので注意してください。

詳しい料金は[こちらを参照](https://aws.amazon.com/jp/workspaces/pricing/)ください。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2029.png)

最後にユーザーボリュームを選択します。

なお、ルートボリュームとユーザーボリュームを暗号化すると以下の制限が発生します。[※公式ページより引用](https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/encrypt-workspaces.html)

- 既存の WorkSpace は暗号化できません。WorkSpace を起動するときは、暗号化する必要があります。
- 暗号化された WorkSpace からのカスタムイメージの作成は、サポートされていません。
- 暗号化された WorkSpace の暗号化を無効にすることは、現在サポートされていません。
- ルートボリュームの暗号化を有効にした状態で起動された WorkSpaces では、プロビジョニングに最大 1 時間かかる場合があります。
- 暗号化された WorkSpace を再起動または再構築するには、AWS KMS キーが有効であることを最初に確認します。
    
    有効ではない場合、WorkSpace は使用できません。KMS キーが有効になっているかどうかを確認する方法については、「AWS Key Management Service デベロッパーガイド」の 「[コンソールで KMS キーを表示する](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys-console.html#viewing-console-details)」を参照してください。
    

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2030.png)

最後にプレビュー画面がでるので問題なければ作成します。

![FireShot Capture 746 - WorkSpaces Console - ap-northeast-1.console.aws.amazon.com.png](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/FireShot_Capture_746_-_WorkSpaces_Console_-_ap-northeast-1.console.aws.amazon.com.png)

作成が完了するまで時間がかかるのでしばし待ち、スタータスが使用可能になったら利用可能です。

最後にアクションよりユーザーを招待するをクリックすることで、WorkSpacesの登録コードなどが表示されますのでメモしておきます。

尚、ADユーザーの情報にメールアドレスが記載されていれば招待メールがぶかと思われます。（試していないのでわかりません。。。）

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2031.png)

WorkSpaceClientを手元にダウンロードしてログイン

招待メールにあった登録コードとADのユーザー情報を使ってログインができれば完了です。

尚、接続するにあたっては接続する端末がインターネット向けに以下のように通信要件を満たす必要があります。

あらかじめインターネット回線が引かれているアパートとかだと共用の部分でFWでブロックされていたりして接続できない可能性がでてくるので注意が必要です。

https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/workspaces-port-requirements.html

# WorkSpacesがインターネットに出れる方法

---

WorkSpacesをプライベートサブネットに配置した場合には、

パブリックサブネットにOutBound通信できるようにNAT Gatewayを利用するか、ネットワークアプライアンスを置くという手が考えられます。

WorkSpacesをパブリックサブネットに配置した場合には、WorkSpacesにグローバルIPを紐づけて通信する方法が考えられます。

またオンプレミスと繋いでいる場合にはオンプレミス環境からインターネットに出る手法も考えられますが通信量が多くなるので回線料金が高くなってしまう可能性もあります。

最後に補足です。

尚、WorkDocsはディレクトリ一つに対して、一つしか作成できないのが注意してください。

![Untitled](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c/Untitled%2032.png)

# WorkSpaces作成後にすること

---

PCoIP のグループポリシー管理用テンプレートをオンプレADへインストールして、固有のグループポリシー設定を変更できるようにします。

作成したWorkSpacesにて以下を実行します。

1. C:\Program Files\Teradici\PCoIP Agent\configuration\policyDefinitionsディレクトリ内のおよびファイルのコピーを作成
    
    PCoIP.admx、PCoIP.adml
    
2. オンプレADのエクスプローラーのアドレスバーで¥¥ドメイン名でアクセスします。
3. sysvol フォルダを開きます。
4. ドメイン名のフォルダを開きます。
5. Policiesフォルダを開きます。
6. PolicyDefinitions という名前のフォルダを作成して、PCoIP.admxを保存します。
7. PolicyDefinitions フォルダに en-US という名前のフォルダを作成して、PCoIP.admlを保存します。
8. オンプレADでグループポリシーの管理を開き、[コンピュータの設定]、[ポリシー]、[管理用テンプレート]、[PCoIP セッション変数] の順に選択します。
9. PCoIP セッション変数グループポリシーオブジェクトを使用して、PCoIP を使用する場合に固有のグループポリシー設定を変更できるようになります。
    
    ユーザーによる設定の上書きを許可するには、[Overridable Administrator Settings] (上書き可能な管理者設定) を選択します。
    
    許可しない場合は、[Not Overridable Administrator Settings] (上書き可能でない管理者設定) を選択します。
    

上記を実行したあとにグループポリシーの設定内容などを確認するとWorkSpaces のポリシーだけになり、従来あった「Windowsコンポーネント」等が無くなってしまいました。。。

調べてみるとAWSのやり方はセントラルストアに配置するやり方のようです。（ADは通常複数あるものだと思うので）

```jsx
セントラルストアは、ドメインの共有フォルダにあたるSYSVOLにPolicyDefinitionsフォルダをセットしてあげることで
ドメイン全体にレプリケーションされ、SYSVOL上のフォルダを優先的に見に行くようになる。
```

復旧方法としては「C:\Windows\PolicyDefinitionsのフォルダ」を「\\(ドメイン名)\SYSVOL\(ドメイン名)\Policies\PolicyDefinitions」に上書きコピーすることで、

従来のあったWindowsコンポーネントなどが表示されるようになります。

そのためセントラルストアでのやり方で実施するのであれば、

1. C:\Windows\PolicyDefinitionsのフォルダを\\(ドメイン名)\SYSVOL\(ドメイン名)\Policies\配下にコピー
2. PolicyDefinitions配下にPCoIP.admxを保存
3. PolicyDefinitions フォルダに en-US という名前のフォルダを作成して、PCoIP.admlを保存

というやり方になるのではないかと思いますが注意が必要で、ローカルコンピューターに配置した(C:\Windows\PolicyDefinitions)拡張子のadmx、adml のファイルは読み込まれなくなります。

AD毎に設定するということであれば、以下にローカルコンピューターに配置することで、このドメインコントローラのローカルのみ適用されます。

C:\Windows\PolicyDefinitions\PCoIP.admx

C:\Windows\PolicyDefinitions\en-US\PCoIP.adml