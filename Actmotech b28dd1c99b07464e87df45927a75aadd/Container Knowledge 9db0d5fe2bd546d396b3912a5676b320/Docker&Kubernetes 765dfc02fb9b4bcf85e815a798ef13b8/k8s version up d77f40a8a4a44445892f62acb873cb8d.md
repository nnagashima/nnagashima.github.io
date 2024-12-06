# k8s version up

![kubernetes-icon-2048x1995-r1q3f8n7.png](k8s%E7%92%B0%E5%A2%83%E3%81%AB%E3%83%9E%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%8E%E3%83%BC%E3%83%88%E3%82%99%E3%82%92%E8%BF%BD%E5%8A%A0%20238a40d0e8bf450b81ceae7ec132b1f2/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# 現在のバージョン確認

---

kubernetesを1.26から1.27にバージョンアップする手順になります。

以下コマンドで各ノードのkubernetesバージョンを確認します。

※ray-k8s-master01がv1.27.0になっているのは、事前にやってしまったためです💦

```bash
# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.0", GitCommit:"b46a3f887ca979b1a5d14fd39cb1af43e7e5d12d", GitTreeState:"clean", BuildDate:"2022-12-08T19:57:06Z", GoVersion:"go1.19.4", Compiler:"gc", Platform:"linux/arm64"}

# kubectl get node
NAME               STATUS   ROLES           AGE   VERSION
ray-k8s-master01   Ready    control-plane   91d   v1.26.0
ray-k8s-master02   Ready    control-plane   91d   v1.26.0
ray-k8s-master03   Ready    control-plane   91d   v1.26.0
ray-k8s-master04   Ready    control-plane   72d   v1.26.0
ray-k8s-master05   Ready    control-plane   72d   v1.26.0
ray-k8s-worker01   Ready    <none>          91d   v1.26.0
ray-k8s-worker02   Ready    <none>          91d   v1.26.0
ray-k8s-worker03   Ready    <none>          91d   v1.26.0
```

# バージョンアップするためのパッチリリースを確認

---

Ubuntuでは以下になりますが、RHEL系でも同様にdnfを利用します。

```bash
# apt update
Get:1 http://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Hit:2 http://deb.debian.org/debian bullseye InRelease                                                                                                                          
Hit:3 http://deb.debian.org/debian bullseye-updates InRelease                                                                                         
Hit:4 https://download.docker.com/linux/debian bullseye InRelease                                                                                     
Hit:5 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                                                                        
Hit:6 http://archive.raspberrypi.org/debian bullseye InRelease
Fetched 48.4 kB in 2s (22.2 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
25 packages can be upgraded. Run 'apt list --upgradable' to see them.

# apt list --upgradable
Listing... Done
containerd.io/bullseye 1.6.20-1 arm64 [upgradable from: 1.6.15-1]
cri-tools/kubernetes-xenial 1.26.0-00 arm64 [upgradable from: 1.25.0-00]
kubeadm/kubernetes-xenial 1.27.0-00 arm64 [upgradable from: 1.26.0-00]
kubectl/kubernetes-xenial 1.27.0-00 arm64 [upgradable from: 1.26.0-00]
kubelet/kubernetes-xenial 1.27.0-00 arm64 [upgradable from: 1.26.0-00]
kubernetes-cni/kubernetes-xenial 1.2.0-00 arm64 [upgradable from: 1.1.1-00]
libcamera-apps-lite/stable 0~git20230301+54a781d-1 arm64 [upgradable from: 0~git20230104+4fea2ee-1]
libcamera0/stable 0~git20230302+923f5d70-1 arm64 [upgradable from: 0~git20230105+5df5b72c-1]
libraspberrypi-bin/stable 1:2+git20230322~143557+9d5250f-1 arm64 [upgradable from: 1:2+git20220324~090146+c4fd1b8-1]
libraspberrypi-dev/stable 1:2+git20230322~143557+9d5250f-1 arm64 [upgradable from: 1:2+git20220324~090146+c4fd1b8-1]
libraspberrypi-doc/stable,stable 1:2+git20230322~143557+9d5250f-1 all [upgradable from: 1:2+git20220324~090146+c4fd1b8-1]
libraspberrypi0/stable 1:2+git20230322~143557+9d5250f-1 arm64 [upgradable from: 1:2+git20220324~090146+c4fd1b8-1]
libssl1.1/stable 1.1.1n-0+deb11u4+rpt1 arm64 [upgradable from: 1.1.1n-0+deb11u4]
linux-libc-dev/stable 1:1.20230405-1 arm64 [upgradable from: 1:1.20230106-1]
openssl/stable 1.1.1n-0+deb11u4+rpt1 arm64 [upgradable from: 1.1.1n-0+deb11u4]
python3-libcamera/stable 0~git20230302+923f5d70-1 arm64 [upgradable from: 0~git20230105+5df5b72c-1]
python3-picamera2/stable,stable 0.3.9-1 all [upgradable from: 0.3.8-1]
python3-v4l2/stable,stable 0.3.2-1 all [upgradable from: 0.3.1-1]
raspberrypi-bootloader/stable 1:1.20230405-1 arm64 [upgradable from: 1:1.20230106-1]
raspberrypi-kernel/stable 1:1.20230405-1 arm64 [upgradable from: 1:1.20230106-1]
raspberrypi-sys-mods/stable 20230329 arm64 [upgradable from: 20221019]
raspi-config/stable,stable 20230214 all [upgradable from: 20221214]
raspinfo/stable,stable 20230123-1 all [upgradable from: 20221220-1]
rpi-eeprom/stable 16.0-1 arm64 [upgradable from: 15.1-1]
tzdata/stable-updates,stable-updates 2021a-1+deb11u9 all [upgradable from: 2021a-1+deb11u8]
```

# コントロールプレーンノードのアップデート

---

kubeadmをバージョンアップします。

```bash
# apt upgrade kubeadm 
```

MasterノードからPodをDrainしてスケジューリング対象から外します。

```bash
# kubectl drain [node名] --ignore-daemonsets
# kubectl get node
NAME               STATUS                     ROLES           AGE   VERSION
ray-k8s-master01   Ready                      control-plane   91d   v1.26.0
ray-k8s-master02   Ready,SchedulingDisabled   control-plane   91d   v1.26.0
ray-k8s-master03   Ready                      control-plane   91d   v1.26.0
ray-k8s-master04   Ready                      control-plane   72d   v1.26.0
ray-k8s-master05   Ready                      control-plane   72d   v1.26.0
ray-k8s-worker01   Ready                      <none>          91d   v1.26.0
ray-k8s-worker02   Ready                      <none>          91d   v1.26.0
ray-k8s-worker03   Ready                      <none>          91d   v1.26.0
```

kubeadmのバージョンを確認します。　

```bash
# kubeadm version
```

kubernetesクラスターがアップグレードできることを確認します。

```bash
# kubeadm upgrade plan
upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.26.0
[upgrade/versions] kubeadm version: v1.27.0
[upgrade/versions] Target version: v1.27.0
[upgrade/versions] Latest version in the v1.26 series: v1.26.3

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     7 x v1.26.0   v1.26.3
            1 x v1.27.0   v1.26.3

Upgrade to the latest version in the v1.26 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.26.0   v1.26.3
kube-controller-manager   v1.26.0   v1.26.3
kube-scheduler            v1.26.0   v1.26.3
kube-proxy                v1.26.0   v1.26.3
CoreDNS                   v1.9.3    v1.10.1
etcd                      3.5.6-0   3.5.7-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.26.3

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     7 x v1.26.0   v1.27.0
            1 x v1.27.0   v1.27.0

Upgrade to the latest stable version:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.26.0   v1.27.0
kube-controller-manager   v1.26.0   v1.27.0
kube-scheduler            v1.26.0   v1.27.0
kube-proxy                v1.26.0   v1.27.0
CoreDNS                   v1.9.3    v1.10.1
etcd                      3.5.6-0   3.5.7-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.27.0

_____________________________________________________________________
```

kubernetesアップグレートを実施します。

```bash
 # kubeadm upgrade apply v1.27.0
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.27.0". Enjoy!
```

kubectl uncordonを実行してMasterノードをスケジューリング対象に戻します。

```bash
# kubectl uncordon [node名]
# kubectl get node
NAME               STATUS   ROLES           AGE   VERSION
ray-k8s-master01   Ready    control-plane   91d   v1.27.0
ray-k8s-master02   Ready    control-plane   91d   v1.26.0
ray-k8s-master03   Ready    control-plane   91d   v1.26.0
ray-k8s-master04   Ready    control-plane   72d   v1.26.0
ray-k8s-master05   Ready    control-plane   72d   v1.26.0
ray-k8s-worker01   Ready    <none>          91d   v1.26.0
ray-k8s-worker02   Ready    <none>          91d   v1.26.0
ray-k8s-worker03   Ready    <none>          91d   v1.26.0

```

kubeletとkubectlのアップグレートを実施します。

```bash
# apt upgrade kubelet kubectl 
```

kubeletの再起動を実施します。

```bash
# systemctl restart kubelet
```

以降Masterノードが複数ある場合には上記を繰り返しします。

# Workerノードのアップデート

---

Workerノードのkubeadmをアップデートします。

Masterノードと被っているところは説明省略します。

```bash
# apt update
# apt list --upgradable
# apt upgrade kubeadm 
```

MasterノードにてWorkerノードからPodをDrainしてスケジューリング対象から外します。

```bash
# kubectl drain [node名] --ignore-daemonsets

ここで気をつけるのがWorkerでローカルストレージを持っているPodがある場合にはDrainすることができません。
なので上記に引っかかる場合には「--force --delete-emptydir-data」の引数を付与することでPodを排除できます。
Daemonsetは無視しているのでこれに紐づくPodは動き続けていますが、Deploymentのように数に縛りがあるものは別のNodeにてPodが展開されます。
しかし別Nodeに空きがない場合には以下のようにPendingとなってしまうため、Drainする前に事前確認をしてください。
※あえて出力されるようにやっています。

# kubectl get event --all-namespaces
NAMESPACE   LAST SEEN   TYPE      REASON                   OBJECT                                  MESSAGE
postgres    7m43s       Warning   FailedScheduling         pod/acid-zabbix-1                       0/8 nodes are available: 1 node(s) were unschedulable, 2 node(s) didn't match

```

Masterノードにてkubelet構成のアップグレードを実施します。

```bash
# kubeadm upgrade node
```

Workerノードにてkubeletとkubectlをアップグレードします。

```bash
# apt upgrade kubelet kubectl 
```

Workerノードにてkubeletの再起動を実施します。

```bash
# systemctl restart kubelet
```

Masterノードにてkubectl uncordonを実行してWorkerノードをスケジューリング対象に戻します。

```bash
# kubectl uncordon [node名]
# kubectl get node
```

以降Workerノードが複数ある場合には上記を繰り返しします。