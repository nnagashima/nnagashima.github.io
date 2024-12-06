# k8s Grafana  deploy

![grafana_logo_icon_171048.png](k8s%20Grafana%20deploy%20785d8686a06a4eab8b5e3112f3b3d9c2/grafana_logo_icon_171048.png)

目次

# はじめに

---

VMで動かしていたGrafanaをk8s環境で実行します。

参考サイトは以下となります。

[Deploy Grafana on Kubernetes |  Grafana documentation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/)

# Grafanaをk8s環境へデプロイ

---

Configを作成します。

```jsx
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-pv
      volumes:
        - name: grafana-pv
          persistentVolumeClaim:
            claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  ports:
    - port: 3000
      protocol: TCP
      targetPort: http-grafana
  selector:
    app: grafana
  sessionAffinity: None
  type: LoadBalancer
```

作成したConfigをデプロイします。

```jsx
# kubectl create namespace grafana
# kubectl apply -f grafana.yaml -n grafana
persistentvolumeclaim/grafana-pvc created
deployment.apps/grafana created
service/grafana created
```

デプロイ結果を確認して以下のようになっていれば完了です。

```jsx
# kubectl get pods,svc,pv,pvc -n grafana
NAME                           READY   STATUS    RESTARTS   AGE
pod/grafana-7657b7f5d7-q6n7s   1/1     Running   0          6m20s

NAME              TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
service/grafana   LoadBalancer   10.107.226.127   192.168.21.104   3000:30783/TCP   6m20s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                STORAGECLASS             VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-3286d51d-b9c6-4321-8e35-e1c4fa7042e4   40Gi       RWX            Delete           Bound    vm/vmstorage-volume-vmcluster-victoria-metrics-cluster-vmstorage-1   synology-iscsi-storage   <unset>                          9d
persistentvolume/pvc-42501332-c18a-45d6-9bc4-7e81a35bdb8a   40Gi       RWX            Delete           Bound    palworld/palworld-server-datadir                                     synology-iscsi-storage   <unset>                          8d
persistentvolume/pvc-914fbdc8-4558-412a-a644-c26e4e45475a   1Gi        RWO            Delete           Bound    grafana/grafana-pvc                                                  synology-iscsi-storage   <unset>                          6m17s
persistentvolume/pvc-9ed6bd27-2a88-4d2a-9681-47d8c20e66fb   40Gi       RWX            Delete           Bound    vm/vmstorage-volume-vmcluster-victoria-metrics-cluster-vmstorage-0   synology-iscsi-storage   <unset>                          9d
persistentvolume/pvc-bcafa8dd-fdd5-449d-a122-5bc70fa03191   100Gi      RWO            Delete           Bound    postgres/pgdata-acid-zabbix-1                                        synology-iscsi-storage   <unset>                          34d
persistentvolume/pvc-ca343f51-8f9b-489d-b6f3-6de39e632993   100Gi      RWO            Delete           Bound    postgres/pgdata-acid-zabbix-0                                        synology-iscsi-storage   <unset>                          34d
persistentvolume/pvc-e23ae50f-2cb2-4cb4-8f66-2647a9c1125c   100Gi      RWO            Delete           Bound    postgres/pgdata-acid-zabbix-2                                        synology-iscsi-storage   <unset>                          34d

NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS             VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/grafana-pvc   Bound    pvc-914fbdc8-4558-412a-a644-c26e4e45475a   1Gi        RWO            synology-iscsi-storage   <unset>                 6m21s
```

# Grafanaへのアクセス

---

上記では「192.168.21.104」でロードバランサーでIPアドレスが割り当てられておりますので、

https://192.168.21.104:3000でアクセスをします。

ログインIDとパスワードはadmin / admin でアクセスします。