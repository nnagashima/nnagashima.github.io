# k8s Zabbix proxy deploy

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

VirtualMachineで動いているZabbix Proxyをkubernetes基盤で移行します。

前提としてkubernetes基盤があること、Helmを実行できるようになっているところがスタート地点になります。

# Deploy Zabbbix in Kubernets

---

GithubでKubernetesにZabbixProxyをデプロイするためのやり方やコードが公開されているので、

そちらを利用します。

```bash
# git clone https://git.zabbix.com/scm/zt/kubernetes-helm.git
# cd kubernetes-helm
```

Defaultの設定をOUTPUTします。

OUTPUTしたら各々の環境に合わせてパラメータを修正したりしてください。

筆者はZabbixProxyのimageを変えたり、Proxyをパッシブモードにしたりしています。

https://git.zabbix.com/projects/zt/repos/kubernetes-helm/browse

```bash
# helm show values . > ./zabbix_value.yaml
```

デプロイ先のNamespaceを作成します。

```bash
# kubectl create namespace zabbix
```

ZabbixProxyをデプロイします。

```bash
# helm install zabbix . --dependency-update -f ./zabbix_values.yaml -n zabbix
Release "zabbix" has been upgraded. Happy Helming!
NAME: zabbix
LAST DEPLOYED: Mon Apr 17 09:53:17 2023
NAMESPACE: zabbix
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing zabbix-helm-chrt.

Your release is named zabbix.
Zabbix agent installed:  "zabbix/zabbix-agent2:6.0-ubuntu-latest"
Zabbix proxy installed:  "zabbix/zabbix-proxy-sqlite3:6.0-ubuntu-latest"

Annotations:
app.kubernetes.io/name: zabbix-zabbix-helm-chrt
helm.sh/chart: zabbix-helm-chrt-1.1.2
app.kubernetes.io/version: "trunk"
app.kubernetes.io/managed-by: Helm

Service account created: 
    zabbix-service-account

To learn more about the release, try:

  $ helm status zabbix -n monitoring
  $ helm get all zabbix -n monitoring
```

Podの状況を確認します。

```bash
# kubectl get pod -n zabbix
NAME                                         READY   STATUS    RESTARTS      AGE
zabbix-agent-gnhzk                           1/1     Running   2 (24m ago)   3d8h
zabbix-agent-hgq8s                           1/1     Running   3 (17m ago)   3d8h
zabbix-agent-nk7br                           1/1     Running   2 (24m ago)   3d8h
zabbix-kube-state-metrics-769b9b6b8c-cdz5q   1/1     Running   3 (15m ago)   2d17h
zabbix-proxy-5b6df9467d-w4k55                1/1     Running   1 (25m ago)   2d17h

# kubectl get svc -n zabbix
NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)           AGE
zabbix-kube-state-metrics       ClusterIP      10.106.38.42    <none>           8080/TCP          3d8h
zabbix-zabbix-helm-chrt-agent   ClusterIP      10.99.0.244     <none>           10050/TCP         3d8h
zabbix-zabbix-helm-chrt-proxy   LoadBalancer   10.98.173.225   192.168.11.153   10051:31304/TCP   3d8h
```

作成できたら、ZabbixServerにZabbixProxyを登録します。

ZabbixServerのUIにログインして「管理」→「プロキシ」で作成したプロキシ名で登録します。

プロキシ名はzabbix_value.yamlの44行目が該当します。

# Uninstall Zabbbix in Kubernets

---

以下コマンドを利用してアンインストールします。

```bash
# helm delete zabbix -n zabbix
```