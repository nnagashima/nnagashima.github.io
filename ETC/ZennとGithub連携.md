[TOPへ戻る](https://actmotech.xyz/)
[ETC Knowledgeへ戻る](/ETC/top.md)

# ZennとGithub連携

目次
- [ZennとGithub連携](#zennとgithub連携)
- [はじめに](#はじめに)
- [環境](#環境)
- [やったこと](#やったこと)

<br>

# はじめに

Qiitaやはてブロなど色々渡り歩いている中、Zennでも投稿していた時期があり

Zennでの記事管理をどうしていたかを残しておきたいと思います。

# 環境

MAC Mini 2018

# やったこと

以下を参考に実施しました。

[GitHubリポジトリでZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)

[Zenn CLIの導入手順](https://zenn.dev/zenn/articles/install-zenn-cli)

セットアップに実行した内容は以下となります。 

※npx zenn initは、実際にGithubと連携するディレクトリ上で実施するようにしてください。

```bash
% brew install npm
% npm init --yes
% npm install zenn-cli
% npx zenn init
Generating articles skipped.Generating books skipped.Generating .gitignore skipped.Generating README.md skipped.  
🎉  Done!  早速コンテンツを作成しましょう  
👇  新しい記事を作成する  $ zenn new:article  
👇  新しい本を作成する  $ zenn new:book  
👇  投稿をプレビューする  $ zenn preview
```

zenn previewを実行するとhttp://localhost:8000でガイド情報を表示することができたり、 

Uploadする前に記載内容の確認ができたりするので立ち上げてガイドをまずはよく読むと良いかと思います。

記事作成コマンドを実行する際には以下になります。

```bash
npx zenn new:article
```

上記コマンドを実行するとarticlesディレクトリ内にmdファイルが作成されるので、 

作成されたmdファイルをベースに作成し、プレビューで確認しGithubへPushすることでZennで公開ができます。

[TOPへ戻る](https://actmotech.xyz/)
[ETC Knowledgeへ戻る](/ETC/top.md)