# Postgres Operator

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%20Zabbix%20proxy%20deploy%2011dc49f92c334b80ac69622e89be82e3/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

[kubeadm+HAProxy+Keepalivedで作る高可用性k8s環境](kubeadm+HAProxy+Keepalived%E3%81%A6%E3%82%99%E4%BD%9C%E3%82%8B%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7k8s%E7%92%B0%E5%A2%83%20ecbeb084af5c4aab92bd9e796a3b41ab.md) 

で記載したようにZalando Postgres Operatorを実施していきます。

# Zalanddo Postgres Operatorの実装

---

リポジトリは以下を利用します。

[https://github.com/zalando/postgres-operator.git](https://github.com/zalando/postgres-operator.git)

自分で調べた内容とNamespaceを変更したかったので、変更をかけてデプロイしております。

```bash
# kubectl create namespace postgres
# git clone https://github.com/zalando/postgres-operator.git
# cd postgres-operator

# vi manifests/configmap.yaml
enable_pod_antiaffinity: "true"

# kubectl create -f manifests/configmap.yaml -n postgres

# cat -n manifests/operator-service-account-rbac.yaml | grep namespace
     5    namespace: postgres
   188  # to get namespaces operator resources can run in
   192    - namespaces
   204  # to create ServiceAccounts in each namespace the operator watches
   242    namespace: postgres

# kubectl create -f manifests/operator-service-account-rbac.yaml

# cat manifests/postgres-operator.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-operator
  namespace: postgres
  labels:
    application: postgres-operator
spec:
  replicas: 1
  strategy:
    type: "Recreate"
  selector:
    matchLabels:
      name: postgres-operator
  template:
    metadata:
      labels:
        name: postgres-operator
    spec:
      serviceAccountName: postgres-operator
      containers:
      - name: postgres-operator
        image: registry.opensource.zalan.do/acid/postgres-operator:v1.8.2
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            cpu: 100m
            memory: 250Mi
          limits:
            cpu: 500m
            memory: 500Mi
        securityContext:
          runAsUser: 1000
          runAsNonRoot: true
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false
        env:
        # provided additional ENV vars can overwrite individual config map entries
        - name: CONFIG_MAP_NAME
          value: "postgres-operator"
        # In order to use the CRD OperatorConfiguration instead, uncomment these lines and comment out the two lines above
        # - name: POSTGRES_OPERATOR_CONFIGURATION_OBJECT
        #  value: postgresql-operator-default-configuration
        # Define an ID to isolate controllers from each other
        # - name: CONTROLLER_ID
        #   value: "second-operator"

# kubectl create -f manifests/postgres-operator.yaml

# cat -n ui/manifests/ui-service-account-rbac.yaml | grep namespace
     5    namespace: postgres
    50    - namespaces
    66    namespace: postgres 

# kubectl create -f ui/manifests/ui-service-account-rbac.yaml

# cat -n ui/manifests/deployment.yaml | grep namespace
     5    namespace: "postgres"

# kubectl create -f ui/manifests/deployment.yaml
```

Postgresqlリソースを宣言したマニフェストを用意して、デプロイします。

StorageClassの宣言はしておりませんが、SynologyNASのStorageClassをDefaultにしているので宣言をしておりません。

明示的に宣言をしたい場合には、volume欄にstorageClass:を追記します。

また外からアクセスするためにServiceをLoadBlanacerを有効にしております。

```bash
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  labels:
    team: acid
  name: acid-zabbix
  namespace: postgres
spec:
  allowedSourceRanges: []
  databases:
    zabbix: zabbix
  enableMasterLoadBalancer: true
  enableReplicaLoadBalancer: true
  numberOfInstances: 3 
  postgresql:
    version: '14'
  resources:
    limits:
      cpu: 500m
      memory: 500Mi
    requests:
      cpu: 100m
      memory: 100Mi
  teamId: acid
  users:
    zabbix: []
  volume:
    iops: 3000
    size: 100Gi
    throughput: 125
  allowedSourceRanges:
  - 0.0.0.0/0
  patroni:
    initdb:
      encoding: "UTF8"
      locale: "C"
      data-checksums: "true"
    pg_hba:
    - local   all all trust
    - hostssl all all 0.0.0.0/0 md5
    - host    all all 0.0.0.0/0 md5
    - host    all all ::1/128   md5
    - host    replication standby 0.0.0.0/0 md5

# kubectl create -f pgsql-zabbix.yaml 

## デプロイした後にpostgresnのDBへアクセスする際のパスワードを得ます。
# kubectl get secret -n postgres
# kubectl get secret -n postgres zabbix.acid-zabbix.credentials.postgresql.acid.zalan.do -o 'jsonpath={.data.password}' | base64 -d

```

これでPostgres Operatorで作成したPostgreSQLに接続することができました。