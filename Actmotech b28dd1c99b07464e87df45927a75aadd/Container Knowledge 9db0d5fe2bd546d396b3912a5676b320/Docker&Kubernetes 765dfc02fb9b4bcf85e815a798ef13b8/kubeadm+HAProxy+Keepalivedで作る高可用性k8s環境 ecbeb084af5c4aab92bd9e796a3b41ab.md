# kubeadm+HAProxy+Keepalivedで作る高可用性k8s環境

![kubernetes-icon-2048x1995-r1q3f8n7.png](kubeadm+HAProxy+Keepalived%E3%81%A6%E3%82%99%E4%BD%9C%E3%82%8B%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7k8s%E7%92%B0%E5%A2%83%20ecbeb084af5c4aab92bd9e796a3b41ab/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# はじめに

---

HAProxy+KeepAlivedを利用した高可用性のk8s環境を作ります。 構成パターンは以下です。

- k8sのMasterはRasbbery Pi4とx86 ProxmoxのVMで構築、Workerはx86 ProxmoxのVMで動かします。
- OSはUbuntu24.04で構築し動かします。
- NASはSynologyのCSIドライバーを利用したかったため、SynologyNASを購入しました

# 利用OS

---

- OS：Ubuntu24.04
- k8s：1.30.2

# 構成

---

- 192.168.21.30 KeepAlivedのVIP
- 192.168.21.31 / 192.168.21. 32 / 192.168.21.33 / 192.168.21.34 / 192.168.21.35：コントロールプレインノード
- 192.168.21.41 / 192.168.21.42 / 192.168.21.43：ワーカーノード

# 事前準備

---

OSのPKGを最新化します。

```bash
# apt update && apt upgrade -y
```

ホスト名を設定します。

```bash
# hostnamectl set-hostname host名
```

# SWAPの無効化

---

- 対象：RasberryPiの場合

RasberryPiはcgroupsの設定をするために、カーネルパラメータを編集します。

Controle Groupの略で、プロセスをグループ化して、利用を制限をかけることができるLinuxのカーネル機能になります。

cgroupがk8sでいくつか利用されているので、設定有効化後にOS再起動します。

```bash
# swapoff -a
# sed -i 's/$/ cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory/g' /boot/cmdline.txt 
# reboot
```

- 対象：x86系の場合

以下の通り実行します。

```bash
# swapoff -a
# vi /etc/fstab
swapの記載があれば無効にする
```

# MicroSDからSSDへの書き込みを実行し起動

---

RasberryPiの場合、MicroSDだと起動が不安定になるので、SSDに書き込みします。

もちろんSDからコピーするのではなく最初からUSBのSSDにOS書き込んで起動でも大丈夫です。

なお、RassberryPiとSSDの接続のコードは相性があります。私は以下コードを購入しております。

https://www.amazon.co.jp/gp/product/B00HJZJI84/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1

```bash
# git clone https://github.com/billw2/rpi-clone.git
# cd rpi-clone/
# cp rpi-clone rpi-clone-setup /usr/local/sbin/
# rpi-clone sda -f
Initialize and clone to the destination disk sda? (yes/no):yes
Optional destination ext type file system label (16 chars max): ssd
Hit Enter when ready to unmount the /dev/sda partitions ...
最後にEnterを押してUnmountします。
書き込みが完了したらraspi-configを利用してBootをMicroSDからSSDに変更します。
```

# kubernetesのセットアップ

---

セットアップに必要なものをScript化しました。

コンテナランタイムについてはDocker、containerd、CRI-Oがあります。

CRI-OについてはCRI-OのメジャーとマイナーバージョンはKubernetesのメジャーとマイナーバージョンと一致しなければならないため、

利用するには注意してください。

```bash
#!/bin/bash

###containerd install###
apt-get install -y iptables arptables ebtables
apdate-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
update-alternatives --set arptables /usr/sbin/arptables-legacy
update-alternatives --set ebtables /usr/sbin/ebtables-legacy

cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system

apt -y install containerd
mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

if grep -q "SystemdCgroup = true" "/etc/containerd/config.toml"; then
  echo "Config found, skip rewriting..."
else
  sed -i -e "s/SystemdCgroup \= false/SystemdCgroup \= true/g" /etc/containerd/config.toml
fi

systemctl restart containerd
systemctl enable containerd

###k8s install###
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

apt update && apt-get install -y apt-transport-https curl gnupg2
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update
apt install -y kubelet=1.30.2-1.1 kubeadm=1.30.2-1.1 kubectl=1.30.2-1.1
apt-mark hold kubelet kubeadm kubectl
```

# HAProxyの構築

---

- 対象：コントロールプレインノード全てで実施

HAProxyをインストールします。

```bash
# apt install haproxy
```

HAProxyの設定ファイルを作成します。

```bash
# cat /etc/haproxy/haproxy.cfg
global
    log /dev/log  local0 warning
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

   stats socket /var/lib/haproxy/stats

defaults
  log global
  option  httplog
  option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000

frontend kube-apiserver
  bind *:8443
  mode tcp
  option tcplog
  default_backend kube-apiserver

backend kube-apiserver
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server kube-apiserver-1 192.168.21.31:6443 check
    server kube-apiserver-2 192.168.21.32:6443 check
    server kube-apiserver-3 192.168.21.33:6443 check
    server kube-apiserver-4 192.168.21.34:6443 check
    server kube-apiserver-5 192.168.21.35:6443 check
```

 設定ファイルを書いたらHAProxyの自動起動有効化と再起動を実施します。

```bash
# systemctl --now enable haproxy; systemctl status haproxy
Synchronizing state of haproxy.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable haproxy
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-07-18 19:29:27 JST; 3h 30min ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
   Main PID: 984 (haproxy)
     Status: "Ready."
      Tasks: 5 (limit: 3863)
     Memory: 13.1M (peak: 14.9M)
        CPU: 14.988s
     CGroup: /system.slice/haproxy.service
             ├─ 984 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─1045 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
```

# Keepalivedの構築

---

- 対象：コントロールプレインノード全てで実施

Keepalivedをインストールします。

```bash
# apt install keepalived
```

keepalivedの設定ファイルを作成します。

```bash
# vi /etc/keepalived/keepalived.conf
global_defs {
  notification_email {
  }
  router_id LVS_DEVEL
  vrrp_skip_check_adv_addr
  vrrp_garp_interval 0
  vrrp_gna_interval 0
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight 2
}

vrrp_instance haproxy-vip {
  state BACKUP
  priority 100 #Priorityが高いものを優先する
  interface eth0
  virtual_router_id 60
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  unicast_src_ip 192.168.21.31 #設定するノードのIPアドレス
  unicast_peer {
    192.168.21.32, #コントロールプレインノードのIPアドレス
    192.168.21.33,
    192.168.21.34,
    192.168.21.35                         
  }

  virtual_ipaddress {
    192.168.21.30/24 #VIPアドレス
  }

  track_script {
    chk_haproxy
  }
}
```

 設定ファイルを書いたらKeepalivedの自動起動有効化と再起動を実施します。

```bash
# systemctl --now enable keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
root@master03:~# systemctl --now enable keepalived; systemctl status keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
● keepalived.service - Keepalive Daemon (LVS and VRRP)
     Loaded: loaded (/usr/lib/systemd/system/keepalived.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-07-18 19:31:35 JST; 3h 40min ago
       Docs: man:keepalived(8)
             man:keepalived.conf(5)
             man:genhash(1)
             https://keepalived.org
   Main PID: 856 (keepalived)
      Tasks: 2 (limit: 3863)
     Memory: 10.1M (peak: 11.4M)
        CPU: 3min 28.144s
     CGroup: /system.slice/keepalived.service
             ├─856 /usr/sbin/keepalived --dont-fork
             └─910 /usr/sbin/keepalived --dont-fork
```

# kubernetesの構築

---

- 対象：コントロールプレインノードのどこか1台で実施

pod-network-cidrは自分の好きなネットワークにしてください（ホストネットワークとバッティングしないようにしてください）

```bash
# vi config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.30.2
controlPlaneEndpoint: "192.168.21.30:8443"
networking:
  podSubnet: "10.20.0.0/16"

# kubeadm init --config ./config.yaml --upload-certs

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join vip-address:8443 --token ********************** \
        --discovery-token-ca-cert-hash sha256:********************** \
        --control-plane --certificate-key **********************

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join vip-address:8443 --token ********************** \
        --discovery-token-ca-cert-hash sha256:**********************
```

実行後に記載のある通りにコマンドを実行します。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```

対象：コントロールプレインノード

コントロールプレインとして参加させます。

```bash
 kubeadm join kube-vip-address:8443 --token ********************** \
       --discovery-token-ca-cert-hash sha256:********************** \
       --control-plane --certificate-key **********************
```

参加させた後に記載のある通りにコマンドを実行します。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

対象：ワーカーノード

ワーカーノードとして参加させます。

```bash
 kubeadm join kube-vip-address:8443 --token ********************** \
       --discovery-token-ca-cert-hash sha256:********************** \
```

# kubernetesの状況確認

---

対象：コントロールプレインノードのどこか1台で実施

クラスタへの参加は問題なくできておりますが、STATUS欄がNotReady状態となっています。

これは異なるホスト上でのPod間通信の設定（CNIプラグイン）ができていないためとなります。

```bash
# kubectl get node
NAME       STATUS     ROLES           AGE   VERSION
master01   NotReady      control-plane   21h   v1.30.2
master02   NotReady      control-plane   21h   v1.30.2
master03   NotReady   control-plane   18m   v1.30.2
master04   NotReady      control-plane   21h   v1.30.2
master05   NotReady      control-plane   21h   v1.30.2
worker01   NotReady      <none>          21h   v1.30.2
worker02   NotReady      <none>          21h   v1.30.2
worker03   NotReady      <none>          21h   v1.30.2
```

次にPodの状況を確認します。

corednsのSTATUSがPendingになっているのも異なるホスト上でのPod間通信の設定ができていないためとなります。

```bash
# kubectl get pod -n kube-system
NAME                               READY   STATUS    RESTARTS         AGE
coredns-7db6d8ff4d-frdkx           0/1     ContainerCreating   0      21h
coredns-7db6d8ff4d-sqzkj           0/1     ContainerCreating   0      21h
etcd-master01                      1/1     Running   1 (3h55m ago)    21h
etcd-master02                      1/1     Running   1 (3h54m ago)    21h
etcd-master03                      1/1     Running   0                20m
etcd-master04                      1/1     Running   1 (3h52m ago)    21h
etcd-master05                      1/1     Running   1 (3h51m ago)    21h
kube-apiserver-master01            1/1     Running   1 (3h55m ago)    21h
kube-apiserver-master02            1/1     Running   1 (3h54m ago)    21h
kube-apiserver-master03            1/1     Running   0                20m
kube-apiserver-master04            1/1     Running   2 (3h51m ago)    21h
kube-apiserver-master05            1/1     Running   2 (3h50m ago)    21h
kube-controller-manager-master01   1/1     Running   11 (3h55m ago)   21h
kube-controller-manager-master02   1/1     Running   11 (3h54m ago)   21h
kube-controller-manager-master03   1/1     Running   0                20m
kube-controller-manager-master04   1/1     Running   9 (3h52m ago)    21h
kube-controller-manager-master05   1/1     Running   6 (3h51m ago)    21h
kube-proxy-5wkr8                   1/1     Running   1 (3h55m ago)    21h
kube-proxy-6kbhw                   1/1     Running   1 (3h52m ago)    21h
kube-proxy-c2lw5                   1/1     Running   1 (3h51m ago)    21h
kube-proxy-hxngr                   1/1     Running   1 (3h54m ago)    21h
kube-proxy-j7mv5                   1/1     Running   1 (3h51m ago)    21h
kube-proxy-n2ktb                   1/1     Running   0                20m
kube-proxy-rgx6p                   1/1     Running   1 (3h51m ago)    21h
kube-proxy-txck6                   1/1     Running   1 (3h51m ago)    21h
kube-scheduler-master01            1/1     Running   12 (3h55m ago)   21h
kube-scheduler-master02            1/1     Running   11 (3h54m ago)   21h
kube-scheduler-master03            1/1     Running   0                20m
kube-scheduler-master04            1/1     Running   10 (3h52m ago)   21h
kube-scheduler-master05            1/1     Running   7 (3h51m ago)    21h

```

# Calicoの設定

---

対象：master01 

異なるホスト上でのPod間通信の設定ができていないのでCNIプラグインの設定をします。

本投稿ではCalicoを採用していますが、他にもFlannelがあります。

### Flannelとは

シンプルなCNIプラグイン

Linuxカーネルの機能を利用し仮想ネットワーク用のプロトコルVXLANを利用して、

ノード間のOverrayNetworkを構成しコンテナ向けのネットワークを作成

ノード内の「flannel.1」というネットワークインタフェースでノードをまたぐPod間通信をVXLANによってカプセルし、

トンネリングをして他のノードに届けます。受信したノードがカプセル化を解き、Pod同士の通信が実現

### Calicoとは

ノードをまたぐPod間通信をFlannelのようにカプセル化する方法と、カプセル化しない方法のどちらかを選択することが可能

Calicoは各ノードに、Podが利用するCIDRのネットワークアドレスを割り当てる

各ノード内で起動するBIRDが、Pod CIDRを経路情報として交換し、ノード内のルートテーブルに反映

ノードはPodが送信したパケットをルートテーブルに基づいて宛先Podが存在するノードに転送し、

パケットは受信先のノード内のルートテーブルを参照して宛先Podに届く

Calicoの設定ファイルをダウンロードし、以下＋の部分になっているところを修正しデプロイします。# 

```bash
# kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/tigera-operator.yaml
# curl -OL https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml
# cp -p calico.yaml calico.yaml.org
# vi calico.yaml
4996             - name: CALICO_IPV4POOL_CIDR
4997               value: "10.20.0.0/16"

# kubectl apply -f custom-resources.yaml
```

Calicoの設定が完了すればNode間通信ができるようになるためNodeのステータスがReadyとなり、

corednsもRunningになり、calicoもRunningになっていることを確認します。

※CalicoがPodとして起動するには時間がかかったので気長に待ちましょう。

```bash
# kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    control-plane   21h   v1.30.2
master02   Ready    control-plane   21h   v1.30.2
master03   Ready    control-plane   25m   v1.30.2
master04   Ready    control-plane   21h   v1.30.2
master05   Ready    control-plane   21h   v1.30.2
worker01   Ready    <none>          21h   v1.30.2
worker02   Ready    <none>          21h   v1.30.2
worker03   Ready    <none>          21h   v1.30.2

# kubectl get pods -n calico-system
NAME                                       READY   STATUS    RESTARTS        AGE
calico-kube-controllers-5d8479686c-fzq85   1/1     Running   1 (4h1m ago)    21h
calico-node-2vvlk                          1/1     Running   1 (3h58m ago)   21h
calico-node-bwdx2                          1/1     Running   1 (4h1m ago)    21h
calico-node-j87qz                          1/1     Running   3 (4h1m ago)    21h
calico-node-jtmkt                          1/1     Running   1 (3h57m ago)   21h
calico-node-ljxh2                          1/1     Running   1 (116s ago)    26m
calico-node-r6tqg                          1/1     Running   1 (3h58m ago)   21h
calico-node-rmch5                          1/1     Running   1 (3h58m ago)   21h
calico-node-w74f2                          1/1     Running   1 (3h57m ago)   21h
calico-typha-5856fcf8f5-bh7zj              1/1     Running   1 (3h58m ago)   21h
calico-typha-5856fcf8f5-mflq8              1/1     Running   1 (3h58m ago)   21h
calico-typha-5856fcf8f5-wtpjl              1/1     Running   1 (3h57m ago)   21h
csi-node-driver-46gpp                      2/2     Running   2 (3h58m ago)   21h
csi-node-driver-b6tvl                      2/2     Running   2 (4h1m ago)    21h
csi-node-driver-bwmp2                      2/2     Running   2 (3h58m ago)   21h
csi-node-driver-ch52x                      2/2     Running   0               7m37s
csi-node-driver-hm59p                      2/2     Running   2 (3h57m ago)   21h
csi-node-driver-kxr6l                      2/2     Running   2 (4h1m ago)    21h
csi-node-driver-wjcdx                      2/2     Running   2 (3h58m ago)   21h
csi-node-driver-zm8wh                      2/2     Running   2 (3h57m ago)   21h
```

# MetalLBの設定

---

MetalLBとはkubernetesでサービスを外部公開する際にはtype: LoadBlancerを利用しますが、オンプレミスでは利用できません。

それを解決するのがMetalLBとなり、機能をデプロイすることでオンプレミス環境でも利用できるようになります。

MetalLBを動かすとspeakerとcontrollerのPodが動作します。

## controllerとは

Deploymetnによって管理されるPodとなり、MetalLBで利用するIPアドレスの管理を担います。

## speakerとは

Nodeで1台ずつ動作するPodになり、ARP（L2） / NDP（ipv6 L2） / BGP（L3）を使ってサービスの通信到達を担保します。

MetalLBにはL2モードとBGPモードの2つあります。

今回はL2モードを利用するので、L2モードの解説を挟みたいと思います。

L2モードの場合でサービスを公開する場合、各ノードでサービスを提供するようになりますが、

ip aコマンドなど実行しても実際にNICには紐づかず、払い出されたIPに対する通信は1台のノードに集約され、

kube-proxyによって設定されたiptablesルールに従いトラフィックをPodに分散します。

なのでL2モードではIPを持つノードがダウンすると別のノードに切り替わるためLBというよりはFailOverする冗長化機能が正しいかもしれません。

IPを持つノードの管理についてはspeakerからリーダが1台払い出されます。

なおL2モードでは「FailOverする冗長化機能が正しいかも」とお伝えしましたが、

kubernetes自体がNodeDownを検知するため、node-monitor-grace-periodで設定した時間+Podのタイムアウトは5分かかるため、

切り替わりには5分以上かかるので注意が必要です。

参考URL：[https://kubernetes.io/ja/docs/concepts/architecture/nodes/#condition](https://kubernetes.io/ja/docs/concepts/architecture/nodes/#condition)

<aside>
💡

Ready conditionが`pod-eviction-timeout`([kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)に渡された引数)に設定された時間を超えても`Unknown`や`False`のままになっている場合、該当ノード上にあるPodはノードコントローラーによって削除がスケジュールされます。デフォルトの退役のタイムアウトの時間は**5分**です。ノードが到達不能ないくつかの場合においては、APIサーバーが該当ノードのkubeletと疎通できない状態になっています。その場合、APIサーバーがkubeletと再び通信を確立するまでの間、Podの削除を行うことはできません。削除がスケジュールされるまでの間、削除対象のPodは切り離されたノードの上で稼働を続けることになります。

</aside>

またもう一つの注意としてはリーダーとして選出されたノードにて集約されるので、

NICの帯域上限が通信の限界（自宅で動かすにはよっぽどのことがない限り大丈夫だと思いますが）となります。

デプロイは以下のように実施します。

```bash
# curl -O https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
# kubectl apply -f metallb-native.yaml
# kubectl get all -n metallb-system
NAME                              READY   STATUS    RESTARTS      AGE
pod/controller-595f88d88f-jncfq   1/1     Running   1 (12s ago)   63s
pod/speaker-4r7jb                 1/1     Running   0             63s
pod/speaker-chj5z                 1/1     Running   0             63s
pod/speaker-clszj                 1/1     Running   0             63s
pod/speaker-cnwgh                 1/1     Running   0             63s
pod/speaker-gxvft                 1/1     Running   0             63s
pod/speaker-hct2b                 1/1     Running   0             63s
pod/speaker-jqsfl                 1/1     Running   0             63s
pod/speaker-mtkxr                 1/1     Running   0             63s

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.101.93.11   <none>        443/TCP   63s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   8         8         8       8            8           kubernetes.io/os=linux   63s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           63s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-595f88d88f   1         1         1       63s
```

Helmを使う場合には以下

```bash
# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
# chmod 700 get_helm.sh
# ./get_helm.sh

# kubectl create namespace metallb
# helm repo add metallb https://metallb.github.io/metallb
# helm install metallb metallb/metallb --namespace metallb
```

MetalLBが自動的にLBを払い出してもらうためにレンジを設定します。

```bash
# cat <<EOF > metallb-config.yaml 
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - IPアドレス範囲開始-IPアドレス範囲終了
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
EOF

# kubectl apply -f metallb-config.yaml 
# kubectl get configmap -n metallb-system
NAME               DATA   AGE
kube-root-ca.crt   1      3h3m
```

# CSIの設定

---

CSIとはContainer Storage Interfaceといい、コンテナオーケストレーション向けに作られたストレージインターフェースです。

本機能を利用することでPod内で動いているコンテナが複数同じVolumeを見にいくことができるようになったり、

PVCやPVのサイズを拡張できたりすることができます。

これを実現する前には自宅でNFSサーバを使ってnfs-subdir-external-provisionerを利用していましたが、

PVCやPVのサイズを拡張ができないため歯がゆい思いをしてきましたが我が家にSynologyのNASが来たので、

ようやくCSIを使うことができます👏

SynolodyのNASであるCSI Driverを利用するための手順は以下です。

パラメータはGithubに乗っているので参考にしてみてください。

[https://github.com/SynologyOpenSource/synology-csi](https://github.com/SynologyOpenSource/synology-csi)

```bash
Workerノードに以下をインストールしてから実行してください。
# apt install -y open-iscsi

Synology CSI Install
# git clone https://github.com/SynologyOpenSource/synology-csi.git
# cd synology-csi
# cp config/client-info-template.yml config/client-info.yml
# cat config/client-info.yml
---
clients:
  - host: NASのIPアドレス
    port: 5000
    https: false
    username: アカウント名 
    password: パスワード

# cat deploy/kubernetes/v1.20/storage-class.yml 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: synology-iscsi-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: csi.san.synology.com
# if all params are empty, synology CSI will choose an available location to create volume
parameters:
  dsm: 'IPアドレス'
  location: '/volumeのパス'
  fsType: 'ext4'
reclaimPolicy: Delete 
allowVolumeExpansion: true

# ./scripts/deploy.sh install --all
→k8sのバージョンがv1.28からversion確認方法が変わっているので、スクリプトの中身を変更してください。
 ver=$(kubectl version | grep Server | awk '{print $3}')
```

これで基本的な機能をkubernetes上に載せることができましたので、

この先は私自身がどれだけkubernetesを使いこなせるか⁉️

になるのでリソース購入台のコスト回収ができるくらい勉強に励みたいと思います。