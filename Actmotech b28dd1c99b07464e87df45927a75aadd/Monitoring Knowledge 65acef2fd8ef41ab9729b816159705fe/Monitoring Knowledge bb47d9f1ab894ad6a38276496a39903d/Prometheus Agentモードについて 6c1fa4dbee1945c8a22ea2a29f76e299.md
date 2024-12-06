# Prometheus Agentモードについて

![prometheus-icon-511x512-1vmxbcxr (1).png](Prometheus%20Agent%E3%83%A2%E3%83%BC%E3%83%88%E3%82%99%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%206c1fa4dbee1945c8a22ea2a29f76e299/prometheus-icon-511x512-1vmxbcxr_(1).png)

# 目次

---

# はじめに

---

本機能については投稿時点ではまだフィーチャー機能であるため動作保証ができないものとなります。

お気をつけください。

※2023/01/25

# Prometheus Agentモードについて

---

Prometheus自身は監視ツールになりますが、AgentモードにすることでPrometheusをターゲットのディスカバリや

メトリクスのスクレイプ、リモート書き込みの機能のみにすることができます。

Agentモードを有効化することでクエリーやアラート、ローカルストレージを無効化し、TSDB WALを利用します。

TSDB WALはRemoteStorageの書き込みが成功するとデータを削除し、

失敗した場合にはRemoteStorageが復帰するまで2時間分のデータを一時的に保持するようになっています。

https://prometheus.io/blog/2021/11/16/agent/

![Untitled](Prometheus%20Agent%E3%83%A2%E3%83%BC%E3%83%88%E3%82%99%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%206c1fa4dbee1945c8a22ea2a29f76e299/Untitled.png)

# Prometheus Agentモードの設定方法

---

PrometheusのAgentモードはv2.32.0-beta.0以降で利用可能です。

コンテナで動かしている場合には起動する際に--enable-feature=agentのオプションを追加することで動作します。

サーバに直インストールしている場合には以下を変更します。

注意：Prometheusのprometheus.ymlにてRemoteStorageの設定が入っていないと上記記載の通り、

　　　2時間分しか保持できませんので、事前にRemoteStorageの設定をしてから実行してください。

```bash
# vi /usr/lib/systemd/system/prometheus.service
# -*- mode: conf -*-

[Unit]
Description=The Prometheus monitoring system and time series database.
Documentation=https://prometheus.io
After=network.target

[Service]
EnvironmentFile=-/etc/default/prometheus
User=root #rootに変更します。
ExecStart=/usr/bin/prometheus $PROMETHEUS_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
## defaultで挿入されている--storage.tsdb.path=/var/lib/prometheus/dataは削除します。
# vi /etc/default/prometheus
PROMETHEUS_OPTS='--enable-feature=agent --config.file=/etc/prometheus/prometheus.yml --web.console.libraries=/usr/share/prometheus/console_libraries —web.console.templates=/usr/share/prometheus/consoles’
```

上記設定変更が完了したら、サービスの再起動を行います。

```bash
# systemctl restart prometheus
```

Agentモードにする前と後でMemory利用量が減っているので、テスト段階の機能ではありますが、

今後に期待したいと思います。

※Memory以外は大きくシステムリソースに変化はありませんでした。

※Grafanaで可視化してDashboardを作成しています。

※赤枠のガッツリ減っているところになります。

![Untitled](Prometheus%20Agent%E3%83%A2%E3%83%BC%E3%83%88%E3%82%99%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%206c1fa4dbee1945c8a22ea2a29f76e299/Untitled%201.png)