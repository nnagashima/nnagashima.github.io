# zabbix proxmox monitoring

![Zabbix_logo-RGB.png](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Zabbix_logo-RGB.png)

目次

# はじめに

---

Zabbix6.0からProxmoxモニタリングのテンプレートが公開されたので、どんなことを診ることができるのか確認します。

本監視テンプレートは外部スクリプトなし、HTTP エージェントによって、メトリックを収集でProxmoxノードを監視することができます。

[Proxmox monitoring and integration with Zabbix](https://www.zabbix.com/integrations/proxmox)

# ProxmoxVEでユーザー作成

---

1. ProxmoxのUIにログインしてユーザーを作成します。
2. API TOKENで作成したユーザーでTOKENを作成します。（あとで利用するのでコピーして保存してください）
3. アクセス権限にてユーザーのアクセス制限とAPITOKENのアクセス制限で、PVEAuditorロールの割り当てを実施する

![Untitled](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Untitled.png)

※APITOKENだけだとアクセス権限にうまく反映されず、ユーザーの権限も追加したらうまくいきました。

# ZabbixにてProxmox VE by HTTPのテンプレート適用

---

Proxmoxのホスト登録してテンプレート適用します。

筆者はすでにZabbixAgentを使ってProxmoxサーバを監視しているので、

同一ホストに「Proxmox VE by HTTP」割り当てを行いホストマクロを設定します。

| マクロ | 値 |
| --- | --- |
| {$PVE.TOKEN.ID} | zabbix-user@pve!zabbix-user
→作成したAPITOKENのユーザーを記載します。 |
| {$KUBE.API.TOKEN} | 作成したAPITOKENのユーザーのTOKENを記載します。 |

ディスカバリーによって、以下が検出されております。

- PrxoxmoxでAPIアクセスする際の応答コード

![Untitled](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Untitled%201.png)

- Proxmox VEのリソース情報

![Untitled](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Untitled%202.png)

- Proxmox VEに登録しているStorageの情報

![Untitled](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Untitled%203.png)

- Proxmox上で起動しているVMのリソース情報やステータス情報

![Untitled](zabbix%20proxmox%20monitoring%209bb60f4069d349b59bd6ab2221348a9b/Untitled%204.png)

家でProxmoxVEを利用していてハイパーバイザーを監視するのに今までZabbixAgentをホストに入れて監視したり、

各VMにZabbixAgentに入れたりして監視していたのがAgentレスで監視することができるようになった、

また両方の側面から監視ができるようになったので便利になりました。