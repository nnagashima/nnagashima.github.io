[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# AWSからRed Hatナレッジベースにアクセス

# 目次
- [AWSからRed Hatナレッジベースにアクセス](#awsからred-hatナレッジベースにアクセス)
- [目次](#目次)
- [はじめに](#はじめに)
- [条件は？](#条件は)
- [条件をクリアすると](#条件をクリアすると)

---

# はじめに

---

AWSでRHELを利用している場合にAWSサポートを経由せずにRedHatのナレッジベースにアクセスできるようにします。

RHELを利用する場合はナレッジベースにアクセスできるようにしておくと便利なこともあるので、

設定することをお勧めします。

# 条件は？

---

以下、条件を確認しました。

- サブスクリプション付きのRHELインスタンスが起動している
- SSMエージェントがインストールされておりFleetMangerで認識できている

# 条件をクリアすると

---

AWS System Manager Feet Managerの画面に移動してアカウント管理からRHELナレッジベースへのアクセスをクリック

もしくは

RHELインスタンスの詳細へ移動してRHELナレッジベースのコンテンツへのアクセスをクリック

することでナレッジベースのTOPページに移動されます。

認証は１日で期限切れする仕様です。

またRHELインスタンス起動してから24時間経過しないとナレッジベースのリンクをクリックしてもエラーが発生するので、

24時間経過してからアクセスできるか確認をしてみてください。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
