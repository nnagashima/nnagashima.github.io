# kube-prometheusを利用してk8sをモニタリング

![kubernetes-icon-2048x1995-r1q3f8n7.png](kube-prometheus%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6k8s%E3%82%92%E3%83%A2%E3%83%8B%E3%82%BF%E3%83%AA%E3%83%B3%E3%82%AF%E3%82%99%2003afb36538464c539f912934022fbba2/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

kubernetesで実行している環境で何かあった時に気づけるようにする仕組みとして、

Prometheusを利用したいと思います。

# kube-prometheusについて

---

kube-prometheusはPrometheus、AlertManger、Grafanaをまとめて用意してくれて、

モニタリングの例を一気にインストールしてくれます。

なお、本リポジトリはprometheus-operatorが管理しているプロジェクトになります。

[https://github.com/prometheus-operator/kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)

# kube-prometheusのデプロイ

---

Quickstartの通りに進めていきます。

```bash
# git clone https://github.com/coreos/kube-prometheus.git
# cd kube-prometheus
# cp -r manifests manifests_org
# cd manifests
# kubectl apply --server-side -f setup/
# kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
# kubectl apply -f ./
```

デプロイ後の結果を確認します。

```bash
# kubectl get all -n monitoring
NAME                                       READY   STATUS    RESTARTS       AGE
pod/alertmanager-main-0                    2/2     Running   8 (22h ago)    34h
pod/alertmanager-main-1                    2/2     Running   4 (22h ago)    34h
pod/alertmanager-main-2                    2/2     Running   0              22h
pod/blackbox-exporter-76b5c44577-9f8c7     3/3     Running   6 (22h ago)    4d17h
pod/grafana-69f6b485b9-74689               0/1     Running   0              16s
pod/kube-state-metrics-cff77f89d-7phgq     3/3     Running   7 (22h ago)    4d17h
pod/node-exporter-47m7s                    2/2     Running   6 (23h ago)    12d
pod/node-exporter-bb66t                    2/2     Running   8 (23h ago)    12d
pod/node-exporter-n9psw                    2/2     Running   8 (22h ago)    12d
pod/node-exporter-qnfpx                    2/2     Running   4 (22h ago)    24h
pod/node-exporter-rz68c                    2/2     Running   6 (24h ago)    12d
pod/node-exporter-swsc6                    2/2     Running   8 (23h ago)    12d
pod/node-exporter-vf4tl                    2/2     Running   8 (23h ago)    12d
pod/node-exporter-z8lwj                    2/2     Running   10 (22h ago)   12d
pod/prometheus-adapter-74894c5547-r4pfg    1/1     Running   3 (22h ago)    4d17h
pod/prometheus-adapter-74894c5547-z57l2    1/1     Running   3 (22h ago)    4d17h
pod/prometheus-k8s-0                       2/2     Running   8 (22h ago)    12d
pod/prometheus-k8s-1                       2/2     Running   6 (22h ago)    34h
pod/prometheus-operator-57757d758c-cf8xr   2/2     Running   7 (22h ago)    4d17h

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-main       ClusterIP   10.101.2.229     <none>        9093/TCP,8080/TCP            12d
service/alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   12d
service/blackbox-exporter       ClusterIP   10.107.148.60    <none>        9115/TCP,19115/TCP           12d
service/grafana                 ClusterIP   10.101.186.199   <none>        3000/TCP                     16s
service/kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP            12d
service/node-exporter           ClusterIP   None             <none>        9100/TCP                     12d
service/prometheus-adapter      ClusterIP   10.103.118.139   <none>        443/TCP                      12d
service/prometheus-k8s          ClusterIP   10.104.49.95     <none>        9090/TCP,8080/TCP            12d
service/prometheus-operated     ClusterIP   None             <none>        9090/TCP                     12d
service/prometheus-operator     ClusterIP   None             <none>        8443/TCP                     12d

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-exporter   8         8         8       8            8           kubernetes.io/os=linux   12d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blackbox-exporter     1/1     1            1           12d
deployment.apps/grafana               0/1     1            0           16s
deployment.apps/kube-state-metrics    1/1     1            1           12d
deployment.apps/prometheus-adapter    2/2     2            2           12d
deployment.apps/prometheus-operator   1/1     1            1           12d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/blackbox-exporter-76b5c44577     1         1         1       12d
replicaset.apps/grafana-69f6b485b9               1         1         0       16s
replicaset.apps/kube-state-metrics-cff77f89d     1         1         1       12d
replicaset.apps/prometheus-adapter-74894c5547    2         2         2       12d
replicaset.apps/prometheus-operator-57757d758c   1         1         1       12d

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   3/3     12d
statefulset.apps/prometheus-k8s      2/2     12d
```

これでPrometheus、AlertManger、Grafanaがデプロイできました。

GrafanaをNodePortで解放してID：admin、Password ：adminでログインでき、

GrafanaでDashboardがインポートされているため確認できれば完了となります。

Grafanaを別で起動させたい場合には、Grafana公式のドキュメントもあるのでそちらを参照してください。

当環境ではのちにkube-prometheusのGrafanaは削除して、別のGrafanaでkube-prometheusのデータ可視化をしております。

https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/

# kube-prometheusのPrometheusをLB経由で公開

---

prometheus-service.yamlに**type: LoadBalancer**を追記して適用します。

```jsx
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
  **type: LoadBalancer**
```

NetworkPolicyを作成して適用させます。

```jsx
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: prometheus-allow-external
  namespace: monitoring
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
  ingress:
  - ports:
    - port: 9090
```

上記適用後に、EXTERNAL-IPにIPアドレスが割り当てられてURLアクセスができればOKです。

```jsx
# kubectl get svc -n monitoring
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                         AGE
alertmanager-main       ClusterIP      10.100.111.17    <none>           9093/TCP,8080/TCP               74m
alertmanager-operated   ClusterIP      None             <none>           9093/TCP,9094/TCP,9094/UDP      73m
blackbox-exporter       ClusterIP      10.111.250.73    <none>           9115/TCP,19115/TCP              74m
kube-state-metrics      ClusterIP      None             <none>           8443/TCP,9443/TCP               74m
node-exporter           ClusterIP      None             <none>           9100/TCP                        74m
prometheus-adapter      ClusterIP      10.102.209.100   <none>           443/TCP                         74m
prometheus-k8s          LoadBalancer   10.107.16.208    192.168.21.105   9090:30132/TCP,8080:32471/TCP   74m
prometheus-operated     ClusterIP      None             <none>           9090/TCP                        73m
prometheus-operator     ClusterIP      None             <none>           8443/TCP                        74m
```

# CoreDNSのモニタリング

---

prometheus-serviceMonitor.yamlに以下を追加して適用します。

適用後Prometheusの画面にてTargetからCoreDNSのエンドポイントが見れていればOKです。

```jsx
endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    interval: 15s
    port: metrics
  namespaceSelector:
    matchNames:
    - coredns
  selector:
    matchLabels:
      release: coredns
```