# ProxmoxでVM作成後のAgent導入

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

目次

# Qemu Guest Agent導入

---

ProxmoxでVMを作成した後に、Proxmoxの管理UI上でVMのIPアドレスの情報を表示したい場合は、

Proxmox VM Qemu Gust Agentを導入する必要があります。

以下はRHEL系でのインストール手順となりますが、Ubuntuでも同じPKG名でインストールすることができます。

```bash
# dnf install -y qemu-guest-agent
# systemctl start qemu-guest-agent
# systemctl enable qemu-guest-agent
```

インストール完了後に、Proxmoxの管理UI上でVMを選択してIPアドレスなどの情報が表示されていることを確認します。