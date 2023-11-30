---
title: "Pydanticで定義したクエリパラメータをFastAPIのドキュメントに反映する"
date: 2022-10-05
description: "Pydanticで定義したモデルを、FastAPIのクエリパラメータとしてDependsしたときに、自動生成されるOpenAPIに詳細情報（説明文）を表示させる方法。"
aliases:
 - /posts/fastapi-tips-01/
categories:
 - テック系
tags:
 - Python
 - FastAPI
 - Pydantic
---

タイトルの通り、FastAPIで自動生成されるOpenAPIドキュメント（Swagger）内で、Pydanticで定義したクラスをDependsしたクエリパラメータに対して、情報（説明文）をつける方法について紹介します。\
※途中で触れますが、今回の解決方法はその場しのぎ的で奇麗な形ではありません

これからの説明では、単純にクエリパラメータとして受け取った文字列（名前）を、挨拶文にして返すAPIを例に見ていきます。（「Taro」を受け取って「Hello Taro!」を返す）

※[手っ取り早く実装例を見に行く](#pydanticを利用する場合)

## Pydanticモデルを利用しない場合

Pydanticを利用せずにFastAPIのみで完結させる場合は、簡単にドキュメントへ説明文を反映させることができます。

### シンプルな定義

まずは、最低限の実装で確認してみます。こちらの方法では、ドキュメントに説明文をつけることはできません。

```main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/hello")
async def hello(name: str):
    return {"message": f"Hello {name}!"}
```

この状態でドキュメントを確認してみます。説明文はありません。**変数名で伝わるので、必要ないと考えることもできます。**

![シンプルな定義](OpenAPI-01.png "シンプルな定義")

### Queryを使った定義

`fastapi`から`Query`をインポートして、クエリパラメータの説明を加えていきます。`Query`は様々なパラメータを持っており、制約（文字数制限等）を加えることもできます。今回は、あくまでドキュメントに情報を加えるだけなので、制約などについては触れません。詳しく見たい方は公式ドキュメントをご覧ください。

公式ドキュメント：<https://fastapi.tiangolo.com/ja/tutorial/query-params-str-validations/>

OpenAPIドキュメントに、対象クエリにどのようなものを入れるのか、説明を加えたいと思います。

```main.py
from fastapi import FastAPI, Query


app = FastAPI()


@app.get("/hello")
async def hello(name: str = Query(description="名前を入れてください")):
    return {"message": f"Hello {name}!"}
```

ドキュメントを見てみましょう。

![Queryを使った定義](OpenAPI-02.png "Queryを使った定義")


`description`を加えることでドキュメントに説明が記載されました。

## Pydanticを利用する場合

FastAPIでPydanticモデルを使用する時は、基本的にリクエストボディのデータを定義する場合になります。公式ドキュメントでは以下のように書かれています。

> リクエスト ボディを宣言するために Pydantic モデルを使用します。そして、その全てのパワーとメリットを利用します。
> <https://fastapi.tiangolo.com/ja/tutorial/body/>

クエリパラメータにPydanticモデルを適応する場合は、`Depends()`を利用します。こちらの方法はQiitaでも共有されています。

[FastAPIでクエリパラメータの指定にPydanticのModelを使う - Qiita](https://qiita.com/tamanugi/items/f3dd35ead7cf90eea24a)

今回の例では以下のようになります。引数`query`に`HelloIn`オブジェクトが代入されるので、受け取った文字列を利用する場合は`query.name`とします。

{{< alert  icon="circle-info" >}}
`HelloIn`内に複数の変数を定義することで、複数のクエリを受け取ることも可能です。
{{< /alert >}}

```main.py
from fastapi import Depends, FastAPI
from pydantic import BaseModel


app = FastAPI()


class HelloIn(BaseModel):
    name: str


@app.get("/hello")
async def hello(query: HelloIn = Depends()):
    return {"message": f"Hello {query.name}!"}

```

この場合のドキュメントは以下のようになります。

![Pydanticを用いた定義](OpenAPI-01.png "Pydanticを用いた定義")


続いてPydanticを使いつつ、ドキュメントに説明文を反映していきたいと思います。**感覚的には、`HelloIn`内の`name: str`を`name: str = Query(description="名前を入れてください")`にすれば良い気がしたのですが、これはうまく反映されませんでした。**

こちらに関して調べていたところ、FastAPIのリポジトリにイシューがあがっていました↓

[Query parameters from Depends do not show description in docs · Issue #4700 · tiangolo/fastapi](https://github.com/tiangolo/fastapi/issues/4700)

今回紹介する方法はこのイシュー内でコメントされている方法になりますが、「I think there should be a better solution for this issue（この問題に対するより良い解決策があるはずです）」と書かれているので、他にいい方法があるかもしれません。

**対処としては、以下のように`Field`や`Query`で2重にすれば良いです**。ただ、issueを見ていても「やってみたらうまくいきました！」という書き方なので、TIPS的な認識で利用しています。

```main.py
from fastapi import Depends, FastAPI, Query
from pydantic import BaseModel, Field


app = FastAPI()


class HelloIn(BaseModel):
    name: str = Field(Query(description="名前を入れてください"))
    # FieldでなくてQueryでもよい↓
    # name: str = Query(Query(description="名前を入れてください"))


@app.get("/hello")
async def hello(query: HelloIn = Depends()):
    return {"message": f"Hello {query.name}!"}
```

以下のように、ドキュメントに説明文が反映されています。

![Pydanticを利用してドキュメントにも反映](OpenAPI-02.png "Pydanticを利用してドキュメントにも反映")
