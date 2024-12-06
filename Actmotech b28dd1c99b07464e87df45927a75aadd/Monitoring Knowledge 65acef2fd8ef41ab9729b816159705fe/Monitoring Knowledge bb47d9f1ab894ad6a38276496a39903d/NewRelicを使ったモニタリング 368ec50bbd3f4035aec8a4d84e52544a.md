# NewRelicを使ったモニタリング

![MediaAsset_Tiles_RGB_POS_HZ.svg](NewRelic%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%A2%E3%83%8B%E3%82%BF%E3%83%AA%E3%83%B3%E3%82%AF%E3%82%99%20368ec50bbd3f4035aec8a4d84e52544a/MediaAsset_Tiles_RGB_POS_HZ.svg)

# 目次

---

# はじめに

---

Zabbixを自宅環境のモニタリングに利用をしていますが、 Zabbixが落ちたことによる監視が現状だとできないので、 

インターネット越しに物理基盤のProxmoxと仮想マシンをモニタリンするにあたって、 NewRelicを採用しました。

採用した背景としてはデータが100GB/月と制約とログインできるユーザーは1名の制約はあるものの、 

機能制限はないので取り込むメトリックを調整できれば、問題ないだろうという観点と 会社でもNewRelicを利用しているということもあり採用に至りました。

# NewRelicのアカウント登録

---

以下のページからサインアップしてアカウント作成します。

 https://newrelic.com/jp/sign-up-japan

# NewRelicAgentのインストール

---

Agentをインストールするにあたって監視対象がアウトバウンドでインターネットアクセスできることが必要です。

プライベート環境で外に出れない場合にはProxyを用意することで監視することができると思います。 

デフォルトのモニタリング間隔はインフラストラクチャの観点ではSamples variablesを参照すると良いと思います。 

https://docs.newrelic.com/jp/docs/infrastructure/install-infrastructure-agent/configuration/infrastructure-agent-configuration-settings/#samples-variables

UI上のAdd dataからNew Relic infrastructure agentを選択し、インストールするOSを選択します。 

その後、インストールコマンドが発行されるのでCopyして監視したい対象で実行をします。 

Agentをインストールする際にホスト上で動いているものがあればディスカバリしてくれますが、 ここでは一旦NOと選択しています。

```
# curl -Ls https://download.newrelic.com/install/newrelic-cli/scripts/install.sh | bash && sudo NEW_RELIC_API_KEY==************ NEW_RELIC_ACCOUNT_ID=************ /usr/local/bin/newrelic install

Running agent status check attempt...
Agent status check ok.
✔ Installing Infrastructure Agent
   Installed

We've detected additional monitoring that can be configured by installing the following:
  Golden Signal Alerts
  Apache Integration
  PHP Agent

? Continue installing?  No

  New Relic installation complete

  --------------------
  Installation Summary

  −  Golden Signal Alerts  (skipped)
  −  Apache Integration  (skipped)
  ✔  Infrastructure Agent  (installed)
  −  PHP Agent  (skipped)

  View your data at the link below:
  ⮕  https://********************

  --------------------
```

Agentをインストールすると以下ファイルが作成されます。

```
# cat /etc/newrelic-infra.yml
enable_process_metrics: true
status_server_enabled: true
status_server_port: 18003
license_key: ******************
```

enable_process_metricsは、プロセスを監視するかどうかの設定になります。 

この設定のままだと全てのプロセスデータを取得してくるので必要なものだけを監視する場合には正規表現で記載をします。

```
include_matching_metrics:
    process.name:
      - regex "^zabbix-server$"
```

status_server_enabledは、NewRelicAgentのステータス監視になる？と思います。 

デフォルトのポートは18003となるのでバッティングされないように気をつけてください。

監視間隔のメトリクスを修正するには以下のように記載します。

```
metrics_system_sample_rate: 300 #デフォルトは5秒です。
metrics_process_sample_rate: 300 #デフォルトは20秒です。
metrics_storage_sample_rate: 300 #デフォルトは20秒です。
metrics_network_sample_rate: 300 #デフォルトは10秒です。
```

Agentをインストールすると以下ファイルも作成されます。 このままだと全てのログが取り込まれてしまうので、 

取り込みたいログのパスやキーワードを記載します。 

https://docs.newrelic.com/docs/logs/forward-logs/forward-your-logs-using-infrastructure-agent/#pattern

```
# cat /etc/newrelic-infra/logging.d/logging.yml
logs:
  - name: cloud-init.log
    file: /var/log/cloud-init.log
    attributes:
      logtype: linux_cloud-init
  - name: messages
    file: /var/log/messages
    attributes:
      logtype: linux_messages
  - name: secure
    file: /var/log/secure
    attributes:
      logtype: linux_secure
  - name: yum.log
    file: /var/log/yum.log
    attributes:
      logtype: linux_yum
  - name: newrelic-cli.log
    file: /root/.newrelic/newrelic-cli.log
    attributes:
      newrelic-cli: true
      logtype: newrelic-cli
```

あとはUI上で取得できたメトリクスをグラフで確認することができます。

![438.png](NewRelic%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%A2%E3%83%8B%E3%82%BF%E3%83%AA%E3%83%B3%E3%82%AF%E3%82%99%20368ec50bbd3f4035aec8a4d84e52544a/438.png)

もう少しNewRelicを磨いたら各ミドルウェアやAPMやアラート設定なども投稿したいと思います。