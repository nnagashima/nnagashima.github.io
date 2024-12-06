# Proxmoxインストール後の初期設定

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

目次

# Proxmoxとは

---

ProxmoxとはKVMで無料でKVMのHA組んだり、マイグレーションしたり、アンチアフィニティ設定ができます。

もちろんStorage追加すればVMのDISKデータを外だしできます。

Backup機能も搭載していて、Storageに対してVM丸ごとバックアップ取得することが可能です。

# リポジトリ設定

---

無料のProxmoxではPKGアップデートするための設定変更が必要ですのでお忘れ無く。

```bash
# cat /etc/apt/sources.list
deb http://ftp.debian.org/debian bullseye main contrib
deb http://ftp.debian.org/debian bullseye-updates main contrib
# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
# security updates
deb http://security.debian.org/debian-security bullseye-security main contrib
```