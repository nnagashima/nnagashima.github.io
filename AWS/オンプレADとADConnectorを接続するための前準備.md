[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# Onpremiss Active Directoryを利用してAWS ADConnectorと接続

# 目次

- [Onpremiss Active Directoryを利用してAWS ADConnectorと接続](#onpremiss-active-directoryを利用してaws-adconnectorと接続)
- [目次](#目次)
- [はじめに](#はじめに)
- [Active Directoryの機能追加](#active-directoryの機能追加)
- [ドメインコントローラーに昇格](#ドメインコントローラーに昇格)
- [OU作成](#ou作成)
- [AWS AD Connectorと接続するためのユーザー作成](#aws-ad-connectorと接続するためのユーザー作成)

---

# はじめに

---

AWS AD Connectorと接続するために、ActiveDirectoryのインストールと制御委任を実施します。

# Active Directoryの機能追加

---

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled.png)

役割ベースまたは機能ベースのインストール

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled1.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled2.png)

Active Directory ドメインサービスをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled3.png)

管理ツールを含めるを有効化し、機能の追加をクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled4.png)

次へをクリッ

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled5.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled6.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled7.png)

インストールをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled8.png)

インストール中の画面

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled9.png)

完了したら閉じるをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled10.png)

# ドメインコントローラーに昇格

---

旗印をクリックして、このサーバをドメインコントローラーに昇格するをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled11.png)

新しいフォレストを追加するを選択し、ルートドメインにドメイン名を入力して次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled12.png)

パスワードを入力

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled13.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled14.png)

少し待つとNetBIOSドメイン名が表示されるので、任意のものしたければ変更し次へをクリッ

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled15.png)

ファイルパスを変更する場合にはパスを入力し、次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled16.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled17.png)

インストールをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled18.png)

インストール中の画面

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled19.png)

インストールが完了するとOS再起動されます。

# OU作成

---

[OUとは(e-words.jpより引用)](https://e-words.jp/w/OU.html)

```bash
Active Directoryではアカウント管理の最小単位となる、アカウントやコンピュータ、リソースの集合をOUと言います。
ドメイン内のユーザーアカウントや共有リソースはいずれかのOUに属している必要があり、OU全体で共通する設定などを一元的に管理することができる。
OUの中にOUを設けて階層構造にすることもでき、ドメイン管理者がOU管理者に権限の一部を移譲したり、上位OUの設定を下位OUに引き継いで適用したりすることができる。
```

OUを作成します。ドメイン直下でないと作成できない模様

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled20.png)

作成例：

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled21.png)

# AWS AD Connectorと接続するためのユーザー作成

---

作成したOU配下で、AWSのAD Connectorと作成するためのユーザーを作成します。

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled22.png)

パスワード入力、パスワードは無期限に設定

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled23.png)

確認がでるので問題なければ完了をクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled24.png)

作成したOU配下で、AWSのAD Connectorと作成したユーザーad connectorが所属するためのConnectorsグループを作成

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled25.png)

作成したadconnectorsに作成したグループのConnectors配下に所属

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled26.png)

AWS AD Connectorと接続するためにドメインルートで制御の委任を実施します。

※個人的に検証した限りではWorkSpacesを格納するOUだけに制御委任でも問題なく動作することを確認しました。

尚、MSADの場合は公式ドキュメントに記載のある通り、ドメインルートレベルで制御委任する権限がないため、OU単位で制御委任を実施

引用URL：[権限をサービスアカウントに委任する](https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/prereq_connector.html#connect_delegate_privileges)

```bash
[Active Directory User and Computers] (Active Directory ユーザーとコンピュータ) ナビゲーションツリーで、ドメインルートを選択します。
メニューで [Action] (アクション) を選択し、[Delegate Control] (制御を委任する) を選択します。
AD Connector が AWS Managed Microsoft AD に接続されている場合、ドメインのルートレベルで制御を委任するアクセス権限がありません。
この場合、制御を委任するには、コンピュータオブジェクトが作成されるディレクトリ OU で OU を選択します。
```

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled27.png)

次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/pic1.png)

先ほど作成したConnectorsグループを追加し、次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitle28.png)

委任するカスタムタスクを作成するを選択し、次へをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled29.png)

フォルダー内の次のオブジェクトのみを選択し、

コンピューターオブジェクト と ユーザーオブジェクト の2つ選択した後に、

選択されたオブジェクトをこのフォルダーに作成すると選択されたオブジェクトをこのフォルダーから削除するを有効化

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled30.png)

全般とプロパティ固有を有効化

最低限の設定としてはアクセス許可に書き込みいらないが、読み取りと書き込みをクリック

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled31.png)

完了をクリックし委任設定は完了

![](/AWS/オンプレADとADConnectorを接続するための前準備/Untitled32.png)

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)