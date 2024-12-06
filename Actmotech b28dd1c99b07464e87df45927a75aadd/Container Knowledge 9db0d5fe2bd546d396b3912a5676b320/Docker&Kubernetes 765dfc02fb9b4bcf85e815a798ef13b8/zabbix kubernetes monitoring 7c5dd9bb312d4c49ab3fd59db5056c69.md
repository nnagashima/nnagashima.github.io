# zabbix kubernetes monitoring

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

Zabbix6.0からKubernetesモニタリングのテンプレートが公開されたので、どんなことを診ることができるのか確認します。

本監視テンプレートは外部スクリプトなし、API サーバー /metrics エンドポイントから HTTP エージェントによって、

メトリックを収集でKubernetesノードを監視することができます。

# 存在しているテンプレート

---

- Kubernetes API server by HTTP
- Kubernetes cluster state by HTTP
- Kubernetes Controller manager by HTTP
- Kubernetes Kubelet by HTTP
- Kubernetes nodes by HTTP
- Kubernetes Scheduler by HTTP

なお、Kubernetes cluster state by HTTPで以下がディスカバリーされるようです。

- Kubernetes API server by HTTP
- Kubernetes Controller manager by HTTP
- Kubernetes Kubelet by HTTP
- Kubernetes Scheduler by HTTP

テンプレートに当てる際に$KUBE.API.TOKENが必要になるので、以下コマンドを実行して取得します。

```bash
 # kubectl get secret zabbix-service-account -n zabbix -o jsonpath={.data.token} | base64 -d
```

# Kubernetes nodes by HTTP テンプレート適用

---

ノード監視用のテンプレート「Kubernetes nodes by HTTP」適用するために、ホスト登録します。

※ray-k8s-nodeで筆者は登録しております。

登録したらテンプレートを適用し、ホストマクロを設定します。

| マクロ | 値 |
| --- | --- |
| {$KUBE.API.ENDPOINT} | <scheme>://<host>:<port>/api
→筆者環境ではhttps://192.168.11.11/api |
| {$KUBE.API.TOKEN} | kubectl get secret zabbix-service-account -n zabbix -o jsonpath={.data.token} | base64 -dでGETしたTOKENになります。 |

ノードディスカバリーによって、Workerノードが検出されております。

![Untitled](zabbix%20kubernetes%20monitoring%207c5dd9bb312d4c49ab3fd59db5056c69/Untitled.png)

Masterノードを検出できない理由としては[k8s Zabbix proxy deploy](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3.md) でDeployしたZabbixAgentの設定が以下になっているのに対して、

```bash
## Tolerations configurations
  tolerations:
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
```

Masterノードの設定情報をみると、

```bash
# kubectl describe node/ray-k8s-master01
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

となっているため、不一致によりMasterノードにZabbixAgentが展開されませんでしたので、

zabbix_values.yamlを修正してUpdateします。

補足：

node-role.kubernetes.io/masterはKubernetes1.20で非推奨になっており、削除される予定となっております。

Kubernetesは3ヶ月ごとにリリースされますので、ReleaseNoteを常に追っていく必要があるので運用が大変です。。。

[enhancements/README.md at master · kubernetes/enhancements](https://github.com/kubernetes/enhancements/blob/master/keps/sig-cluster-lifecycle/kubeadm/2067-rename-master-label-taint/README.md)

修正箇所

```bash
## Tolerations configurations
  tolerations:
    - effect: NoSchedule
      key: node-role.kubernetes.io/control-plane
```

Update

```bash
# helm upgrade zabbix . --dependency-update -f ./zabbix_values.yaml -n zabbix
```

補足：

kubernetesのMasternノードにはTaintsが設定されており、Podが展開されないようになっております。

そのためMasterノードにPodを展開するためのやり方は２つあり、

MasterノードのTaint情報を剥がす方法と、Pod側で対応する場合のやり方があります。

今回はPod側で制御する後者のやり方を取りました。

※MasterにPod展開することって本来ないと思っているので、本来のやり方ではないかと思っています。。。

ノードディスカバリーによって、Masterノードが検出されております。

![Untitled](zabbix%20kubernetes%20monitoring%207c5dd9bb312d4c49ab3fd59db5056c69/Untitled%201.png)

# Kubernetes cluster state by HTTP テンプレート適用

---

Kubernetesのコンポーネントを監視します。

クラスタ監視用のテンプレート「Kubernetes cluster state by HTTP」適用するために、ホスト登録します。

※ray-k8s-clusterで筆者は登録しております。

登録したらテンプレートを適用し、ホストマクロを設定します。

| マクロ | 値 |
| --- | --- |
| {$KUBE.API.HOST} | APIのIP、筆者環境では192.168.11.11 |
| {$KUBE.API.TOKEN} | kubectl get secret zabbix-service-account -n zabbix -o jsonpath={.data.token} | base64 -dでGETしたTOKENになります。 |
| {$KUBE.API.CERT.EXPIRATION}  | 90
デフォルトは7 |

{$KUBE.API.CERT.EXPIRATION} は、Kubernetes API server by HTTPテンプレートのMacroに該当します。

デフォルトでは7日前に証明書期限切れの通知をしてくれますが、早めに欲しいため90日前にしました。

ディスカバリーによって、各コンポーネントがホストで自動登録されています。

![Untitled](zabbix%20kubernetes%20monitoring%207c5dd9bb312d4c49ab3fd59db5056c69/Untitled%202.png)

# 監視をすることはできたが。。。

---

Kubernetesの監視をZabbix6で、できることはできたが環境に合わせて必要な監視テンプレート追加などの検討も必要という印象を受けた。

また可視化しないとアイテム数が膨大なので自分の見やすい画面を作る必要があると感じている。

Kube-PrometheusなどはDashboardが用意されていて可視化まですぐにできるので、Prometheusの方が導入から運用スタートまで、

早くできるところが良いなと感じる一方で、こうやってUIで操作できるのがZabbixの特徴でもあるのでどちらも捨てがたい。