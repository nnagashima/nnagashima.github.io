# FleetManagerでOSにログインせずにできることは

![Untitled](AWS%20Systems%20Manager%20Patch%20Manager%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6OS%E3%83%8F%E3%82%9A%E3%83%83%E3%83%81%E9%81%A9%E7%94%A8%E3%82%92%E8%87%AA%E5%8B%95%E5%8C%96%203d38d67817f44f84a31e115a326d328b/Untitled.png)

# 目次

---

# SessionManagerとの違いは？

---

個人的な理解では以下が違うところかと思います。

WindowsServerへRDP接続がFleetMangerではできるが、SessionManagerではCLI接続

その他FleetManagerでできることは以下に記載します。

# FleetManagerでできること

---

よくお世話になっているFleetMangerでOSへの接続はしていますが、その他できることをまとめてみました。

- EC2の簡易情報の表示
- ファイルシステム情報の表示
- パフォーマンスカウンターの表示
- プロセス情報の表示
- ユーザーとグループの表示
- Windowsイベントログ【WindowsOSのみ】
- Windowsレジストリ【WindowsOSのみ】
- EBSボリューム【WindowsOSのみ】

# EC2の簡易情報の表示

---

以下のような情報が表示されます。

以下インスタンスではPatchMangerを利用してパッチ管理をしているためパッチのインストール数などが表示されています。

![FireShot Capture 1012 - 全般 - フリートマネージャー - Systems Manager - ap-northeast-1_ - ap-northeast-1.console.aws.amazon.com.png](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/FireShot_Capture_1012_-_%25E5%2585%25A8%25E8%2588%25AC_-_%25E3%2583%2595%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%2588%25E3%2583%259E%25E3%2583%258D%25E3%2583%25BC%25E3%2582%25B7%25E3%2582%2599%25E3%2583%25A3%25E3%2583%25BC_-_Systems_Manager_-_ap-northeast-1__-_ap-northeast-1.console.aws.amazon.com.png)

# ファイルシステム情報の表示

---

Linuxの場合には以下のようにディレクトリが表示できます。

ここからファイルの中身をテキスト形式でみることができます。

## 【Linux】

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled.png)

## 【Windows】

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%201.png)

# パフォーマンスカウンターの表示

---

フリートマネージャーで EC2 インスタンスのパフォーマンスカウンターを見るには、

SSM Agent バージョン 2.3.539.0 以降がインストールされていることが必要です。

またセッションマネージャーの設定で AWS Key Management Service (AWS KMS) 暗号化の有効化が必要となります。

作成したKMSキーにアクセスできるよう以下のようなIAMポリシーをEC2のIAMロールに追加設定します。

```jsx
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": [
            "kms:Decrypt"
        ],
        "Resource": [
            "arn:aws:kms:region:AccountID:key/KMSキー"
        ]
    }
}
```

上記設定後にしばらくするとパフォーマンスカウンターから以下のようにメトリクスを表示できます。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%202.png)

# プロセス情報の表示

---

プロセス情報の表示も同様にセッションマネージャーの設定で AWS Key Management Service (AWS KMS) 暗号化の有効化が必要となります。

画面上からプロセスの停止とプロセスの起動することができます。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%203.png)

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%204.png)

# ユーザーとグループの表示

---

ユーザーの一覧とグループの一覧が表示ができ、ユーザーやグループ作成/削除することも可能です。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%205.png)

# Windowsイベントログ【WindowsOSのみ】

---

WindowsのイベントログがOSにログインしなくとも見ることができます。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%206.png)

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%207.png)

# Windowsレジストリ【WindowsOSのみ】

---

Windowsのレジストリ設定がOSにログインしなくとも見ることができます。

また「レジストリキーの作成」「レジストリエントリの作成」もすることができます。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%208.png)

# EBSボリューム【WindowsOSのみ】

---

アタッチされているEBSボリュームのパーティションスタイルやサイズなど見ることができます。

また追加したEBSボリュームの初期化とフォーマットもできるようです。

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%209.png)

![Untitled](FleetManager%E3%81%A6%E3%82%99OS%E3%81%AB%E3%83%AD%E3%82%AF%E3%82%99%E3%82%A4%E3%83%B3%E3%81%9B%E3%81%99%E3%82%99%E3%81%AB%E3%81%A6%E3%82%99%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%2073d62fffac0340698630f6b54fef784b/Untitled%2010.png)