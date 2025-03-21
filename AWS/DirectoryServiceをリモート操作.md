[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# AWS Directory Serviceをリモート操作

# 目次

---

# はじめに

---

AWS Directory ServiceはManaged Serviceになるためドメインに参加している端末から、

リモートでの操作が必要になってきます。

そのために必要なツールをインストールします。

# ツールのインストール

---

サーバーマネジャーを開き、機能と役割の追加をクリックします。

機能の画面のところで以下を追加します。

- グループポリシー管理ツール
- リモートサーバ管理ツール
    - 役割管理ツール
        - AD DSおよびAD LDS管理ツール
- DNSサーバーツール

# Directory Serviceへのアクセス方法

---

ユーザーを作成したい場合、スタートメニューからWindows管理ツールをクリックし、

Active Directory ユーザーとコンピューターを右クリックします。

その後、別のユーザーとして実行をクリックしてSimpleAD、MSADを作成した際の管理者アカウントで認証を実施することで、

Directory Serviceの設定を実施することができます。

※他の操作をする場合でも同様です。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
