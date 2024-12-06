# k8s Zabbix Server deploy

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

VMでZabbixを利用していましたがVersionUp対応など特定のバージョンで検証したいときに不便でしたので、

k8s環境で手軽に始められないかということで検索したら簡単にできそうだったので初めてみました。

# 使うもの

---

- k8s環境を用意してください。
- k8s環境でHelmが使える状態としてください。
- ZabbixのDatabaseには外部のPostgres（PostgresOperator）を利用します。
- 利用するHelmのレポジトリは以下です。

[https://github.com/zabbix-community/helm-zabbix](https://github.com/zabbix-community/helm-zabbix)

# Helm Repoへの追加

---

ZabbixCommunityに書いてある通りにRepojitoryを追加します。

```bash
# helm repo add zabbix-community https://zabbix-community.github.io/helm-zabbix
# helm repo update
```

# Zabbixの設定ファイルの取得

---

以下コマンドを使って、Zabbixのデフォルトパラメータを取得します。

```bash
# export ZABBIX_CHART_VERSION='4.0.2'
# helm show values zabbix-community/zabbix --version $ZABBIX_CHART_VERSION > ./zabbix_values.yaml
```

# Zabbixの設定ファイルの修正

---

差分は以下となります。

```bash
# diff -U0 zabbix_values_org.yaml zabbix_values.yaml
--- zabbix_values_org.yaml      2023-10-11 15:51:08.353709214 +0900
+++ zabbix_values.yaml  2023-10-11 20:19:10.022694258 +0900
@@ -31 +31 @@
-  useUnifiedSecret: true
+  useUnifiedSecret: false 
@@ -35 +35 @@
-  unifiedSecretAutoCreate: true
+  unifiedSecretAutoCreate: false 
@@ -41 +41 @@
-  host: "zabbix-postgresql"
+  host: 192.168.11.101
@@ -51 +51 @@
-  password: "zabbix"
+  password: "L0cmUJF3rSddauQ2KYPqBKWp4AIJSFgqna9gfUv7wlx8tPalgtVHiTNjy2KRqdn4"
@@ -106 +108 @@
-    type: ClusterIP
+    type: LoadBalancer 
@@ -143 +145 @@
-  enabled: true
+  enabled: false 
@@ -157 +159 @@
-    enabled: false
+    enabled: true
@@ -159 +161 @@
-    existingClaimName: false
+    existingClaimName: false 
@@ -161 +163 @@
-    storageSize: 5Gi
+    storageSize: 500Gi
@@ -163 +165 @@
-    #storageClass: my-storage-class
+    storageClass: synology-iscsi-storage
@@ -197 +199 @@
-  enabled: false
+  enabled: true
@@ -231 +233 @@
-    type: ClusterIP
+    type: LoadBalancer
@@ -352 +354 @@
-    type: ClusterIP
+    type: LoadBalancer 
@@ -357 +359 @@
-    loadBalancerIP: ""
+    loadBalancerIP: "192.168.11.110"
```

# Zabbixのデプロイ

---

DebugオプションをつけることでZabbixへパラメータが反映されているか確認ができるので、

Debugオプションをつけて筆者は実行しています。

あとはZabbixServerのLBのIPをつけているので、IPアドレスでWebUIへアクセスできれば完了です。

```bash
# helm upgrade --install zabbix zabbix-community/zabbix  --dependency-update  --create-namespace  --version $ZABBIX_CHART_VERSION  -f ./zabbix_values.yaml -n monitoring --debug
```

# デフォルトのままで放置していたら・・・

---

案の定Zabbix value cache working in low memory modeになってしまったので、

上記の設定ファイルに追加します。追加した場所がわかるように記載しています。ついでにConfigの同期時間を1分間隔にしています。

また自宅環境にVMwareもあるのでProxyの設定にVMwareCollectorsを起動させるようにしています。

```bash
▪️ ZabbixServerのENV
123   extraEnv:
124     #- name: ENABLE_TIMESCALEDB
125     #  value: "true"
126   - name: "ZBX_CACHESIZE"
127     value: 512M
128   - name: "ZBX_PROXYCONFIGFREQUENCY"
129     value: "60"

▪️ ZabbixProxyのENV
245   extraEnv:
246   - name: "ZBX_STARTVMWARECOLLECTORS"
247     value: "2"
248   - name: "ZBX_CONFIGFREQUENCY"
249     value: "60"
```

修正後、以下コマンドを実行します。

```bash
# helm upgrade --install zabbix zabbix-community/zabbix  --dependency-update  --create-namespace  --version $ZABBIX_CHART_VERSION  -f ./zabbix_values.yaml -n monitoring --debug
```

# 最後に

---

こんな感じでVMで起動しているZabbixからk8s環境へ移植することができました。

最新版の機能確認などはVMインストールするとして家庭内のインフラの監視はZabbixで、k8s環境はPrometheusで必要に応じて分けて実施していきます。

Zabbixはまだパラメータいじらないといけないところがあるのでいじっていきます。