[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# EC2のAutoRecovery設定


# 目次

- [EC2のAutoRecovery設定](#ec2のautorecovery設定)
- [目次](#目次)
- [AutoRecoveryとは](#autorecoveryとは)
- [AutoRecoveryのアラーム作成](#autorecoveryのアラーム作成)
- [AutoRecoveryの注意点](#autorecoveryの注意点)

---

# AutoRecoveryとは

---

CloudWatchのモニタリングサービスを利用して物理的な障害が発生した際に、

EC2インスタンス単位での自動復旧ができるようにする仕組みになります。

EC2作成時に自動復旧の設定がありますが、この設定だとアラートになった際に通知がこないので、

通知が必要な場合には手動でアラームを作成する必要があります。

# AutoRecoveryのアラーム作成

---

EC2インスタンスを右クリックして「CloudWatchアラームの管理」をクリックしてアラーム作成をクリックします。

- 「アラーム作成」を選択します。
- アラーム通知に通知したいSNSトピックを選択します。
- アラームアクションに「復旧」をクリックします。
- サンプルのグループか基準を「最小」にします。
- サンプリングするデータのタイプを「ステータスチェックの失敗：システム」にします。

上記内容を入力したら作成をクリックします。

![](/AWS/EC2のAutoRecovery設定/pic1.png)

![](/AWS/EC2のAutoRecovery設定/pic2.png)

CloudWatchの画面に移動して作成したアラームが存在していることを確認します。

![](/AWS/EC2のAutoRecovery設定/pic3.png)

アクションタブにてインスタンス復旧のアクションと、アラーム状態の時のアクションが存在していることを確認します。

このままだと復旧時の時にはアラーム通知が実施されないので、右上のアクション▼から「編集」をクリックします。
アクションの設定の画面で「通知の追加」をクリックし、復旧アクションを追加してアラームを更新します。

![](/AWS/EC2のAutoRecovery設定/pic5.png)

# AutoRecoveryの注意点

---

AutoRecoveryを設定するにあたっての条件は以下になります。

```jsx
1. 対応しているインスタンスタイプは以下です。
A1、C3、C4、C5、C5a、C5n、C6g、Inf1、 M3、M4、M5、M5a、M5n、M5zn、M6g、 P3、R3、R4、R5、R5a、R5b、R5n、R6g、 T2、T3、T3a、X1、X1eのいずれか

2. EBSボリュームのみ（インスタンスストアがない）

3. defaultまたはdedicatedインスタンステナンシーで起動していること
```

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
