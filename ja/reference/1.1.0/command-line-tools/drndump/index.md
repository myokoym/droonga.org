---
title: drndump
layout: ja
---

{% comment %}
##############################################
  THIS FILE IS AUTOMATICALLY GENERATED FROM
  "_po/ja/reference/1.1.0/command-line-tools/drndump/index.po"
  DO NOT EDIT THIS FILE MANUALLY!
##############################################
{% endcomment %}


* TOC
{:toc}

## 概要 {#abstract}

`drndump`は、指定されたDroongaクラスタ内の特定のデータセットについて、全てのスキーマ定義、レコード、およびインデックスの定義を抽出します。

例えば、`192.168.100.50`というDroonga Engineノードがあり、同一ネットワークセグメント内のコンピュータ`192.168.100.10`にログインしている場合、クラスタ内の全てのデータを抽出するコマンド列は以下のようになります：

~~~
(on 192.168.100.10)
$ drndump --host 192.168.100.50 --receiver-host 192.168.100.10
{
  "type": "table_create",
  "dataset": "Default",
  "body": {
    "name": "Location",
    "flags": "TABLE_PAT_KEY",
    "key_type": "WGS84GeoPoint"
  }
}
{
  "type": "table_create",
  "dataset": "Default",
  "body": {
    "name": "Store",
    "flags": "TABLE_PAT_KEY",
    "key_type": "ShortText"
  }
}
...
{
  "type": "column_create",
  "dataset": "Default",
  "body": {
    "table": "Term",
    "name": "store_name",
    "type": "Store",
    "flags": "COLUMN_INDEX|WITH_POSITION",
    "source": "name"
  }
}
~~~

このコマンドの出力は、別のDroongaクラスタに同じデータを復元することのできる妥当なメッセージの一覧です。
言い換えると、このコマンドはあるDroongaクラスタの完全なバックアップを作成することができます。

出力はリダイレクトを使って以下のようにファイルに保存できます：

~~~
(on 192.168.100.10)
$ drndump --host 192.168.100.50 --receiver-host 192.168.100.10 \
    > dump.jsons
~~~

[`droonga-request`コマンド](../droonga-request/)または[`droonga-send`コマンド](../droonga-send/)を使うと、ダンプ出力からデータセットを復元できます。
各コマンドの説明も参照して下さい。


## パラメータ {#parameters}

`--host=NAME`
: メッセージの送信先となるEngineノードのホスト名。
  既定値は、コマンドを実行しているコンピュータ自身の推測されたホスト名です。

`--port=PORT`
: Engineノードとの通信に使うポート番号。
  既定値は`10031`です。

`--tag=TAG`
: Engineノードとの通信に使うタグ名。
  既定値は`droonga`です。

`--dataset=NAME`
: ダンプ出力するデータセットの名前。
  既定値は`Default`です。

`--receiver-host=NAME`
: このコマンドを実行しているコンピュータのホスト名。
  既定値は、そのコンピュータのホスト名として推測される名前です。

`--help`
: コマンドの使い方の説明を表示します。


## インストール方法 {#install}

このコマンドは、Rubygemsのパッケージ`drndump`の一部としてインストールされます。

~~~
# gem install drndump
~~~

