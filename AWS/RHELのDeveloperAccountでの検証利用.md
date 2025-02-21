[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# RHELのDeveloperAccountを使ったAWSでのRHELの検証利用方法

# 目次

- [RHELのDeveloperAccountを使ったAWSでのRHELの検証利用方法](#rhelのdeveloperaccountを使ったawsでのrhelの検証利用方法)
- [目次](#目次)
- [はじめに](#はじめに)
- [Developerアカウントの作成](#developerアカウントの作成)
- [Developerアカウントの作成の作成後](#developerアカウントの作成の作成後)
- [Red Hat Customer Portalにログイン](#red-hat-customer-portalにログイン)
- [RHELをサブスクリプションに登録して利用する](#rhelをサブスクリプションに登録して利用する)
- [AWSにも同じように適用できないか](#awsにも同じように適用できないか)
- [AWSへの適用方法](#awsへの適用方法)
- [AWSにDeveloperアカウント適用後にEC2を起動](#awsにdeveloperアカウント適用後にec2を起動)
- [RHEL9のEOLについて](#rhel9のeolについて)

---

# はじめに

---

RHELのDeveloperAcctountについて記載します。

本来RHELのライセンスは費用がかかるものになりますが検証目的であれば、

16台まで無償でどのRHELのバージョンでも利用することが可能です。（商用目的で利用した場合違反となります）

尚、ライセンスの有効期限は1年ごとになりますが再度更新してサブスクリプションを割り当てることで、

利用継続することが可能になります。詳細はRHELのBLOGを見てください。

https://rheb.hatenablog.com/entry/developer-program

# Developerアカウントの作成

---

以下URLにアクセスするとActive your subscriptionのボタンがあるのでクリックし、

Registor for a Red Hat Accountをクリックし、必要事項を入力してアカウントを作成します。

https://developers.redhat.com/products/rhel/download

# Developerアカウントの作成の作成後

---

先ほど表示した画面でログインを行うと以下のような形でIOSをダウンロードすることができます。

# Red Hat Customer Portalにログイン

---

Red Hat Customer Portalにログインして左上にSubscriptionと表示されているので、クリックします。

https://access.redhat.com/

ログインするとRed Hat Developer Subscription for Individualsがあるのでクリックすると

エンタイトルメントの使用率が16となっていることを確認できればOKです。

# RHELをサブスクリプションに登録して利用する

RHELをインストールしたらサブスクリプション登録が必要になります。

ISOイメージは先ほどの上記画面からダウンロードしてください。

以下root権限で作業します。※RHEL8の例でのコマンドです。

```bash
利用可能なリポジトリがないことを確認します。
# dnf repolit

ユーザー名とパスワードはRed Hat Customer Portalのアカウントを利用します。--nameはオプションです。（省略した場合はホスト名での登録となる）
登録が成功するとこのシステムは、次の ID で登録されましたの表示がされます。
# subscription-manager register --username [username] --password [password] --name [SYSTEM_NAME]

システムのロールとサービスレベルを設定します。
自動の場合には以下コマンドです。
# subscription-manager attach --auto

プールIDを指定したい場合には以下コマンドです。
プールIDの確認はsubscription-manager list --consumedで確認できます。
# subscription-manager attach --auto

最後にサブスクリプションのステータスを確認します。
# subscription-manager list

再度リポジトリを確認するとリポジトリが使えるようになります。
# dnf repolit

アップデートテストでアップデートできれば成功です。
# dnf check-update
```

最後にRed Hat Customer Portalで登録状況を確認して対象ホスト名が緑色の●で登録されていれば登録完了です。

お疲れ様でした。。。

としたいのですが最後に疑問に思って調べたことを書かせてください。

# AWSにも同じように適用できないか

AWSでもMarketplaceでRHELのOSは提供されていますが、インスタンス料金にOS分も含まれて課金されます。

個人のAWS料金はなるべく抑えたいのでどうにかしてできないか確認をしました。

まず前提としてAWSのMarketplaceで利用したRHELのEC2で障害など問合せしたい場合には、

AWSサポートに連絡する形になりますがAWSのサポート契約（デベロッパー、ビジネス、エンタープライズどれか）を結んでいる必要があります。

またAWSサポートでRed Hat へのエスカレーションも行う（AWSが必要と判断した場合のみ）ため、利用ユーザーから直接Red Hatへ問い合わせることはできません。

※その代わり別途Red Hatとの契約は不要になります。

※Red Hat経由でOSライセンスは購入していたけど後からAWSのMarketplaceで提供されているRHELのOSにしたいはできないようです。

# AWSへの適用方法

Red Hat Customer Portalへログインします。

Cloud AccessからRed Hat Cloud Accessの画面へ移動し、AWS AccountsのタブをクリックしAdd accountsをクリックします。

Red Hat Hybrid Cloud Consoleの画面が表示されますので、Amazon Web Servicesをクリックします。

※この画面で確認した際にはAWS以外にもGoogle Cloud、Microsoft Azureも繋げるようでした。

Name Sourceの画面が表示されたらAWSアカウントの任意の名前を記載し、Nextをクリックします。

認証する方法にはアカウント認証方法とマニュアルでの認証方法がありますが、

Account認証のほうではAPIのアクセスキーとシークレットキーの登録になるので本利用は避けて、

マニュアルでの認証方法をとり、Manual configurationをクリックします。

Select applicationの画面ではRHEL管理を選択し、IAM Policyが表示されるので対象のAWSアカウントで作成します。

次にIAM roleでIAMロール作成する手順が表示されるので作成します。

最後にIAMロールのARN情報を入力して最終確認ページで問題なければAddをクリックします。

適用されているかの確認についてはRed Hat Cloud AccessのRed Hat Hybrid Cloud ConsoleでAvailableになることを確認します。

# AWSにDeveloperアカウント適用後にEC2を起動

AWSコンソールのEC2サービスからプライベートイメージ選択して、検索テキストボックスに「309956199498」と入力し、

GoldImageが一覧に表示されていることを確認してください。

※Red Hat Gold Image の場合には、AMI 名のサブスクリプションモデルを表す部分は Access の指定になります。

※RHELのライセンスをBYOLする際に利用することをGold Imageと呼びます。

# RHEL9のEOLについて

以下ページに記載があったので追記しておきます。

RHEL7/8もついでに載せておきます。

https://access.redhat.com/support/policy/updates/errata

![](/AWS/RHELのDeveloperAccountでの検証利用/Untitled.png)

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
