---
title: search
layout: ja
---

{% comment %}
##############################################
  THIS FILE IS AUTOMATICALLY GENERATED FROM
  "_po/ja/reference/1.0.4/commands/search/index.po"
  DO NOT EDIT THIS FILE MANUALLY!
##############################################
{% endcomment %}


* TOC
{:toc}

## 概要 {#abstract}

`search` は、1つ以上のテーブルから指定された条件にマッチするレコードを検索し、見つかったレコードに関する情報を返却します。

これは、Droonga において検索機能を提供する最も低レベルのコマンドです。
検索用のコマンドをプラグインとして実装する際は、内部的にこのコマンドを使用して検索を行うという用途が想定されます。

## APIの形式 {#api-types}

### HTTP {#api-types-http}

リクエスト先
: `(ドキュメントルート)/droonga/search`

リクエストメソッド
: `POST`

リクエストのURLパラメータ
: なし。

リクエストのbody
: [パラメータ](#parameters)のハッシュ。

レスポンスのbody
: [レスポンスメッセージ](#response)。

### REST {#api-types-rest}

リクエスト先
: `(ドキュメントルート)/tables/(テーブル名)`

リクエストメソッド
: `GET`

リクエストのURLパラメータ
: [検索リクエストのパラメータ](#parameters)に対応する以下のパラメータを受け付けます:
  
   * `query`: [`(root).(テーブル名).condition.query`](#usage-condition-query-syntax) に対応する文字列。
   * `match_to`: [`(root).(テーブル名).condition.matchTo`](#usage-condition-query-syntax) に対応するカンマ区切りの文字列。
   * `sort_by`:  [`(root).(テーブル名).sortBy`](#query-sortBy) に対応するカンマ区切りの文字列。
   * `attributes`: [`(root).(テーブル名).output.attributes`](#query-output) に対応するカンマ区切りの文字列。
   * `offset`: [`(root).(テーブル名).output.offset`](#query-output) に対応する整数。
   * `limit`: [`(root).(テーブル名).output.limit`](#query-output) に対応する整数。
   * `timeout`: [`(root).timeout`](#parameter-timeout) に対応する整数。

<!--
   * `group_by[(column name)][key]`: A string, applied to [`(root).(column name).groupBy.key`](#query-groupBy).
   * `group_by[(column name)][max_n_sub_records]`: An integer, applied to [`(root).(column name).groupBy.maxNSubRecords`](#query-groupBy).
   * `group_by[(column name)][attributes]`: A comma-separated string, applied to [`(root).(column name).output.attributes`](#query-output).
   * `group_by[(column name)][attributes][(attribute name)][source]`: A string, applied to [`(root).(column name).output.attributes.(attribute name).source`](#query-output).
   * `group_by[(column name)][attributes][(attribute name)][attributes]`: A comma-separated string, applied to [`(root).(column name).output.attributes.(attribute name).attributes`](#query-output).
   * `group_by[(column name)][limit]`: An integer, applied to [`(root).(column name).output.limit`](#query-output).
-->
  
  例:
  
   * `/tables/Store?query=NY&match_to=_key&attributes=_key,*&limit=10`

<!--
   * `/tables/Store?query=NY&match_to=_key&attributes=_key,*&limit=10&group_by[location][key]=location&group_by[location][limit]=5&group_by[location][attributes]=_key,_nsubrecs`
   * `/tables/Store?query=NY&match_to=_key&attributes=_key,*&limit=10&group_by[location][key]=location&group_by[location][limit]=5&group_by[location][attributes][_key][souce]=_key&group_by[location][attributes][_nsubrecs][souce]=_nsubrecs`
   * `/tables/Store?query=NY&match_to=_key&limit=0&group_by[location][key]=location&group_by[location][max_n_sub_records]=5&group_by[location][limit]=5&group_by[location][attributes][sub_records][source]=_subrecs&group_by[location][attributes][sub_records][attributes]=_key,location`
-->

リクエストのbody
: なし。

レスポンスのbody
: [レスポンスメッセージ](#response)。

### Fluentd {#api-types-fluentd}

形式
: Request-Response型。コマンドに対しては必ず対応するレスポンスが返されます。

リクエストの `type`
: `search`

リクエストの `body`
: [パラメータ](#parameters)のハッシュ。

レスポンスの `type`
: `search.result`

## パラメータの構文 {#syntax}

    {
      "timeout" : <タイムアウトするまでの秒数>,
      "queries" : {
        "<クエリ1の名前>" : {
          "source"    : "<検索対象のテーブル名、または別の検索クエリの名前>",
          "condition" : <検索条件>,
          "sortBy"    : <ソートの条件>,
          "groupBy"   : <集約の条件>,
          "output"    : <出力の指定>
        },
        "<クエリ2の名前>" : { ... },
        ...
      }
    }

## 使い方 {#usage}

この項では、以下のテーブルが存在する状態を前提として、典型的な使い方を通じて `search` コマンドの使い方を説明します。

Personテーブル（主キーあり）:

|_key|name|age|sex|job|note|
|Alice Arnold|Alice Arnold|20|female|announcer||
|Alice Cooper|Alice Cooper|30|male|musician||
|Alice Miller|Alice Miller|25|female|doctor||
|Bob Dole|Bob Dole|42|male|lawer||
|Bob Cousy|Bob Cousy|38|male|basketball player||
|Bob Wolcott|Bob Wolcott|36|male|baseball player||
|Bob Evans|Bob Evans|31|male|driver||
|Bob Ross|Bob Ross|54|male|painter||
|Lewis Carroll|Lewis Carroll|66|male|writer|the author of Alice's Adventures in Wonderland|

※`name`、`note` には `TokensBigram` を使用したインデックスが用意されていると仮定します。

### 基本的な使い方 {#usage-basic}

最も単純な例として、Person テーブルのすべてのレコードを出力する例を示します。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source" : "Person",
            "output" : {
              "elements"   : ["count", "records"],
              "attributes" : ["_key", "*"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 9,
             "records" : [
               ["Alice Arnold", "Alice Arnold", 20, "female", "announcer", ""],
               ["Alice Cooper", "Alice Cooper", 30, "male", "musician", ""],
               ["Alice Miller", "Alice Miller", 25, "male", "doctor", ""],
               ["Bob Dole", "Bob Dole", 42, "male", "lawer", ""],
               ["Bob Cousy", "Bob Cousy", 38, "male", "basketball player", ""],
               ["Bob Wolcott", "Bob Wolcott", 36, "male", "baseball player", ""],
               ["Bob Evans", "Bob Evans", 31, "male", "driver", ""],
               ["Bob Ross", "Bob Ross", 54, "male", "painter", ""],
               ["Lewis Carroll", "Lewis Carroll", 66, "male", "writer",
                "the author of Alice's Adventures in Wonderland"]
             ]
           }
         }
       }

`people` は、この検索クエリおよびその処理結果に対して付けた一時的な名前です。
`search` のレスポンスは、検索クエリに付けた名前を伴って返されます。
よって、これは「この検索クエリの結果を `people` と呼ぶ」というような意味合いになります。

どうしてこのコマンドが全レコードのすべての情報を出力するのでしょうか？　これは以下の理由に依ります。

 * 検索条件を何も指定していないため。検索条件を指定しないとすべてのレコードがマッチします。
 * [`output`](#query-output) の `elements` パラメータに `records` （および `count`）が指定されているため。 `elements` は結果に出力する情報を制御します。マッチしたレコードの情報は `records` に、マッチしたレコードの総数は `count` に出力されます。
 * [`output`](#query-output) の `limit` パラメータに `-1` が指定されているため。 `limit` は出力するレコードの最大数の指定ですが、 `-1` を指定するとすべてのレコードが出力されます。
 * [`output`](#query-output) の `attributes` パラメータに `"_key"` と `"*"` の2つが指定されているため（これは「`_key` を含む Person テーブルのすべてのカラムを出力する」という指定で、`["_key", "name", "age", "sex", "job", "note"]` と書くのに等しいです）。 `attributes` は個々のレコードに出力するカラムを制御します。


#### 検索条件 {#usage-condition}

検索条件は `condition` パラメータで指定します。指定方法は、大きく分けて「スクリプト構文形式」と「クエリー構文形式」の2通りがあります。詳細は [`condition` パラメータの仕様](#query-condition) を参照して下さい。

##### スクリプト構文形式の検索条件 {#usage-condition-script-syntax}

スクリプト構文形式は、ECMAScriptの書式に似ています。「`name` に `Alice` を含み、且つ`age` が `25` 以上である」という検索条件は、スクリプト構文形式で以下のように表現できます。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source"    : "Person",
            "condition" : "name @ 'Alice' && age >= 25"
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          }
        }
      }
    }

    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 2,
             "records" : [
               ["Alice Arnold", 20],
               ["Alice Cooper", 30],
               ["Alice Miller", 25]
             ]
           }
         }
       }

スクリプト構文の詳細な仕様は[Groonga のスクリプト構文のリファレンス](http://groonga.org/ja/docs/reference/grn_expr/script_syntax.html)を参照して下さい。

##### クエリー構文形式の検索条件 {#usage-condition-query-syntax}

クエリー構文形式は、主にWebページなどに組み込む検索ボックス向けに用意されています。例えば「検索ボックスに入力された語句を `name` または `note` に含むレコードを検索する」という場面において、検索ボックスに入力された語句が `Alice` であった場合の検索条件は、クエリー構文形式で以下のように表現できます。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source"    : "Person",
            "condition" : {
              "query"   : "Alice",
              "matchTo" : ["name", "note"]
            },
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "note"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 4,
             "records" : [
               ["Alice Arnold", ""],
               ["Alice Cooper", ""],
               ["Alice Miller", ""],
               ["Lewis Carroll",
                "the author of Alice's Adventures in Wonderland"]
             ]
           }
         }
       }

クエリー構文の詳細な仕様は[Groonga のクエリー構文のリファレンス](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html)を参照して下さい。


#### 検索結果のソート {#usage-sort}

出力するレコードのソート条件は `sortBy` パラメータで指定します。以下は、結果を `age` カラムの値の昇順でソートする場合の例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source"    : "Person",
            "condition" : "name @ 'Alice'"
            "sortBy"    : ["age"],
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 8,
             "records" : [
               ["Alice Arnold", 20],
               ["Alice Miller", 25],
               ["Alice Cooper", 30]
             ]
           }
         }
       }

ソートするカラム名の前に `-` を付けると、降順でのソートになります。以下は `age` の降順でソートする場合の例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source"    : "Person",
            "condition" : "name @ 'Alice'"
            "sortBy"    : ["-age"],
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 8,
             "records" : [
               ["Alice Cooper", 30],
               ["Alice Miller", 25],
               ["Alice Arnold", 20]
             ]
           }
         }
       }

詳細は [`sortBy` パラメータの仕様](#query-sortBy) を参照して下さい。

#### 検索結果のページング {#usage-paging}

[`output`](#query-output) パラメータの `offset` と `limit` を指定することで、出力されるレコードの範囲を指定できます。以下は、20件以上ある結果を先頭から順に10件ずつ取得する場合の例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source" : "Person",
            "output" : {
              "elements"   : ["count", "records"],
              "attributes" : ["name"],
              "offset"     : 0,
              "limit"      : 10
            }
          }
        }
      }
    }
    
    => 0件目から9件目までの10件が返される。
    
    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source" : "Person",
            "output" : {
              "elements"   : ["count", "records"],
              "attributes" : ["name"],
              "offset"     : 10,
              "limit"      : 10
            }
          }
        }
      }
    }
    
    => 10件目から19件目までの10件が返される。
    
    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source" : "Person",
            "output" : {
              "elements"   : ["count", "records"],
              "attributes" : ["name"],
              "offset"     : 20,
              "limit"      : 10
            }
          }
        }
      }
    }
    
    => 20件目から29件目までの10件が返される。

