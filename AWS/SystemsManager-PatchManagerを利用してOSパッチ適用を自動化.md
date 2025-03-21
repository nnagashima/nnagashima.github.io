[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# AWS Systems Manager Patch Managerを利用してOSパッチ適用を自動化

# 目次

---

# AWSのPatch Managerとは

---

AWS Systems Manager の一機能である Patch Manager は、セキュリティ関連の更新およびその他の種類の更新の両方を使用してマネージドノードにパッチを適用するプロセスを自動化します。

WindowsOS、LinuxOSどちらにも対応しており、パッチの区分や重要度に応じて適用可否を制御する機能もあります。

# 前提条件

---

1. パッチ適用対象のインスタンスはSystemsMangerによって管理されている必要があるので、
    
    SSMAgentの導入と適切なIAMポリシーが必要になります。
    
2. パッチレポジトリにアクセスできるようにする必要があります。
    
    LinuxについてはAWSが提供しているリポジトリがありますが、
    
    Windowsはないためインターネット上のパッチレポジトリにアクセスできるようにする必要があります。
    
3. プライベートサブネットの場合、SSMのVPCエンドポイントが必要になります。
    
    東京リージョンの場合：
    
    - com.amazonaws.ap-northeast-1.ssm
    - com.amazonaws.ap-northeast-1.ec2messages
    - com.amazonaws.ap-northeast-1.ssmmessages
    - com.amazonaws.ap-northeast-1.s3

# 作成までの流れ

---

1. パッチベースラインの作成
2. パッチグループ名の指定
3. パッチ適用対象EC2インスタンスへタグの付与
4. パッチ適用を設定
5. 実行結果確認

# 1. パッチベースラインの作成

---

- 設定名は任意の名前にしてください。
- オペレーティングシステムはWindowsとします。
- オペレーティングシステムルールの「製品」「分類」「重要度」を設定します。
- 自動承認には「指定した日数後にパッチを承認する」を選択します。

![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled1.png)

# 2. パッチグループ名の指定

---

作成したパッチベースラインを「アクション」から「パッチグループの変更」を選択します。

「パッチグループ」欄に任意の文字列を入力し「追加」ボタンをクリックしパッチグループが追加されたことを確認します。

![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled2.png)

# 3. **パッチ適用対象EC2インスタンスへタグの付与**

---

2で実施した以下の文字列をEC2インスタンスのタグに指定します。

タグのキーにはPatch Groupとします。

```jsx
「パッチグループ」欄に任意の文字列を入力し「追加」ボタンをクリックしパッチグループが追加されたことを確認
```

# 4.パッチ適用を設定

---

パッチポリシーを作成します。

スキャンについては毎日0時、インストールについては0時の日曜日に実施としました。

必要に応じてメンテナンス時間が決まっていればパッチ適用後にOS再起動も実行が可能です。

![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled3.png)

同じようにAmazonLinux2023向けのパッチベースラインとパッチポリシーを作成しました。

# 5. パッチポリシーの実行結果確認

---

動作結果は以下の通りとなります。RunCommandの結果を確認します。

![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled4.png)

コンプライアンスレポートから準拠になっていることを確認、また適用されているパッチの一覧を見ることもできます。

![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled5.png)
![](/AWS/SystemsManager-PatchManagerを利用してOSパッチ適用を自動化/Untitled6.png)

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
