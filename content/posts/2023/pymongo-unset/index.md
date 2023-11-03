---
title: "MongoDBのネストされたデータの一部を $unset を利用して削除する"
date: 2023-11-03T16:30:00
description: "MongoDBをPyMongoで触っているときにドキュメントのバリューが辞書型（ネストしている）データの中から、特定の要素を削除（$unset）することがあったのでやり方のメモ"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/mongodb.png
meta_image: images/cover/top-cover.png
categories:
 - 技術系備忘録
tags:
 - Python
 - PyMongo
 - MongoDB
---

アウトプット強化月間2本目は、MongoDBのちょっとした操作に関する備忘録です。業務ではDocumentDBを使っているのですが、基本的にMongoDBと扱いは同じなのでMongoDBで説明していきます（個人で使うには無料枠があるという理由もあり）。

## やりたいこと

以下のようなコレクションがあったとします。なお簡略化のために `_id` は省いています。

```json
[
    {
        "name":"ヴィッセル神戸",
        "players":{"1":"前川","2":"飯野","3":"トゥーレル"}
    }
    {
        "name":"ガンバ大阪",
        "players":{"1":"東口","2":"福岡","3":"半田"}
    }
]

```

J1リーグのチームごとにドキュメントが存在し、チーム情報が保存されているイメージです。この時、背番号がkeyで選手名がvalueとなっている辞書型データがあり、 `players`というフィールドがこのデータを保持しています。

まさにネストしている状態ですが、このデータの中から特定の選手のデータを消したり、追加することは当然考えられます（選手の移籍ですね）。今回はこのデータ処理方法についてPyMongoでどのように操作するかまとめます。

MongoDBではupdate系のメソッドにおいて `$unset`、 `$set` という演算子が存在しており、これらを実現することができます。

## $unset で一部を削除更新する

具体例として、ヴィッセル神戸の背番号3トゥーレル選手が脱退してしまったとしましょう（起きてほしくないですが）。この場合のプログラムの一例は以下となります。

```python
from pymongo.mongo_client import MongoClient
from pymongo.server_api import ServerApi


mongo_uri = "接続先情報"

# コレクションへのアクセス
client = MongoClient(mongo_uri, server_api=ServerApi("1"))
db = client.j1
collection = db.teams

collection.update_one(filter={"name": "ヴィッセル神戸"}, update={"$unset": {"players.3": ""}})
```

ポイントはネストしている分のフィールド名を `players.3`のように`.`でつないでいることです。valueとして空文字を指定していますが、`$unset`演算子においてはここに何を入れても問題ありません。

なお、指定したフィールド名が存在しない場合は何も起きません。この辺りは[公式ドキュメント](https://www.mongodb.com/docs/manual/reference/operator/update/unset/)を参考にするとよいと思います。

## $set で一部を追加更新する

トゥーレルが脱退してしまった状態は悲しいので、元に戻したいと思います。やり方は先ほどの `$unset` を `$set` に変えるだけです。

```python
from pymongo.mongo_client import MongoClient
from pymongo.server_api import ServerApi


mongo_uri = "接続先情報"

# コレクションへのアクセス
client = MongoClient(mongo_uri, server_api=ServerApi("1"))
db = client.j1
collection = db.teams

collection.update_one(filter={"name": "ヴィッセル神戸"}, update={"$set": {"players.3": "トゥーレル"}})
```

なお、`$set` と `$unset` は同時に使うことができます。例えばですが「トゥーレル選手が移籍し、代わりに背番号64のマタ選手が入ってきた」みたいな状況を考えると、以下のようになります。

```python
collection.update_one(
    filter={"name": "ヴィッセル神戸"}, 
    update={"$set":{"players.64": "マタ"},"$unset": {"players.3": ""}}
)
```

## まとめ

MongoDBのネストされたデータの操作について紹介いたしました。MongoDB（DocumentDB）に関しては今後触る機会が増えそうな気がするので、継続してキャッチアップや検証を進めていきたいと思っています。