`limit` の指定 `-1` は、実際の運用では推奨されません。膨大な量のレコードがマッチした場合、出力のための処理にリソースを使いすぎてしまいますし、ネットワークの帯域も浪費してしまいます。コンピュータの性能にもよりますが、`limit` には `100` 程度までの値を上限として指定し、それ以上のレコードは適宜ページングで取得するようにして下さい。

詳細は [`output` パラメータの仕様](#query-output) を参照して下さい。

また、ページングは [`sortBy` パラメータの機能](#query-sortBy-hash)でも行う事ができ、一般的にはそちらの方が高速に動作します。
よって、可能な限り `output` でのページングよりも `sortBy` でのページングの方を使う事が推奨されます。


#### 出力形式 {#usage-format}

ここまでの例では、レコードの一覧はすべて配列の配列として出力されていました。[`output`](#query-output) パラメータの `format` を指定すると、出力されるレコードの形式を変える事ができます。以下は、`format` に `complex` を指定した場合の例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source" : "Person",
            "output" : {
              "elements"   : ["count", "records"],
              "attributes" : ["_key", "name", "age", "sex", "job", "note"],
              "limit"      : 3,
              "format"     : "complex"
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 9,
             "records" : [
               { "_key" : "Alice Arnold",
                 "name" : "Alice Arnold",
                 "age"  : 20,
                 "sex"  : "female",
                 "job"  : "announcer",
                 "note" : "" },
               { "_key" : "Alice Cooper",
                 "name" : "Alice Cooper",
                 "age"  : 30,
                 "sex"  : "male",
                 "job"  : "musician",
                 "note" : "" },
               { "_key" : "Alice Miller",
                 "name" : "Alice Miller",
                 "age"  : 25,
                 "sex"  : "female",
                 "job"  : "doctor",
                 "note" : "" }
             ]
           }
         }
       }

`format` に `complex` を指定した場合、レコードの一覧はこの例のようにカラム名をキーとしたハッシュの配列として出力されます。
`format` に `simple` を指定した場合、または `format` の指定を省略した場合、レコードの一覧は配列の配列として出力されます。

詳細は [`output` パラメータの仕様](#query-output) および [レスポンスの仕様](#response) を参照して下さい。


### 高度な使い方 {#usage-advanced}

#### 検索結果の集約 {#usage-group}

[`groupBy`](#query-groupBy) パラメータを指定することで、レコードを指定カラムの値で集約した結果を取得することができます。以下は、テーブルの内容を `sex` カラムの値で集約した結果と、集約前のレコードがそれぞれ何件あったかを取得する例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "sexuality" : {
            "source"  : "Person",
            "groupBy" : "sex",
            "output"  : {
              "elements"   : ["count", "records"],
              "attributes" : ["_key", "_nsubrecs"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "sexuality" : {
             "count" : 2,
             "records" :
               ["female", 2],
               ["male", 7]
             ]
           }
         }
       }

上記の結果は、 `sex` の値が `female` であるレコードが2件、`male` であるレコードが7件存在していて、`sex` の値の種類としては2通りが登録されている事を示しています。

また、集約前のレコードを代表値として取得する事もできます。以下は、`sex` カラムの値で集約した結果と、それぞれの集約前のレコードを2件ずつ取得する例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "sexuality" : {
            "source"  : "Person",
            "groupBy" : {
              "keys"           : "sex",
              "maxNSubRecords" : 2
            },
            "output"  : {
              "elements"   : ["count", "records"],
              "attributes" : [
                "_key",
                "_nsubrecs",
                { "label"      : "subrecords",
                  "source"     : "_subrecs",
                  "attributes" : ["name"] }
              ],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "sexuality" : {
             "count" : 2,
             "records" :
               ["female", 2, [["Alice Arnold"], ["Alice Miller"]]],
               ["male",   7, [["Alice Cooper"], ["Bob Dole"]]]
             ]
           }
         }
       }


詳細は [`groupBy` パラメータの仕様](#query-groupBy) を参照して下さい。


#### 複数の検索クエリの列挙 {#usage-multiple-queries}

`search` は、一度に複数の検索クエリを受け付ける事ができます。以下は、`age` が `25` 以下のレコードと `age` が `40` 以上のレコードを同時に検索する例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "junior" : {
            "source"    : "Person",
            "condition" : "age <= 25",
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          },
          "senior" : {
            "source"    : "Person",
            "condition" : "age >= 40",
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "junior" : {
             "count" : 2,
             "records" : [
               ["Alice Arnold", 20],
               ["Alice Miller", 25]
             ]
           },
           "senior" : {
             "count" : 3,
             "records" : [
               ["Bob Dole", 42],
               ["Bob Ross", 54],
               ["Lewis Carroll", 66]
             ]
           }
         }
       }

レスポンスに含まれる検索結果は、各クエリに付けた一時的な名前で識別することになります。

#### 検索のチェーン {#usage-chain}

検索クエリを列挙する際は、`source` パラメータの値として実在するテーブルの名前だけでなく、別の検索クエリに付けた一時的な名前を指定する事ができます。これにより、1つの検索クエリでは表現できない複雑な検索を行う事ができます。

以下は、Personテーブルについて `name` カラムが `Alice` を含んでいるレコードを検索た結果と、それをさらに `sex` カラムの値で集約した結果を同時に取得する例です。

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "people" : {
            "source"    : "Person",
            "condition" : "name @ 'Alice'"
            "output"    : {
              "elements"   : ["count", "records"],
              "attributes" : ["name", "age"],
              "limit"      : -1
            }
          },
          "sexuality" : {
            "source"  : "people",
            "groupBy" : "sex",
            "output"  : {
              "elements"   : ["count", "records"],
              "attributes" : ["_key", "_nsubrecs"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "people" : {
             "count" : 8,
             "records" : [
               ["Alice Cooper", 30],
               ["Alice Miller", 25],
               ["Alice Arnold", 20]
             ]
           },
           "sexuality" : {
             "count" : 2,
             "records" :
               ["female", 2],
               ["male", 1]
             ]
           }
         }
       }

個々の検索クエリの結果は出力しない（中間テーブルとしてのみ使う）事もできます。
以下は、Personテーブルについて `job` カラムの値で集約した結果をまず求め、そこからさらに `player` という語句を含んでいる項目に絞り込む例です。
（※この場合の2つ目の検索ではインデックスが使用されないため、検索処理が遅くなる可能性があります。）

    {
      "type" : "search",
      "body" : {
        "queries" : {
          "allJob" : {
            "source"  : "Person",
            "groupBy" : "job"
          },
          "playerJob" : {
            "source"    : "allJob",
            "condition" : "_key @ `player`",
            "output"  : {
              "elements"   : ["count", "records"],
              "attributes" : ["_key", "_nsubrecs"],
              "limit"      : -1
            }
          }
        }
      }
    }
    
    => {
         "type" : "search.result",
         "body" : {
           "playerJob" : {
             "count" : 2,
             "records" : [
               ["basketball player", 1],
               ["baseball player", 1]
             ]
           }
         }
       }


## パラメータの詳細 {#parameters}

### 全体のパラメータ {#container-parameters}

#### `timeout` {#parameter-timeout}

*Note: This parameter is not implemented yet on the version 1.0.4.

概要
: 検索処理がタイムアウトするまでの時間を指定します。

値
: タイムアウトするまでの時間の数値（単位：ミリ秒）。

省略時の初期値
: 10000（10秒）

指定した時間以内に Droonga Engine が検索の処理を完了できなかった場合、Droonga はその時点で検索処理を打ち切り、エラーを返却します。
クライアントは、この時間を過ぎた後は検索処理に関するリソースを解放して問題ありません。

#### `queries` {#parameter-queries}

概要
: 検索クエリとして、検索の条件と出力の形式を指定します。

値
: 個々の検索クエリの名前をキー、[個々の検索クエリ](#query-parameters)の内容を値としたハッシュ。

省略時の既定値
: なし。このパラメータは必須です。

`search` は、複数の検索クエリを一度に受け取る事ができます。

バージョン 1.0.4 ではすべての検索クエリの結果を一度にレスポンスとして返却する動作のみ対応していますが、将来的には、それぞれの検索クエリの結果を分割して受け取る（結果が出た物からバラバラに受け取る）動作にも対応する予定です。

### 個々の検索クエリのパラメータ {#query-parameters}

#### `source` {#query-source}

概要
: 検索対象とするデータソースを指定します。

値
: テーブル名の文字列、または結果を参照する別の検索クエリの名前の文字列。

省略時の既定値
: なし。このパラメータは必須です。

別の検索クエリの処理結果をデータソースとして指定する事により、ファセット検索などを行う事ができます。

なお、その場合の各検索クエリの実行順（依存関係）は Droonga が自動的に解決します。
依存関係の順番通りに各検索クエリを並べて記述する必要はありません。

#### `condition` {#query-condition}

概要
: 検索の条件を指定します。

値
: 以下のパターンのいずれかをとります。
  
  1. [スクリプト構文](http://groonga.org/ja/docs/reference/grn_expr/script_syntax.html)形式の文字列。
  2. [スクリプト構文](http://groonga.org/ja/docs/reference/grn_expr/script_syntax.html)形式の文字列を含むハッシュ。
  3. [クエリー構文](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html)形式の文字列を含むハッシュ。
  4. 1〜3および演算子の文字列の配列。 

省略時の既定値
: なし。

検索条件を指定しなかった場合、データソースに含まれるすべてのレコードが検索結果として取り出され、その後の処理に使われます。

##### スクリプト構文形式の文字列による検索条件 {#query-condition-script-syntax-string}

以下のような形式の文字列で検索条件を指定します。

    "name == 'Alice' && age >= 20"

上記の例は「 `name` カラムの値が `"Alice"` と等しく、且つ `age` カラムの値が20以上である」という意味になります。

詳細は[Groonga のスクリプト構文のリファレンス](http://groonga.org/ja/docs/reference/grn_expr/script_syntax.html)を参照して下さい。

##### スクリプト構文形式の文字列を含むハッシュによる検索条件 {#query-condition-script-syntax-hash}

[スクリプト構文形式の文字列による検索条件](#query-condition-script-syntax-string)をベースとした、以下のような形式のハッシュで検索条件を指定します。

    {
      "script"      : "name == 'Alice' && age >= 20",
      "allowUpdate" : true
    }

（詳細未稿：仕様が未確定、動作が不明、未実装のため）

##### クエリー構文形式の文字列を含むハッシュ {#query-condition-query-syntax-hash}

以下のような形式のハッシュで検索条件を指定します。

    {
      "query"                    : "Alice",
      "matchTo"                  : ["name * 2", "job * 1"],
      "defaultOperator"          : "&&",
      "allowPragma"              : true,
      "allowColumn"              : true,
      "matchEscalationThreshold" : 10
    }

`query`
: クエリを文字列で指定します。
  詳細は[Groonga のクエリー構文の仕様](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html)を参照して下さい。
  このパラメータは省略できません。


: 検索対象のカラムを、カラム名の文字列またはその配列で指定します。
  カラム名の後に `name * 2` のような指定を加える事で、重み付けができます。
  このパラメータは省略可能で、省略時の初期値は `"_key"` です。

`defaultOperator`: `query` に複数のクエリが列挙されている場合の既定の論理演算の条件を指定します。
  以下のいずれかの文字列を指定します。
  
   * `"&&"` : AND条件と見なす。
   * `"||"` : OR条件と見なす。
   * `"-"`  : [論理否定](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html#logical-not)条件と見なす。
  
  このパラメータは省略可能で、省略時の初期値は `"&&"` です。

`allowPragma`
: `query` の先頭において、`*E-1` のようなプラグマの指定を許容するかどうかを真偽値で指定します。
  このパラメータは省略可能で、省略時の初期値は `true` （プラグマの指定を許容する）です。

`allowColumn`
: `query` において、カラム名を指定した `name:Alice` のような書き方を許容するかどうかを真偽値で指定します。
  このパラメータは省略可能で、省略時の初期値は `true` （カラム名の指定を許容する）です。

`matchEscalationThreshold`
: 検索方法をエスカレーションするかどうかを決定するための閾値を指定します。
  インデックスを用いた全文検索のヒット件数がこの閾値以下であった場合は、非分かち書き検索、部分一致検索へエスカレーションします。
  詳細は [Groonga の検索の仕様の説明](http://groonga.org/ja/docs/spec/search.html)を参照して下さい。
  このパラメータは省略可能で、省略時の初期値は `0` です。


##### 配列による検索条件 {#query-condition-array}

以下のような形式の配列で検索条件を指定します。

    [
      "&&",
      <検索条件1>,
      <検索条件2>,
      ...
    ]

配列の最初の要素は、論理演算子を以下のいずれかの文字列で指定します。

 * `"&&"` : AND条件と見なす。
 * `"||"` : OR条件と見なす。
 * `"-"`  : [論理否定](http://groonga.org/ja/docs/reference/grn_expr/query_syntax.html#logical-not)条件と見なす。

配列の2番目以降の要素で示された検索条件について、1番目の要素で指定した論理演算子による論理演算を行います。
例えば以下は、スクリプト構文形式の文字列による検索条件2つによるAND条件であると見なされ、「 `name` カラムの値が `"Alice"` と等しく、且つ `age` カラムの値が20以上である」という意味になります。

    ["&&", "name == 'Alice'", "age >= 20"]

配列を入れ子にする事により、より複雑な検索条件を指定する事もできます。
例えば以下は、「 `name` カラムの値が `"Alice"` と等しく、且つ `age` カラムの値が20以上であるが、 `job` カラムの値が `"engineer"` ではない」という意味になります。

    [
      "-",
      ["&&", "name == 'Alice'", "age >= 20"],
      "job == 'engineer'"
    ]

#### `sortBy` {#query-sortBy}

概要
: ソートの条件および取り出すレコードの範囲を指定します。

値
: 以下のパターンのいずれかをとります。
  
  1. カラム名の文字列の配列。
  2. ソート条件と取り出すレコードの範囲を指定するハッシュ。 

省略時の既定値
: なし。

ソート条件が指定されなかった場合、すべての検索結果がそのままの並び順でソート結果として取り出され、その後の処理に使われます。

##### 基本的なソート条件の指定 {#query-sortBy-array}

ソート条件はカラム名の文字列の配列として指定します。

Droongaはまず最初に指定したカラムの値でレコードをソートし、カラムの値が同じレコードが複数あった場合は2番目に指定したカラムの値でさらにソートする、という形で、すべての指定カラムの値に基づいてソートを行います。

ソート対象のカラムを1つだけ指定する場合であっても、必ず配列として指定する必要があります。

ソート順序は指定したカラムの値での昇順となります。カラム名の前に `-` を加えると降順となります。

例えば以下は、「 `name` の値で昇順にソートし、同じ値のレコードはさらに `age` の値で降順にソートする」という意味になります。

    ["name", "-age"]

##### ソート結果から取り出すレコードの範囲の指定 {#query-sortBy-hash}

ソートの指定において、以下の形式でソート結果から取り出すレコードの範囲を指定する事ができます。

    {
      "keys"   : [<ソート対象のカラム>],
      "offset" : <ページングの起点>,
      "limit"  : <取り出すレコード数>
    }

`keys`
: ソート条件を[基本的なソート条件の指定](#query-sortBy-array)の形式で指定します。
  このパラメータは省略できません。

`offset`
: 取り出すレコードのページングの起点を示す `0` または正の整数。
  
  このパラメータは省略可能で、省略時の既定値は `0` です。

`limit`
: 取り出すレコード数を示す `-1` 、 `0` 、または正の整数。
  `-1`を指定すると、すべてのレコードを取り出します。
  
  このパラメータは省略可能で、省略時の既定値は `-1` です。

例えば以下は、ソート結果の10番目から19番目までの10件のレコードを取り出すという意味になります。

    {
      "keys"   : ["name", "-age"],
      "offset" : 10,
      "limit"  : 10
    }

これらの指定を行った場合、取り出されたレコードのみがその後の処理の対象となります。
そのため、 `output` における `offset` および `limit` の指定よりも高速に動作します。


#### `groupBy` {#query-groupBy}

概要
: 処理対象のレコード群を集約する条件を指定します。

値
: 以下のパターンのいずれかをとります。
  
  1. 基本的な集約条件（カラム名または式）の文字列。
  2. 複雑な集約条件を指定するハッシュ。 

省略時の既定値
: なし。

集約条件を指定した場合、指定に基づいてレコードを集約した結果がレコードとして取り出され、その後の処理に使われます。

##### 基本的な集約条件の指定 {#query-groupBy-string}

基本的な集約条件では、処理対象のレコード群が持つカラムの名前を文字列として指定します。

Droongaはそのカラムの値が同じであるレコードを集約し、カラムの値をキーとした新しいレコード群を結果として出力します。
集約結果のレコードは以下のカラムを持ちます。

`_key`
: 集約前のレコード群における、集約対象のカラムの値です。

`_nsubrecs`
: 集約前のレコード群における、集約対象のカラムの値が一致するレコードの総数を示す数値です。

例えば以下は、`job` カラムの値でレコードを集約し、`job` カラムの値としてどれだけの種類が存在しているのか、および、各 `job` の値を持つレコードが何件存在しているのかを集約結果として取り出すという意味になります。

    "job"

##### 複雑な集約条件の指定 {#query-groupBy-hash}

集約の指定において、集約結果の一部として出力する集約前のレコードの数を、以下の形式で指定する事ができます。

    {
      "key"            : "<基本的な集約条件>",
      "maxNSubRecords" : <集約結果の一部として出力する集約前のレコードの数>
    }

`key`
: [基本的な集約条件の指定](#query-groupBy-string)の形式による、集約条件を指定する文字列。
  このパラメータは省略できません。

`maxNSubRecords`
: 集約結果の一部として出力する集約前のレコードの最大数を示す `0` または正の整数。
  `-1` は指定できません。
  
  このパラメータは省略可能で、省略時の既定値は `0` です。
  
例えば以下は、`job` カラムの値でレコードを集約した結果について、各 `job` カラムの値を含んでいるレコードを代表として1件ずつ取り出すという意味になります。

    {
      "key"            : "job",
      "maxNSubRecords" : 1
    }

集約結果のレコードは、[基本的な集約条件の指定](#query-groupBy-string)の集約結果のレコード群が持つすべてのカラムに加えて、以下のカラムを持ちます。

`_subrecs`
: 集約前のレコード群における、集約対象のカラムの値が一致するレコードの配列。
  
※バージョン 1.0.4 では、データセットが複数のボリュームに別れている場合、集約前のレコードの代表が `maxNSubRecords` で指定した数よりも多く返される場合があります。これは既知の問題で、将来のバージョンで修正される予定です。


#### `output` {#query-output}

概要
: 処理結果の出力形式を指定します。

値
: 出力形式を指定するハッシュ。 

省略時の既定値
: なし。

指定を省略した場合、その検索クエリの検索結果はレスポンスには出力されません。
集約操作などのために必要な中間テーブルにあたる検索結果を求めるだけの検索クエリにおいては、 `output` を省略して処理時間や転送するデータ量を減らすことができます。

出力形式は、以下の形式のハッシュで指定します。

    {
      "elements"   : [<出力する情報の名前の配列>],
      "format"     : "<検索結果のレコードの出力スタイル>",
      "offset"     : <ページングの起点>,
      "limit"      : <出力するレコード数>,
      "attributes" : <レコードのカラムの出力指定の配列>
    }

`elements`
: その検索クエリの結果として[レスポンス](#response)に出力する情報を、プロパティ名の文字列の配列で指定します。
  以下の項目を指定できます。項目は1つだけ指定する場合であっても必ず配列で指定します。
  
   * `"startTime"`
   * `"elapsedTime"`
   * `"count"`
   * `"attributes"`
   * `"records"`
  
  このパラメータは省略可能で、省略時の初期値はありません（結果を何も出力しません）。

`format`
: 検索結果のレコードの出力スタイルを指定します。
  以下のいずれかの値（文字列）を取ります。
  
   * `"simple"`  : 個々のレコードを配列として出力します。
   * `"complex"` : 個々のレコードをハッシュとして出力します。
  
  このパラメータは省略可能で、省略時の初期値は `"simple"` です。

`offset`
: 出力するレコードのページングの起点を示す `0` または正の整数。
  
  このパラメータは省略可能で、省略時の既定値は `0` です。

`limit`
: 出力するレコード数を示す `-1` 、 `0` 、または正の整数。
  `-1`を指定すると、すべてのレコードを出力します。
  
  このパラメータは省略可能で、省略時の既定値は `0` です。

`attributes`
: レコードのカラムの値について、出力形式を配列で指定します。
  個々のカラムの値の出力形式は以下のいずれかで指定します。
  
   1. カラムの定義の配列。
   2. カラムの定義を値としたハッシュ。
  
  各カラムは以下の形式のいずれかで指定します。
  
   * カラム名の文字列。例は以下の通りです。
     * `"name"` : `name` カラムの値をそのまま `name` カラムとして出力します。
     * `"age"`  : `age` カラムの値をそのまま `age` カラムとして出力します。
   * 詳細な出力形式指定のハッシュ。例は以下の通りです。
     * 以下の例は、 `name` カラムの値を `realName` カラムとして出力します。
       
           { "label" : "realName", "source" : "name" }
       
     * 以下の例は、 `name` カラムの値について、全文検索にヒットした位置を強調したHTMLコード片の文字列を `html` カラムとして出力します。
       
           { "label" : "html", "source": "snippet_html(name)" }
       
     * 以下の例は、`country` カラムについて、すべてのレコードの当該カラムの値が文字列 `"Japan"` であるものとして出力します。
       （存在しないカラムを実際に作成する前にクライアント側の挙動を確認したい場合などに、この機能が利用できます。）
       
           { "label" : "country", "source" : "'Japan'" }
       
     * 以下の例は、集約前の元のレコードの総数を、集約後のレコードの `"itemsCount"` カラムの値として出力します。
       
           { "label" : "itemsCount", "source" : "_nsubrecs", }
       
     * 以下の例は、集約前の元のレコードの配列を、集約後のレコードの `"items"` カラムの値として出力します。
       `"attributes"` は、この項の説明と同じ形式で指定します。
       
           { "label" : "items", "source" : "_subrecs",
             "attributes": ["name", "price"] }
  
  カラムの定義の配列には、上記の形式で示されたカラムの定義を0個以上含めることができます。例：
  
      [
        "name",
        "age",
        { "label" : "realName", "source" : "name" }
      ]
  
  この場合、「`_key` のような特殊なカラムを除くすべてのカラム」を意味する特別なカラム名 `"*"`を使用できます。
  
    * `["*"]` と指定すると、（`_key` や `_id` 以外の）すべてのカラムがそのままの形で出力されます。
    * `["_key", "*"]` と指定すると、 `_key` に続いてすべてのカラムがそのままの形で出力されます。
    * `["*", "_nsubrecs"]` と指定すると、 すべてのカラムがそのままの形で出力された後に `_nsubrecs` が出力されます。
  
  カラムの定義を値としたハッシュでは、カラムの出力名をキー、上記の形式で示されたカラムの定義を値として、カラムの定義を0個以上含めることができます。例：
  
      {
        "name"     : "name",
        "age"      : "age",
        "realName" : { "source" : "name" },
        "country"  : { "source" : "'Japan'" }
      }
  
  このパラメータは省略可能で、省略時の既定値はありません。カラムの指定がない場合、カラムの値は一切出力されません。


## レスポンス {#response}

このコマンドは、検索結果を`body` 、ステータスコード `200` を `statusCode` の値としたレスポンスを返します。

検索結果のハッシュは、個々の検索クエリの名前をキー、対応する[個々の検索クエリ](#query-parameters)の処理結果を値とした、以下のような形式を取ります。

    {
      "<クエリ1の名前>" : {
        "startTime"   : "<検索を開始した時刻>",
        "elapsedTime" : <検索にかかった時間（単位：ミリ秒）),
        "count"       : <指定された検索条件に該当するレコードの総数>,
        "attributes"  : <出力されたレコードのカラムの情報の配列またはハッシュ>,
        "records"     : [<出力されたレコードの配列>]
      },
      "<クエリ2の名前>" : { ... },
      ...
    }

検索クエリの処理結果のハッシュは以下の項目を持つことができ、[検索クエリの `output`](#query-output) の `elements` で明示的に指定された項目のみが出力されます。

### `startTime` {#response-query-startTime}

検索を開始した時刻（ローカル時刻）の文字列です。

形式は、[W3C-DTF](http://www.w3.org/TR/NOTE-datetime "Date and Time Formats")のタイムゾーンを含む形式となります。
例えば以下の要領です。

    2013-11-29T08:15:30+09:00

### `elapsedTime` {#response-query-elapsedTime}

検索にかかった時間の数値（単位：ミリ秒）です。

### `count` {#response-query-count}

検索条件に該当するレコードの総数の数値です。
この値は、検索クエリの [`sortBy`](#query-sortBy) や [`output`](#query-output) における `offset` および `limit` の指定の影響を受けません。

### `attributes` および `records` {#response-query-attributes-and-records}

 * `attributes` は出力されたレコードのカラムの情報を示す配列またはハッシュです。
 * `records` は出力されたレコードの配列です。

`attributes` および `records` の出力形式は[検索クエリの `output`](#query-output) の `format` の指定に従って以下の2通りに別れます。

#### 単純な形式のレスポンス {#response-query-simple-attributes-and-records}

`format` が　`"simple"` の場合、個々の検索クエリの結果は以下の形を取ります。

    {
      "startTime"   : "<検索を開始した時刻>",
      "elapsedTime" : <検索にかかった時間),
      "count"       : <検索結果のレコードの総数>,
      "attributes"  : [
        { "name"   : "<カラム1の名前>",
          "type"   : "<カラム1の型>",
          "vector" : <カラム1がベクターカラムかどうか> },
        { "name"   : "<カラム2の名前>",
          "type"   : "<カラム2の型>",
          "vector" : <カラム2がベクターカラムかどうか> },
        { "name"       : "<カラム3(サブレコードが存在する場合)の名前>"
          "attributes" : [
          { "name"   : "<カラム3-1のカラムの名前>",
            "type"   : "<カラム3-1のカラムの型>",
            "vector" : <カラム3-1がベクターカラムかどうか> },
          { "name"   : "<カラム3-2のカラムの名前>",
            "type"   : "<カラム3-2のカラムの型>",
            "vector" : <カラム3-2がベクターカラムかどうか> },
          ],
          ...
        },
        ...
      ],
      "records"     : [
        [<レコード1のカラム1の値>,
         <レコード1のカラム2の値>,
         [
          [<レコード1のサブレコード1のカラム3-1の値>,
           <レコード1のサブレコード1のカラム3-2の値>,
           ...],
          [<レコード1のサブレコード2のカラム3-1の値>,
           <レコード1のサブレコード2のカラム3-2の値>,
           ...],
          ...],
         ...],
        [<レコード2のカラム1の値>,
         <レコード2のカラム2の値>,
         [
          [<レコード2のサブレコード1のカラム3-1の値>,
           <レコード2のサブレコード1のカラム3-2の値>,
           ...],
          [<レコード2のサブレコード2のカラム3-1の値>,
           <レコード2のサブレコード2のカラム3-2の値>,
           ...],
          ...],
         ...],
        ...
      ]
    }

これは、受け取ったデータの扱いやすさよりも、データの転送量を小さく抑える事を優先する出力形式です。
大量のレコードを検索結果として受け取る場合や、多量のアクセスが想定される場合などに適しています。

##### `attributes` {#response-query-simple-attributes}

出力されたレコードのカラムについての情報の配列で、[検索クエリの `output`](#query-output) における `attributes` で指定された順番で個々のカラムの情報を含みます。

個々のカラムの情報はハッシュの形をとり、その形式はレコードの値に応じて以下の3種類で与えられます。ハッシュのキーと値は以下のとおりです。

###### 通常のカラム

`name`
: カラムの出力名の文字列です。[検索クエリの `output`](#query-output) における `attributes` の指定内容に基づきます。

`type`
: カラムの値の型を示す文字列です。
  値は[Groonga のプリミティブなデータ型](http://groonga.org/ja/docs/reference/types.html)の名前か、もしくはテーブル名です。

`vector`
: カラムが[ベクター型](http://groonga.org/ja/docs/tutorial/data.html#vector-types)かどうかを示す真偽値です。
  以下のいずれかの値をとります。
  
   * `true`  : カラムはベクター型である。
   * `false` : カラムはベクター型ではない（スカラー型である）。

###### サブレコードに対応するカラム

`name`
: カラムの出力名の文字列です。[検索クエリの `output`](#query-output) における `attributes` の指定内容に基づきます。

サブレコードのカラム情報を含む配列です。この形式は主レコードの `attributes` と同様です。つまり `attribuets` は再帰的な構造になっています。

###### 式

`name`
: カラムの出力名の文字列です。[検索クエリの `output`](#query-output) における `attributes` の指定内容に基づきます。

##### `records` {#response-query-simple-records}

出力されたレコードの配列です。

個々のレコードは配列の形をとり、[検索クエリの `output`](#query-output) における `attributes` で指定された各カラムの値を同じ順番で含みます。

[日時型](http://groonga.org/ja/docs/tutorial/data.html#date-and-time-type)のカラムの値は、[W3C-DTF](http://www.w3.org/TR/NOTE-datetime "Date and Time Formats")のタイムゾーンを含む形式の文字列として出力されます。

#### 複雑な形式のレスポンス {#response-query-complex-attributes-and-records}

`format` が　`"complex"` の場合、個々の検索クエリの結果は以下の形を取ります。

    {
      "startTime"   : "<検索を開始した時刻>",
      "elapsedTime" : <検索にかかった時間),
      "count"       : <検索結果のレコードの総数>,
      "attributes"  : {
        "<カラム1の名前>" : { "type"   : "<カラム1の型>",
                            "vector" : <カラム1がベクターカラムかどうか> },
        "<カラム2の名前>" : { "type"   : "<カラム2の型>",
                            "vector" : <カラム2がベクターカラムかどうか> },
        "<カラム3(サブレコードが存在する場合)の名前>" : {
          "attributes" : {
            "<カラム3-1の名前>" : { "type"   : "<カラム3-1の型>",
                                  "vector" : <カラム3-1がベクターカラムかどうか> },
            "<カラム3-2の名前>" : { "type"   : "<カラム3-2の型>",
                                  "vector" : <カラム3-2がベクターカラムかどうか> },
            ...
          }
        },
        ...
      ],
      "records"     : [
        { "<カラム1の名前>" : <レコード1のカラム1の値>,
          "<カラム2の名前>" : <レコード1のカラム2の値>,
          "<カラム3の名前(サブレコードが存在する場合)>" : [
            { "<カラム3-1の名前>" : <レコード1のサブレコード1のカラム3-1の値>,
              "<カラム3-2の名前>" : <レコード1のサブレコード1のカラム3-2の値>,
              ... },
            { "<カラム3-1の名前>" : <レコード1のサブレコード2のカラム3-1の値>,
              "<カラム3-2の名前>" : <レコード1のサブレコード2のカラム3-2の値>,
              ... },
            ...
          ],
          ...                                                                },
        { "<カラム1の名前>" : <レコード2のカラム1の値>,
          "<カラム2の名前>" : <レコード2のカラム2の値>,
          "<カラム3の名前(サブレコードが存在する場合)>" : [
            { "<カラム3-1の名前>" : <レコード2のサブレコード1のカラム3-1の値>,
              "<カラム3-2の名前>" : <レコード2のサブレコード1のカラム3-2の値>,
              ... },
            { "<カラム3-1の名前>" : <レコード2のサブレコード2のカラム3-1の値>,
              "<カラム3-2の名前>" : <レコード2のサブレコード2のカラム3-2の値>,
              ... },
            ...
          ],
          ...                                                                },
        ...
      ]
    }

これは、データの転送量を小さく抑える事よりも、受け取ったデータの扱いやすさを優先する出力形式です。
検索結果の件数が小さい事があらかじめ分かっている場合や、管理機能などのそれほど多量のアクセスが見込まれない場合などに適しています。

##### `attributes` {#response-query-complex-attributes}

出力されたレコードのカラムについての情報を含むハッシュで、[検索クエリの `output`](#query-output) における `attributes` で指定された出力カラム名がキー、カラムの情報が値となります。

個々のカラムの情報はハッシュの形をとり、その形式はレコードの値に応じて以下の3種類で与えられます。ハッシュのキーと値は以下のとおりです。

###### 通常のカラム

`type`
: カラムの値の型を示す文字列です。
  値は[Groonga のプリミティブなデータ型](http://groonga.org/ja/docs/reference/types.html)の名前か、もしくはテーブル名です。

`vector`
: カラムが[ベクター型](http://groonga.org/ja/docs/tutorial/data.html#vector-types)かどうかを示す真偽値です。
  以下のいずれかの値をとります。
  
   * `true`  : カラムはベクター型である。
   * `false` : カラムはベクター型ではない（スカラー型である）。

###### サブレコードに対応するカラム

サブレコードのカラム情報を含む配列です。この形式は主レコードの `attributes` と同様です。つまり `attribuets` は再帰的な構造になっています。

###### 式

キーはありません。空のハッシュ `{}` です。

##### `records` {#response-query-complex-records}


出力されたレコードの配列です。

個々のレコードは、[検索クエリの `output`](#query-output) における `attributes` で指定された出力カラム名をキー、カラムの値を値としたハッシュとなります。

[日時型](http://groonga.org/ja/docs/tutorial/data.html#date-and-time-type)のカラムの値は、[W3C-DTF](http://www.w3.org/TR/NOTE-datetime "Date and Time Formats")のタイムゾーンを含む形式の文字列として出力されます。


## エラーの種類 {#errors}

このコマンドは[一般的なエラー](/ja/reference/message/#error)に加えて、以下のエラーを場合に応じて返します。

### `MissingSourceParameter`

`source` の指定がないクエリがあることを示します。ステータスコードは `400` です。

### `UnknownSource`

`source` の値として、他のクエリの名前ではない、実際には存在しないテーブルの名前が指定されていることを示します。ステータスコードは `404` です。

### `CyclicSource`

`source` の循環参照があることを示します。ステータスコードは `400` です。

### `SearchTimeout`

`timeout` で指定された時間内に検索処理が完了しなかったことを示します。ステータスコードは `500` です。
