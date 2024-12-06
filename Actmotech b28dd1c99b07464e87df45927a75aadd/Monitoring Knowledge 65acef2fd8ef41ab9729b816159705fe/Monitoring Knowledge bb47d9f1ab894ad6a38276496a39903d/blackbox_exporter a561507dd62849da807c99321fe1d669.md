# blackbox_exporter

![prometheus-icon-511x512-1vmxbcxr (1).png](blackbox_exporter%20a561507dd62849da807c99321fe1d669/prometheus-icon-511x512-1vmxbcxr_(1).png)

# 目次

---

# blackbox_exporterとは

---

死活監視に利用するexporterになり、ICMPやHTTPによる外型監視を実現できます。

# blackbox_exporterのインストール

---

利用するOSはRokcyLinux9になります。

Prometheusサーバ上で実行します。

blackbox_exporterのバイナリを持ってきて、exporterをコンパイルします。※GO言語が取り扱える状態にしといてください。

```bash
# useradd --no-create-home --shell /bin/false blackbox_exporter
# curl -LO https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
# tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
# cd blackbox_exporter-0.25.0.linux-amd64
# cp -p blackbox_exporter /usr/local/bin/blackbox_exporter
# chown blackbox_exporter:blackbox_exporter /usr/local/bin/blackbox_exporter
# mkdir /etc/blackbox_exporter
# vi /etc/blackbox_exporter/blackbox.yml
modules:
  http_2xx:
    prober: http
    http:
      method: GET
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: []  # Defaults to 2xx
      no_follow_redirects: false  
      preferred_ip_protocol: "ip4"
      ip_protocol_fallback: false 
      fail_if_ssl: false
      fail_if_not_ssl: false 
      tls_config:
        insecure_skip_verify: true 
  http_3xx:
    prober: http
    http:
      method: GET
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: []  # Defaults to 2xx
      no_follow_redirects: false
      preferred_ip_protocol: "ip4"
      ip_protocol_fallback: false
      fail_if_ssl: false
      fail_if_not_ssl: true
      tls_config:
        insecure_skip_verify: true 
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  grpc:
    prober: grpc
    grpc:
      tls: true
      preferred_ip_protocol: "ip4"
  grpc_plain:
    prober: grpc
    grpc:
      tls: false
      service: "service1"
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
      - send: "SSH-2.0-blackbox-ssh-check"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
    icmp:
      preferred_ip_protocol: "ip4"
  icmp_ttl5:
    prober: icmp
    timeout: 5s
    icmp:
      ttl: 5

# chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter/blackbox.yml
# vi /etc/systemd/system/blackbox_exporter.service
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file /etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target

# systemctl daemon-reload
# systemctl start blackbox_exporter
```

# blackbox_exporterの設定

---

/etc/prometheus/blackbox.ymlで必要モジュールを記載

[blackbox_exporter/CONFIGURATION.md at master · prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md)

/etc/prometheus/prometheus.ymlに監視ターゲットを追加します。

例：ICMPの場合

```bash
- job_name: 'blackbox_icmp_v4'
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
        - 監視対象ホスト名 or IPアドレス
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox_exporterが動いているサーバホスト名 or IPアドレス:9115
```

例：httpの応答コード200の場合

```bash
- job_name: 'blackbox_http'
    metrics_path: /probe
    params:
      module: [http_2xx,http_3xx]
    static_configs:
      - targets:
        - http://監視対象ホスト名 or IPアドレス/パス名
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox_exporterが動いているサーバホスト名 or IPアドレス:9115
```

例：TCPポートの場合

```bash
- job_name: 'blackbox_tcp'
    metrics_path: /probe
    params:
      module: [tcp]
    static_configs:
      - targets:
        - 監視対象ホスト名 or IPアドレス:監視対象ポート番号
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox_exporterが動いているサーバホスト名 or IPアドレス:9115
```

blackbox_exporterサービスを再起動します。

```bash
## サービスの起動
# systemctl restart blackbox_exporter
```

# 監視できているか確認

---

http://IPアドレス:9090へアクセスし、ターゲットがUPかつ各ターゲットのProbeStatusが1になっていることを確認します。

ターゲットがUPしているかは画面の上のメニューのStatus→Targetsをクリックします。

各ターゲットのProbeStatusを確認するには角URLリンクをクリックして確認か、PromQLで検索します。

![Untitled](blackbox_exporter%20a561507dd62849da807c99321fe1d669/Untitled.png)

![Untitled](blackbox_exporter%20a561507dd62849da807c99321fe1d669/Untitled%201.png)

![Untitled](blackbox_exporter%20a561507dd62849da807c99321fe1d669/Untitled%202.png)

ICMPのProbeStatusが1になっていない場合には、以下を実行してblackbox_exporterを再起動してみてください。

```bash
$ sudo setcap cap_net_raw+ep /path/top/blackbox_exporter
```