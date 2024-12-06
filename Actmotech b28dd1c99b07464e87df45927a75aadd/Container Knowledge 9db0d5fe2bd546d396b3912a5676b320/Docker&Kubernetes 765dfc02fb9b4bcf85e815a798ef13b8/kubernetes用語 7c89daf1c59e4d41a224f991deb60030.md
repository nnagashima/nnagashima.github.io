# kubernetes用語

![kubernetes-icon-2048x1995-r1q3f8n7.png](kubernetes%E7%94%A8%E8%AA%9E%207c89daf1c59e4d41a224f991deb60030/kubernetes-icon-2048x1995-r1q3f8n7.png)

目次

# そもそもkubernetesとは

---

DockerやDockerComopseで起動するコンテナはホストに依存し、ホストが倒れたらコンテナも当然のように倒れます。

そのためにはホスト間を跨いでコンテナが起動するようにする必要があります。

これを解決してくれるのがコンテナオーケストレーションになり、kubernetesはその一つです。

※他にもDockerSwarmがありましたが、kubernetesに負けてしまったのかな？

# kubernetesで出てくる用語

---

何事もそうですが用語がわからないことには次に進めませんよね。

そこで出てくる用語を簡単にまとめてみました。

## Node

kubernetesを実行する仮想マシンもしくは物理マシンのことです。

kubernetesではNodeが複数集まってクラスターを構成します。

NodeにはMasterとWorkerが存在し、MasterがWorkerノードをコントロールします。

利用ユーザーはMaster側からコマンドを利用して指示を出してコントロールをしていきます。

## Pod

コンテナをグループ化する単位になります。

そのためコンテナが単一のPodもあれば、複数コンテナが存在するPodもあります。

## Replicaset

Podのレプリカ作成やPodの起動数の管理を行う機能になります。

## Deployment

Replicasetを管理しデプロイ状態を管理するリソースとなり、Podに異常があれば再起動などを試み正常なPod数で起動するように管理します。

## Service

実行中のPodにアクセスしやすいようにエンドポイントを提供します。

# なぜkubernetesを調べたのか

---

単純に言うと難しそうでワクワクしたからです。

あとはナレッジや公式ドキュメントが英語ということもあり、久しく英語環境に触れていない私にとっては復習にもなると思ったからです。

またCKAを昨年取得しているのですが、CKAを取得するにも机上のみでは私は勉強が捗らない傾向にあり、

実践を通してやったほうが覚えが早いということもありました。

そのためにベアボーンの機器にメモリを詰めるだけ積んでやSynologyのNASを購入して環境を整えました。

やりたいことはDBのHA構成を簡単に用意したいこと、VMをKubernetes環境の上で実行することです。

通常DBを冗長化するにはPacemaker+pgpool-II+Postgresであったり、PatoroniであったりあるのですがHA構成組むだけでまぁ大変です。

仕組みを理解し勉強するのはとても楽しいですが、とにかくHA構成を作るのに時間がかかる上に運用で気にしないといけないことが多々あります。

DBに至ってはZalando Postgres Operatorというものがあるのですが、

コンテナの管理ツールであるKubernetes上でHA構成のPostgreSQLクラスタを容易に構築できるようにするoperatorです。

※operatorはKubernetesの機能を拡張するプラグインみたいなものです。

PostgresOperatorには以下機能があります。

- PostgreSQL HAクラスタの管理
- ストリーミングレプリケーション (同期・非同期)
- コネクションプール (PgBouncer使用)
- フェイルオーバー (pg_rewindにより高速な復旧が可能)
- ローリングアップデート
- 論理バックアップ (S3使用)
- WALアーカイブ・PITR (WAL-E、S3使用)

![Untitled](kubernetes%E7%94%A8%E8%AA%9E%207c89daf1c59e4d41a224f991deb60030/Untitled.png)

一方でZalando Postgres OperatorはPrometheusで監視するために自前で監視するための設定が必要となります。

[このサイトが参考になるかと思いますので、リンクをつけておきます。](https://shivering-isles.com/postgres-operator-with-monitoring)

もう一つの理由で挙げていたVMをKubernetes環境の上で実行です。

全てコンテナとして実行できれば、それはそれで理想だと思いますが、そうはいかないものも世の中にはたくさんあるかと思います。

なのでkubernetes上でVMを起動して管理ができたらいいなと思って調べていたらKubeVirtとなるものを見つけました。

まだ触れたこともないのでちゃんと理解はできておりませんがKubernetes上にインストールするアドオンでPodと同じノードでVMを起動するようです。

ちなみに執筆時点(2023/01/27)時点でv0.58.0なのでv1に到達しておりません。

これらを2つを利用することでkubernetesの勉強をしつつ、なんちゃんてマネージドサービスの真似事ができるのではないかと考え、

やる気を出した感じです。

また他にも今までVMで動かしていたものを少しずつkubernetes環境に移行して、

最終的には全て載せることが最高だなと思っていますが、そんな未来はまだまだ先だろうと思っています。

※キャッシュDNSとかMinecraftとかZabbixとかGuacamoleを移行対象に考えています。