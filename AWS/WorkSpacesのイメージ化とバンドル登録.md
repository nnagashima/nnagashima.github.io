[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# WorkSpacesのイメージ化とバンドル登録

# 目次

- [WorkSpacesのイメージ化とバンドル登録](#workspacesのイメージ化とバンドル登録)
- [目次](#目次)
- [はじめに](#はじめに)
- [WorkSpacesのイメージ化するにあたって気をつけること](#workspacesのイメージ化するにあたって気をつけること)
- [WorkSpacesのイメージをバンドル登録](#workspacesのイメージをバンドル登録)
- [WorkSpacesのバンドルから展開](#workspacesのバンドルから展開)
- [WorkSpacesのマスターイメージの更新](#workspacesのマスターイメージの更新)

---

# はじめに

---

WorkSpacesのイメージ化の際に気をつけること、バンドル登録時に気をつけることを記載します。

# WorkSpacesのイメージ化するにあたって気をつけること

---

起動しているWorkSpacesからイメージ作成を実施します。

- イメージ化するWorkSpacesのディスクの暗号化しているとイメージ化できません。
- イメージ化するWorkSpacesのアプリケーションやWorkSpaces展開時に必要なファイルはCドライブにインストールしてください。
- イメージ化するWorkSpacesのWindowsUpdateは最新化してから実施してください。
- イメージ化する際にアプリケーションにログイン情報などはいれないようにしてください。
- アプリケーションによってはイメージ化したものからWorkSpacesを展開すると想定動作しないものがあるので動作検証しましょう。
- アプリケーションのショートカットはパブリックデスクトップに配置してください。

# WorkSpacesのイメージをバンドル登録

---

作成したイメージからバンドル作成を実施します。

バンドル登録時のハードウェアタイプはWorkSpaces展開する際に選択するので、

複数のハードウェアタイプを登録したい場合には、複数バンドル登録を実施してください。
    

# WorkSpacesのバンドルから展開

---

WorkSpaces作成の画面からカスタムバンドルをクリックしてWorkSpacesを作成します。

# WorkSpacesのマスターイメージの更新

---

WorkSpacesのマスターイメージを更新したらイメージ化します。

その後、登録しているバンドルのソースイメージを新しく取得したイメージに付け替えることで、

展開済みのWorkSpacesを再構築した際、もしくは新規作成した際に新しいイメージで作成することができます。

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
