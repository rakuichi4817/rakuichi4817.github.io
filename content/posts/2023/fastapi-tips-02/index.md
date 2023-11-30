---
title: "FastAPIのファイル構成"
date: 2023-11-06
description: "ぼくがかんがえたさいきょうのFastAPIのファイル構成（最強ではないです）"
summary: "ぼくがかんがえたさいきょうのFastAPIのファイル構成（最強ではないです）"
draft: false
categories:
 - テック系
tags:
 - Python
 - FastAPI
aliases:
 - /posts/fastapi-tips-02/

---

アウトプット強化月間3本目はFastAPIネタです。FastAPIはずっと業務で触っていることもあり、書ける内容が多いのでできる限りいろいろ書いていこうと思います。

今回はファイル構成についてです。

FastAPIでそれなりの量のコードを書いていると、どうしてもファイル量が増えていきます。その時にどのような構成で配置すればよいのか簡単に書いていきます。

## 参考にしたサイト

いきなりですが、私が参考にしたサイトを載せておきます。

- [Bigger Applications - Multiple Files - FastAPI (tiangolo.com)](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [zhanymkanov/fastapi-best-practices (github.com)](https://github.com/zhanymkanov/fastapi-best-practices)
- [A Comprehensive Guide to Structuring a FastAPI Project for Reproducibility and Maintainability | by Durgesh Samariya | Python in Plain English](https://python.plainenglish.io/a-comprehensive-guide-to-structuring-a-fastapi-project-for-reproducibility-and-maintainability-1705c41dac41)
- [takashi-yoneya/fastapi-mybest-template (github.com)](https://github.com/takashi-yoneya/fastapi-mybest-template)

## ファイル構成

ファイル構成と一部実際のコードもお見せしようと思います。

### 全体構成

全体構成は以下のようになります。appディレクトリ部分のみを書いていますので、Dockerfileやdocs・testsディレクトリ等はありません。




※以下で「エンドポイント」と表現しているものに関しては、APIリクエスト時の処理のことだと考えてください。

```text
app 
├── api                 # エンドポイント関連コード
│   ├── common/         # エンドポイント上でしか使わない関数をまとめたモジュール群
│   ├── v1/             # エンドポイント部分のモジュール群
│   ├── __init__.py
│   ├── api.py          # v〇ディレクトリから各 `APIRouter` をまとめる 
│   ├── auth.py         # 認証関係のエンドポイント
│   └── dependencies.py # `Depends`関数の集約（量が多ければディレクトリにする）
├── aws                 # AWS（boto3）関連の基本操作処理
├── core                # 設定項目やプログラム全体で使う可能性のある処理
├── crud                # DBのCRUD処理
├── schemas             # pydanticモデル定義
├── __init__.py
└── main.py             # 実行ファイル
```

**インポートがなるべくディレクトリ単位で一方方向になることを意識**しています。例えば、coreディレクトリがその他の場所にあるモジュールからインポートされることはあっても、coreディレクトリ内部のモジュールが他のディレクトリ上のモジュールをインポートすることはないようにしています（例外はあってもよいと思います）。

なお、「aws」ディレクトリは他のクラウドであれば変わると思いますし、例えば外部APIと連携するのであればそのAPI名なるかと思います。また、数が増えるのであれば、それらもまとめた「external」のようなディレクトリを作成してもよいかもしれません。

### apiディレクトリ

補足したほうが良いと思うところだけ書いていきます。

#### 基本構成

メインとなるapiディレクトリですが、バージョンごとに「v1」のようなディレクトリを作成するようにしています。APIであれば複数のバージョンが一時的に存在することは考えられますので。

例えば、「/users」と「/items」みたいなURIのエンドポイントがあるのであれば、「users.py」と「items.py」を「v1」ディレクトリに設置します。単純なコード例も書いておきます。

```python
#app/api/v1/users.py
from fastapi import APIRouter


router = APIRouter()

@router.get("")
"""以下略"""
```

そして、「api.py」で以下のようなコードを書きます。

```python:app/api/api.py
from fastapi import APIRouter

from app.api.v1 import users, items


v1_routers = APIRouter()
v1_routers.include_router(users.router, prefix="/users", tags=["ユーザ"])
```

そして `v1_routers` を「main.py」で`FastAPI()` インスタンスに `include_router()`すればオッケーです。

バージョンがどんどん増えていくなら、各バージョンのディレクトリにある「_\_init\_\_.py」上でまとめてもいいかもしれません。

#### commonディレクトリ：エンドポイント用関数

エンドポイント用の処理を書いていると、どうしても処理が肥大化して関数化することになります。この時、この関数をどこに置くか迷ったのですが、私はcommonディレクトリを作成しそこに集約することにしました。

最初は、今回の例でいうと「v1/users.py」上にそのまま書いていました。しかし、「v1/items.py」でその関数を使うことになったり、バージョンをまたいで使うことも可能性として考えられます。さらに、1つのファイルのコードが長くなり、エンドポイント部分のコードなのか、細分化した関数なのかわかりづらい問題がありました。

この辺りを考慮した結果「common」ディレクトリを作成してそこに集約することとしました。今のところこの形でうまくいっていると感じています。

#### auth.py：認証用エンドポイント処理

認証用トークンを発行するエンドポイントは独立させています。バージョンをまたいで共通のものを使う場合はこのままでよいと思います。

### coreディレクトリ：設定やシステム共通関数

こちらにはシステムの設定値等を指定するファイルをおいています。また、時間の取得やログ関連のようなコード全体で使用する可能性のある処理をまとめています。

### その他

その他のディレクトリなどの説明は、[全体構成](#全体構成)のところを見ていただければわかるかと思いますので割愛します。

## まとめ

今回は簡単にですがFastAPIのファイル構成を紹介しました。あまり細かい部分の説明は書いていませんので、また気が向けばまとめてみようかなと思います。