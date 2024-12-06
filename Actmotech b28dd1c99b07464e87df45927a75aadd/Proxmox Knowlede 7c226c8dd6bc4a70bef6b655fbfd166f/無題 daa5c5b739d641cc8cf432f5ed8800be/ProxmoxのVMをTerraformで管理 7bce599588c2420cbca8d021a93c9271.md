# ProxmoxのVMをTerraformで管理

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

目次

# はじめに

---

VMを手動で作ることに疲れてしまったためProxmoxをTerraformで管理できないか、

調べたところProvidorが提供されていることを知り、管理することにしました。

# 利用環境

---

RockyLinux9を利用しています。

# Terraformインストール

---

RockyLinuxにTerraformをインストールします。

```
# dnf install -y yum-utilis
# yum-config-manager --add-repo [https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo](https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo)
# dnf install -y terraform
# terraform  version
```

# Proxmox環境でユーザー周りの作成

---

管理したいProxmoxにSSHでログインして以下コマンドを実行します。

```
## Terraform用のロール作成
# pveum role add TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"

## Terraform用のProxmoxアカウントを作成
# pveum user add terraform-prov@pve --password password

## Terraform用のProxmoxアカウントに先ほど作成したロールを割り当て
# pveum aclmod / -user terraform-prov@pve -role TerraformProv

## Terraform用のProxmoxアカウントでAPIユーザーを作成
# pveum user token add terraform-prov@pve terraform-token

## Terraform用のProxmoxアカウントでAPIユーザーを作成後、先ほど作成したロールを割り当て
# pveum aclmod / --tokens 'terraform-prov@pve!terraform-token' --roles TerraformProv
※表示された結果はメモするようにしておいてください。
```

# VMを作る際のテンプレート作成

---

VMを作る際の管理テンプレートをこれから作りますが、

VMデプロイ時にネットワーク設定なども同時にしたいので、

cloud-initテンプレートを準備します。

今回はUbuntu22.04のテンプレートを用意していますがwgetで取得するURLの階層をあがることで、

他のOSやバージョンを手に入れることができます。

管理したいProxmoxにSSHでログインして以下コマンドを実行します。

```
## cloud-initテンプレートをダウンロードします
# wget [https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img](https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img)

## VMのメモリとネットワークを設定して作成します
# qm create 10000 --memory メモリサイズ（バイト） --net0 virtio,bridge=vmbr0 --agent 1

## 先ほど作成したVMにダウンロードしたimgファイルをインポートし、保存するストレージを最後に記載します。 
# qm importdisk 10000 jammy-server-cloudimg-amd64.img 保存する場所

## インポートしたディスクをscsi0としてVMにアタッチします。
# qm set 10000 --scsihw virtio-scsi-pci --scsi0 保存した場所:10000/vm-10000-disk-0.raw

## 作成したVMに名前をつけます。
# qm set 10000 --name "Ubuntu22.04"

## cloud-initが利用するCDROMドライブを設定します
# qm set 10000 --ide2 synology-haproxmox:cloudinit

## アタッチしたディスクをブートディスクとして設定します
# qm set 10000 --boot c --bootdisk scsi0

## cloud-initはシリアルコンソールを使用するので設定をします
# qm set 10000 --serial0 socket --vga serial0

## Bashからログインする際に必要なユーザー名とパスワードを設定します
# qm set 10000 --ciuser ユーザー名
# qm set 10000 --cipassword パスワード

## SSHでリモートでアクセスする際に事前に鍵を作り、キーをインポートします
# qm set 10000 --sshkey .ssh/id_rsa.pub
```

# Terraformの作業スペース作成

---

Terraformで作業フォルダを作成し、必要な設定ファイルを作成してワークスペースを初期化するコマンドを実行します。

```
# mkdir work
# cd work

# vi main.tf
provider "proxmox" {
	pm_api_url = var.api_url
	pm_api_token_id = var.api_token_id
	pm_api_token_secret = var.api_token_secret
	pm_tls_insecure = true
}

# vi var.tf
variable "api_url" {
	default = "https://192.168.21.21:8006/api2/json"
}

variable "api_token_id"{
	default = "terraform-prov@pve!terraform"
}

variable "api_token_secret"{
	default = "シークレット情報を記載"
}

# vi version.tf
terraform {
	required_providers {
		proxmox = {
			source = "Telmate/proxmox"
			version = "2.9.11"
		}
	}
}

# terraform init
```

# VM作成するtfファイルを作成

---

VMを作成するファイルを作成します。

項目ごとにコメントを記載しているので、参考にしてみてください。

```
# vi k8s-vm.tf
resource "proxmox_vm_qemu" "vm01" {
	vmid = 100
	name = "VMの名前"
	target_node = "Proxmoxでデプロイする場所"
	clone = "先ほど作成したVMテンプレートのVM名を指定"
	memory = メモリサイズ
	sockets = CPUのソケット数
	cores = CPUのコア数
	os_type = "cloud-init"
	ipconfig0 = "ip=IPアドレス/サブネット,gw=GWアドレス"
	nameserver = "DNS参照先"
	disk{
		type = "scsi"
		storage = "Diskを作成するStorageの場所"
		size = "ディスクサイズ"
	}
}

```

# VMをデプロイ

---

ここまでできたら構文チェックと実行したらどうなるかの確認を行ってから、

本番実行を実施します。

```
## 変数情報に誤りや、構文ミスがないかチェック
# terraform validate
※問題があればエラーが出力されます。

## 本番実行前の確認
# terraform plan
※問題があればエラーが出力されます。

## デプロイの実行
#terraform apply
```

実行したらProxmox上でVMがデプロイができていることを確認します。