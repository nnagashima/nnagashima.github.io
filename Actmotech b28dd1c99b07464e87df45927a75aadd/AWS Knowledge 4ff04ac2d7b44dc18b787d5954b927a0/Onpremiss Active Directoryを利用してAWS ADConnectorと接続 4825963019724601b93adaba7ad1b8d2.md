# Onpremiss Active Directoryを利用してAWS ADConnectorと接続

![Windows_icon-icons.com_56585.png](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Windows_icon-icons.com_56585.png)

# 目次

---

# はじめに

---

AWS AD Connectorと接続するために、ActiveDirectoryのインストールと制御委任を実施します。

# Active Directoryの機能追加

---

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled.png)

役割ベースまたは機能ベースのインストール

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%201.png)

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%202.png)

Active Directory ドメインサービスをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%203.png)

管理ツールを含めるを有効化し、機能の追加をクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%204.png)

次へをクリッ

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%205.png)

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%206.png)

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%207.png)

インストールをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%208.png)

インストール中の画面

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%209.png)

完了したら閉じるをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2010.png)

# ドメインコントローラーに昇格

---

旗印をクリックして、このサーバをドメインコントローラーに昇格するをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2011.png)

新しいフォレストを追加するを選択し、ルートドメインにドメイン名を入力して次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2012.png)

パスワードを入力

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2013.png)

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2014.png)

少し待つとNetBIOSドメイン名が表示されるので、任意のものしたければ変更し次へをクリッ

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2015.png)

ファイルパスを変更する場合にはパスを入力し、次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2016.png)

次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2017.png)

インストールをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2018.png)

インストール中の画面

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2019.png)

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

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2020.png)

作成例：

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2021.png)

# AWS AD Connectorと接続するためのユーザー作成

---

作成したOU配下で、AWSのAD Connectorと作成するためのユーザーを作成します。

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2022.png)

パスワード入力、パスワードは無期限に設定

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2023.png)

確認がでるので問題なければ完了をクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2024.png)

作成したOU配下で、AWSのAD Connectorと作成したユーザーad connectorが所属するためのConnectorsグループを作成

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2025.png)

作成したadconnectorsに作成したグループのConnectors配下に所属

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2026.png)

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

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2027.png)

次へをクリック

![FireShot Capture 685 - 192.168.11.52_ray-win2022ad - gua2.actmotech.xyz.png](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/FireShot_Capture_685_-_192.168.11.52_ray-win2022ad_-_gua2.actmotech.xyz.png)

先ほど作成したConnectorsグループを追加し、次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2028.png)

委任するカスタムタスクを作成するを選択し、次へをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2029.png)

フォルダー内の次のオブジェクトのみを選択し、

コンピューターオブジェクト と ユーザーオブジェクト の2つ選択した後に、

選択されたオブジェクトをこのフォルダーに作成すると選択されたオブジェクトをこのフォルダーから削除するを有効化

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2030.png)

全般とプロパティ固有を有効化

最低限の設定としてはアクセス許可に書き込みいらないが、読み取りと書き込みをクリック

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2031.png)

完了をクリックし委任設定は完了

![Untitled](Onpremiss%20Active%20Directory%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6AWS%20ADConnector%E3%81%A8%E6%8E%A5%E7%B6%9A%204825963019724601b93adaba7ad1b8d2/Untitled%2032.png)

AWS AD Connectorと接続する内容は以下を参照ください。

[オンプレADと連携してAWS DirectoryService AD ConnectorとWorkSpacesを作成](%E3%82%AA%E3%83%B3%E3%83%95%E3%82%9A%E3%83%ACAD%E3%81%A8%E9%80%A3%E6%90%BA%E3%81%97%E3%81%A6AWS%20DirectoryService%20AD%20Connector%E3%81%A8Work%20056d3c2c1abf4fb0a02d38849f43c44c.md)