# kube-prometheus-stackを利用してk8sをモニタリング

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

kube-prometheusはPrometheusOperatorの元に作られていますが、

kube-prometheus-stackはPrometheusCommunityの元に作られている違いがあるので、

PrometheusOperatorのサポートがあるかないかがあります。

# kube-prometheus-stackついて

---

kube-prometheusはPrometheus、AlertManger、Grafanaをまとめて用意してくれて、

モニタリングの例を一気にインストールしてくれます。

[GitHub - prometheus-community/helm-charts: Prometheus community Helm charts](https://github.com/prometheus-community/helm-charts)

# kube-prometheus-stackのデプロイ

---

kube-prometheus-stackのリポジトリを追加します。

```bash
# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# helm repo update
```

kube-prometheus-stackをデプロイする前に設定ファイルを出力します。

```jsx
# kubectl create ns monitoring
# helm show values prometheus-community/kube-prometheus-stack > kube-prometheus-stack.yaml
# helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f kube-prometheus-stack.yaml -n monitoring
NAME: kube-prometheus-stack
LAST DEPLOYED: Thu Jan 11 20:57:13 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=kube-prometheus-stack"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

※kube-prometheus-stack.yaml

```jsx
# diff -U0 kube-prometheus-stack.yaml.org kube-prometheus-stack.yaml
```

デプロイ後の結果を確認します。

```bash
# kubectl get all -n monitoring
NAME                                                            READY   STATUS    RESTARTS   AGE
pod/alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   0          27s
pod/kube-prometheus-stack-grafana-5698997d4b-qc56z              3/3     Running   0          31s
pod/kube-prometheus-stack-kube-state-metrics-8444bb469d-sgk8d   1/1     Running   0          31s
pod/kube-prometheus-stack-operator-69c7c996ff-858bd             1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-7nqwf        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-8ch7b        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-dj9k6        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-gp9jc        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-kg4cv        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-sgczf        1/1     Running   0          31s
pod/kube-prometheus-stack-prometheus-node-exporter-x29dc        1/1     Running   0          31s
pod/prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          27s

NAME                                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                            ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   28s
service/kube-prometheus-stack-alertmanager               ClusterIP   10.100.121.83    <none>        9093/TCP,8080/TCP            33s
service/kube-prometheus-stack-grafana                    ClusterIP   10.102.171.170   <none>        80/TCP                       33s
service/kube-prometheus-stack-kube-state-metrics         ClusterIP   10.108.106.248   <none>        8080/TCP                     33s
service/kube-prometheus-stack-operator                   ClusterIP   10.98.70.254     <none>        443/TCP                      33s
service/kube-prometheus-stack-prometheus                 ClusterIP   10.108.253.237   <none>        9090/TCP,8080/TCP            33s
service/kube-prometheus-stack-prometheus-node-exporter   ClusterIP   10.106.143.131   <none>        9100/TCP                     33s
service/prometheus-operated                              ClusterIP   None             <none>        9090/TCP                     28s

NAME                                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-prometheus-stack-prometheus-node-exporter   7         7         7       7            7           kubernetes.io/os=linux   32s

NAME                                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kube-prometheus-stack-grafana              1/1     1            1           32s
deployment.apps/kube-prometheus-stack-kube-state-metrics   1/1     1            1           32s
deployment.apps/kube-prometheus-stack-operator             1/1     1            1           32s

NAME                                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/kube-prometheus-stack-grafana-5698997d4b              1         1         1       32s
replicaset.apps/kube-prometheus-stack-kube-state-metrics-8444bb469d   1         1         1       32s
replicaset.apps/kube-prometheus-stack-operator-69c7c996ff             1         1         1       32s

NAME                                                               READY   AGE
statefulset.apps/alertmanager-kube-prometheus-stack-alertmanager   1/1     28s
statefulset.apps/prometheus-kube-prometheus-stack-prometheus       1/1     28s
```

これでPrometheus、AlertManger、Grafanaがデプロイできました。

ここから設定ファイルをカスタマイズします。

```jsx
# vi kube-prometheus-stack-origin.yaml
prometheus:
  service:
    type: LoadBalancer
alertmanager:
  service:
    type: LoadBalancer
grafana:
  deploymentStrategy:
    type: Recreate
  persistence:
    enabled: true
    type: pvc
    storageClassName: synology-iscsi-storage
    accessModes:
    - ReadWriteMany
    size: 20Gi
    finalizers:
    - kubernetes.io/pvc-protection
  # Admin user password
  adminPassword: "password"
  defaultDashboardsTimezone: Asia/Tokyo
  service:
    type: LoadBalancer
```

カスタマイズファイルを使ってアップグレードします。

```jsx
# helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack -f kube-prometheus-stack-orign.yaml -n monitoring
Release "kube-prometheus-stack" has been upgraded. Happy Helming!
NAME: kube-prometheus-stack
LAST DEPLOYED: Thu Jan 11 21:19:55 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 2
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=kube-prometheus-stack"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

デプロイ後の結果を確認します。

```jsx
# kubectl get all -n monitoring
NAME                                                            READY   STATUS    RESTARTS        AGE
pod/alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   2 (5h31m ago)   12d
pod/kube-prometheus-stack-grafana-5b7d8df4c8-829d8              3/3     Running   3 (5h31m ago)   12d
pod/kube-prometheus-stack-kube-state-metrics-8444bb469d-ghwpf   1/1     Running   3 (5h28m ago)   12d
pod/kube-prometheus-stack-operator-69c7c996ff-d8wfq             1/1     Running   1 (5h30m ago)   12d
pod/kube-prometheus-stack-prometheus-node-exporter-2jf9b        1/1     Running   1 (9d ago)      12d
pod/kube-prometheus-stack-prometheus-node-exporter-54w46        1/1     Running   1 (5h30m ago)   12d
pod/kube-prometheus-stack-prometheus-node-exporter-96t2s        1/1     Running   1 (5h31m ago)   12d
pod/kube-prometheus-stack-prometheus-node-exporter-bg8jm        1/1     Running   1 (5h30m ago)   12d
pod/kube-prometheus-stack-prometheus-node-exporter-fl9d6        1/1     Running   1 (9d ago)      12d
pod/kube-prometheus-stack-prometheus-node-exporter-kk66z        1/1     Running   1 (5h30m ago)   12d
pod/kube-prometheus-stack-prometheus-node-exporter-vpp5b        1/1     Running   1 (9d ago)      12d
pod/kube-prometheus-stack-prometheus-node-exporter-wfd4b        1/1     Running   1 (5h32m ago)   12d
pod/prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   2 (5h30m ago)   12d

NAME                                                     TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                         AGE
service/alertmanager-operated                            ClusterIP      None             <none>           9093/TCP,9094/TCP,9094/UDP      12d
service/kube-prometheus-stack-alertmanager               LoadBalancer   10.107.2.4       192.168.21.104   9093:30728/TCP,8080:32680/TCP   12d
service/kube-prometheus-stack-grafana                    LoadBalancer   10.98.134.88     192.168.21.105   80:30500/TCP                    12d
service/kube-prometheus-stack-kube-state-metrics         ClusterIP      10.105.186.117   <none>           8080/TCP                        12d
service/kube-prometheus-stack-operator                   ClusterIP      10.102.126.207   <none>           443/TCP                         12d
service/kube-prometheus-stack-prometheus                 LoadBalancer   10.99.38.40      192.168.21.106   9090:32722/TCP,8080:30976/TCP   12d
service/kube-prometheus-stack-prometheus-node-exporter   ClusterIP      10.108.71.10     <none>           9100/TCP                        12d
service/prometheus-operated                              ClusterIP      None             <none>           9090/TCP                        12d

NAME                                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-prometheus-stack-prometheus-node-exporter   8         8         8       8            8           kubernetes.io/os=linux   12d

NAME                                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kube-prometheus-stack-grafana              1/1     1            1           12d
deployment.apps/kube-prometheus-stack-kube-state-metrics   1/1     1            1           12d
deployment.apps/kube-prometheus-stack-operator             1/1     1            1           12d

NAME                                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/kube-prometheus-stack-grafana-5b7d8df4c8              1         1         1       12d
replicaset.apps/kube-prometheus-stack-kube-state-metrics-8444bb469d   1         1         1       12d
replicaset.apps/kube-prometheus-stack-operator-69c7c996ff             1         1         1       12d

NAME                                                               READY   AGE
statefulset.apps/alertmanager-kube-prometheus-stack-alertmanager   1/1     12d
statefulset.apps/prometheus-kube-prometheus-stack-prometheus       1/1     12d
```

```jsx
# kubectl get pv,pvc -n monitoring
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                STORAGECLASS             VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-e3b791cb-0e34-4bab-b443-e66088663cea   20Gi       RWO            Delete           Bound    monitoring/kube-prometheus-stack-grafana                             synology-iscsi-storage   <unset>                          12d

NAME                                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS             VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/kube-prometheus-stack-grafana   Bound    pvc-e3b791cb-0e34-4bab-b443-e66088663cea   20Gi       RWO            synology-iscsi-storage   <unset>                 12d
```

これでGrafanaをローカルIPで公開ができており、Grafanaのデータも永続化することができました。

Kubernetes環境は本構成を使ってこれからは監視し、ホストなどの監視はNewrelicとZabbixを利用していきたいと思います。