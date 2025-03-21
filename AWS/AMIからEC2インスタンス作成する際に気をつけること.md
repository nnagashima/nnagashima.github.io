[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# AMIからEC2インスタンス作成する際に気をつけること

# 目次

- [AMIからEC2インスタンス作成する際に気をつけること](#amiからec2インスタンス作成する際に気をつけること)
- [目次](#目次)
- [AMIからEC2を起動する際に気をつけること](#amiからec2を起動する際に気をつけること)
  - [Windows編](#windows編)
  - [Linux編](#linux編)

---

# AMIからEC2を起動する際に気をつけること

---

## Windows編

---

- 注意点1

```jsx
作成したAMIからEC2を新規作成して起動した後にパスワード取得してもパスワードが取得できないケースがあります。
Sysprepを実行してないAMIから復元した場合に「パスワードは使用できません。」といった警告がでるので、
元のAMIのAdministratorパスワードは必ず控えるようにしておきましょう。
```

- 注意点2

```jsx
AWS ConsoleからWindowsパスワードの取得をしたい場合は、
作成したAMIからEC2を新規作成した後はEC2にログイン後に「EC2LaunchSettings」を起動し、
「Administrator password setting」の選択で「Random(retieve from console)」が
選択されていることを確認し、「Shutdown_with_Sysprep」を実行してSysprepを実行しましょう。
その後、展開用AMIとしてSysprepを実行したEC2で取得します。
```

- 注意点3

```jsx
AWS Backupで取得したものはSysprepが実行されていないので、
注意点2と同様にSysprepが必要になります。
```

- 注意点4

```jsx
Sysprepを実行するとタイムゾーンがリセットされるので再設定が必要になります。
```

- 注意点5

```jsx
パブリック公開のAMIをプライベートサブネット（インターネットに出れないサブネット）で作成するとWidowsのライセンス認証が失敗するケースがあります。
Windowsインスタンスは、WindowsKMSライセンス認証を使用し180日ごとにライセンス認証を行う必要があります。
解決方法としては以下ドキュメント記載にある通りとなります。
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/common-messages.html#activate-windows
```

- 注意点6

```jsx
ドメイン参加しているEC2のAMIを取得して復元する場合には、同じSIDとコンピューター名で実行されているドメインに参加しているWindowsインスタンスで問題が発生する可能性があります。
他のサーバーやサービスが新しいインスタンスに接続するのを防ぐには、セキュリティグループを使用して、
アクセスやテスト用に自分のIPアドレスを除く新しいインスタンスのすべてのインバウンド接続を一時的にブロックします。
また、新しいインスタンスのアウトバウンド接続を一時的にブロックして、サービスやアプリケーションが他のリソースへの接続や更新を開始しないようにすることもできます。
新しいインスタンスの準備ができたら、既存のインスタンスを停止し、新しいインスタンスでサービスとプロセスを開始し、実装したインバウンドまたはアウトバウンドのネットワーク接続のブロックを解除します。
※この場合はAMIから復元するのではなくスナップショットからEBSを作成してインスタンス停止してボリュームのデタッチ／アタッチをした方がいいと思います。
```

## Linux編

---

- 注意点1

```jsx
作成したAMIからEC2を起動するとホスト名がデフォルトになってしまうので、
/etc/cloud/cloud.cfgにpreserve_hostnameをtrueを追加します。
```

- 注意点2

```jsx
作成したAMIからEC2を起動するとパスワード認証の設定（PasswordAuthentication yes）
を設定してもパスワード認証ができなくなるので、
/etc/cloud/cloud.cfgにssh_pwauth: 1を追加します。
```

- 注意点3

```jsx
作成したAMIからEC2を起動するとシステム言語の設定（localectl set-locale LANG=ja_JP.utf8）
を設定しても言語設定が戻ってしまうので、
/etc/cloud/cloud.cfgにlocale: ja_JP.UTF-8を追加します。
```

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
