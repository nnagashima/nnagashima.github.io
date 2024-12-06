# azure-monitor-exporter

![prometheus-icon-511x512-1vmxbcxr (1).png](azure-monitor-exporter%20f825b33f244c4fe3a5b02425b542b391/prometheus-icon-511x512-1vmxbcxr_(1).png)

# 目次

---

# PrometheusでAzureMonitorのメトリクスを取得

---

Githubでexporterはいくつかあったのですが、更新が直近であるものとスターが多いものから、

以下を選択しました。

https://github.com/webdevops/azure-metrics-exporter

# 準備

---

Prometheusサーバ上でGO言語をインストールします。

上記exporterを利用するのに、1.19以上のバージョンが必要になります。

```bash
# curl -LO https://go.dev/dl/go1.19.5.linux-amd64.tar.gz
# tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
# echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bash_profile
# source .bash_profile
```

# Azure Monitor からAPIアクセスするための認証情報について

---

認証情報がなければAzureMonitorのメトリクスを取得することはできません。

そのため認証情報を作成するためにPrometheusサーバにazure-cliをインストールします。

```bash
# dnf install -y https://packages.microsoft.com/config/rhel/9.0/packages-microsoft-prod.rpm
# dnf install azure-cli
# az login
→AzurePortalにログインしているアカウントを利用します。
```

AZコマンドで認証情報を作成しますが、認証情報について注意点です。

年数を指定しない場合、有効期限は1年間のみとなります。

期限が切れると認証情報がなくなるため年数を指定します。（最大で99年）

ただし99年はCLIのみでできますが、UI上では最大で2年となります。

将来的にCLIでも99年指定はセキュリティの観点からできなくなるそうです。

https://jpazureid.github.io/blog/azure-active-directory/azuread-clientsecrets-202104/

認証情報を作成するAZコマンドは以下となります。

- createname：作成する名前を記載します。
- subscriptionid：AzureのサブスクリプションIDを記載します。
- resourcegroupname：対象となるリソースグループ名を記載します。

```bash
az ad sp create-for-rbac --name="${createname}" --role="Contributor" --scopes="/subscriptions/${subscriptionid}/resourceGroups/${resourcegroupname}”
```

これらのことから継続的にAzureMonitorからメトリクスを取得するためには、

Scriptなどで継続的に認証情報を更新する必要があると考えます。

そのため、こんなスクリプトを考えてみました。

```bash
#!/bin/sh
createname="test"
subscriptionid="subscription id"
resourcegroupname="resource group name"

az ad sp create-for-rbac --name="${createname}" --role="Contributor" --scopes="/subscriptions/${subscriptionid}/resourceGroups/${resourcegroupname}" > ./azure-rbac.log

AZURE_TENANT_ID=`cat ./azure-rbac.log | awk 'NR==5 {print $2}' | sed -e 's/,//g'`
AZURE_CLIENT_ID=`cat ./azure-rbac.log | awk 'NR==2 {print $2}' | sed -e 's/,//g'`
AZURE_CLIENT_SECRET=`cat ./azure-rbac.log | awk 'NR==4 {print $2}' | sed -e 's/,//g'`

cat << EOF > ./.profile
AZURE_TENANT_ID=$AZURE_TENANT_ID
AZURE_CLIENT_ID=$AZURE_CLIENT_ID
AZURE_CLIENT_SECRET=$AZURE_CLIENT_SECRET
export AZURE_TENANT_ID
export AZURE_CLIENT_ID
export AZURE_CLIENT_SECRET
EOF

logger "Azure rbac update" 

rm ./azure-rbac.log
```

上記だけだとProfileが再読み込みできておりません。

Shellに追記しましたがそれでも反映できなかったため、Cronで実行する際に以下のようにコマンドを追記しました。

```bash
# crontab -l
0 0 24 1 * /root/azure-rbac-create.sh && source .profile
```

# azure-monitor-exporterのセットアップ

---

exporterをコンパイルします。

```bash
# dnf install git make
# git clone https://github.com/webdevops/azure-metrics-exporter
# cd azure-metrics-exporter
# make
# ./azure-metrics-exporter
→実行できることを確認します

# cp -r azure-metrics-exporter /bin/
# azure-metrics-exporter
→実行できることを確認します
```

実行ができるようになったらPrometheusのConfigを作成するUIがありますので、

http://IPアドレス:8080/query へアクセスします。

必要な項目を各項目に入れて最後にExecute queryを実行することで、

Result欄にHTTP Statusや結果のBody、Configが表示されますので/etc/prometheus/prometheus.ymlに記載します。

なお、AzureMonitorのメトリクスは私個人の調べによると現時点(2023/01/25)では取得するにあたって、

料金などはかからないようですが今後変わるかもしれませんので気をつけてください。

# azure-monitor-exporterのサービス登録

---

azure-metrics-exporterがプログラムで動いているので、サービス登録して動かします。

```bash
# vi /etc/systemd/system/azure-metrics-exporter.service
[Unit]
Description = prometheus azure-monitor-exporter

[Service]
ExecStart = /usr/bin/azure-metrics-exporter 
Restart = always
Type = simple

[Install]
WantedBy = multi-user.target
```

サービスファイルを作成したらdaemon-reloadをしてサービスを起動します。

```bash
# systemctl daemon-reload
# systemctl start azure-metrics-exporter
# systemctl enable azure-metrics-exporter
```