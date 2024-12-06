# PrometheusのUIをパスワード認証化させる方法

![prometheus-icon-511x512-1vmxbcxr (1).png](Prometheus%E3%81%AEUI%E3%82%92%E3%83%8F%E3%82%9A%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%88%E3%82%99%E8%AA%8D%E8%A8%BC%E5%8C%96%E3%81%95%E3%81%9B%E3%82%8B%E6%96%B9%E6%B3%95%2001c759f6ae734ebbbf7f80c17e89fd7c/prometheus-icon-511x512-1vmxbcxr_(1).png)

# 目次

---

# Prometheus UIについて

---

Prometheusの画面は認証設定がデフォルトではありませんが、

設定することでIDとパスワードでの認証することができます。

下記ガイドに従って対応すれば可能です。

https://prometheus.io/docs/guides/basic-auth/

# Prometheus UI の認証設定

---

/etc/prometheus配下に、webui.yqmlを作成します。

```bash
basic_auth_users:
    ino-admin: $2b$12$hNf2lSsxfm0.i4a.1kVpSOVyBCfIB51VRjgBUyv6kdnyTlgWj81Ay
```

/etc/default/prometheusに--web.config.file=/etc/prometheus/webui.yamlを記載します。

```bash
PROMETHEUS_OPTS='--web.config.file=/etc/prometheus/webui.yaml --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/data --web.console.libraries=/usr/share/prometheus/console_libraries —web.console.templates=/usr/share/prometheus/consoles'
```

Prometheusのwebui.yamlのConfigチェックをします。

```bash
# promtool check web-config /etc/prometheus/webui.yaml
/etc/prometheus/webui.yaml SUCCESS
```

認証設定をすることでPrometheus自身が監視できなくなるため、

prometheus.ymlに設定を追加します。（basic_auth以下が追加したものです。）

```bash
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "ray-prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: 
        - 'localhost:9090'
        - 'localhost:9100'
    basic_auth:
      username: ino-admin
      password: test
```

Prometheusサービスの再起動を実施します。

```bash
# systemctl restart prometheus
```

# 認証設定の確認

---

http://IPアドレス:9090にアクセスし、IDとパスワードが求められログインできれば設定完了です。