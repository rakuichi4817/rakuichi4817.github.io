---
title: "FastAPI×Streamlitでアプリ開発（Getリクエスト）"
date: 2022-10-07T18:00:00+09:00
description: "FastAPIとStreamlitを用いて、Pythonだけでバックエンドとフロントエンドの両方を作ります。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/python.png
meta_image: "posts/poms-01/app-01.png"
categories:
 - 技術系備忘録
tags:
 - Python
 - FastAPI
 - Pydantic
 - Streamlit
series:
 - アプリ開発
---

PythonでAPIを作成するときに使われるFastAPIですが、やはりアプリにしたいというところで、フロントエンドの部分を作るのに困っていました。手っ取り早く画面を作るには、そこまでフロントエンドの技術を持っていないので...。そんな時、[Streamlit](https://streamlit.io/)というPythonで簡単にアプリが作れるフレームワークがあることを知りました。

Streamlitは、リッチなアプリが作れる便利なフレームワークです。このフレームワークは、データを扱うアプリの作成をターゲットとしています。そのため、データ分析や機械学習系のツールを作る際に、手っ取り早くアプリ化するには、とても適していると思います。

ただ今回は、こちらの**Streamlitをあくまでフロントエンド用に利用し、FastAPIで作成したAPIサーバとやり取りさせる**ことで、**Pythonだけでバックエンドとフロントエンドを分離したアプリを作成**していきます。

## 実際に作成してしているアプリの紹介

{{< alert theme="warning" dir="ltr" >}}
こちらの内容に関しては記事執筆時点（2022年10月7日）のものなので、リポジトリの内容とずれがあるかもしれません。
{{< /alert >}}

↓リポジトリ

<a href="https://github.com/rakuichi4817/poms"><img src="https://gh-card.dev/repos/rakuichi4817/poms.svg?fullname="></a>

練習で作成しているアプリの画面です。

{{< img src="/posts/poms-01/app-01.png" title="足し算機能" caption="単純なGetリクエスト" width="80%" position="center">}}
{{< img src="/posts/poms-01/app-02.png" title="モザイク処理" caption="画像データのやり取り" width="80%" position="center">}}

**本記事では足し算機能についてのみ紹介いたします。** 今後も様々な機能を、練習として実装していこうと思っています（機能毎の記事も書くかもしれません）。

作成中のアプリでは以下のようなプロジェクト構成をしています。マイクロサービスというならPipfileも分けるべきなのですが、開発の簡略化のためにまとめています。

```text
poms
├─.vscode             VSCode開発用設定
│      settings.json
├─backend             バックエンド（FastAPI）ソース
│  │  main.py         アプリ立ち上げ用
│  │  __init__.py
│  ├─config           バックエンドの設定関連
│  ├─libs             バックエンドのロジック部分
│  ├─schemas          Pydanticモデル定義
│  └─v1               API作成部分
│      │  api.py
│      │  __init__.py
│      └─routers      エンドポイント定義
├─docs                全般のドキュメント
└─frontend            フロントエンド（Streamlit）ソース
│   │  home.py        立ち上げ用（ホーム画面）
│   │  __init__.py 
│   ├─config          フロントの設定関連
│   └─pages           フロントの各ページ
├─Pipfile
├─Pipfile.lock
└─README.md
```

なお、以下の説明では、上記のプロジェクト構成は利用せずにシンプルな形で説明しています。

## 足し算機能を作成してみる

非常に単純な足し算機能を作成してみます。役割分担としては以下のようになります。

- Streamlit（フロントエンド）
  - aとb、2つのクエリパラメータ（数値）の入力
  - バックエンドへの足し算処理リクエスト
  - レスポンスを受け取って表示
- FastAPI（バックエンド）
  - 足し算エンドポイントの作成

記事内ではソースを簡略化していますので、実際の実装を見たい方はリポジトリをご参照ください。

{{< alert theme="warning" dir="ltr" >}}
繰り返しになりますが、以下のコードは上記のプロジェクト構成に依存していません。記事内ではコードを簡略化していますので、実際の実装を見たい方はリポジトリをご参照ください。
{{< /alert >}}

### フロントエンド部分

APIへのリクエストは`requests`ライブラリを利用します。ボタンを用意して、そのボタンがクリックされたら`if submitted`内の処理が走り、リクエストが実行されます。

```home.py
import requests
import streamlit as st

with st.form("get_sample_form"):
    # リクエスト先
    endpoint_url = "http://127.0.0.1:8000/plus"
    # 表示と入力
    st.write("## 足し算（a + b）")
    st.write("Getリクエスト（クエリ付き）")
    a = st.number_input("a を入れてください")
    b = st.number_input("b を入れてください")

    # 計算実行ボタン
    submitted = st.form_submit_button("計算する")
    if submitted:
        # ボタン押下時の処理
        response = requests.get(endpoint_url, params={"a": a, "b": b})
        if response.status_code == 200:
            result = response.json()["result"]
            st.success(f"**{result}**")
        else:
            st.error(f"{response.status_code}エラーが発生しました。詳細は以下を参照ください")
            st.json(response.json())
```

Streamlitは[公式ドキュメント](https://docs.streamlit.io/)が充実しているので、ボタンや入力欄の作成はそちらを見てください。処理の流れとしては、受け取った入力を`requests`でGetリクエストするだけです。`st.number_input()`を利用しているので、数値以外は入力できません。また、デフォルトで0がはいっているため、中身がないままリクエストされることもありません。

### バックエンド部分

今回、Python3.10を利用しているので、型ヒントの書き方がPython3.9までとは違う点に注意が必要です。Python3.9までの方は`int | float`となっている部分を、`Union[int, float]`としてください（`from typing import Union`）。

```main.py
from fastapi import FastAPI, Query
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field


class AddIn(BaseModel):
    """GET /plus の入力"""

    a: int | float = Field(Query(description="足される数"))
    b: int | float = Field(Query(description="足される数"))


class AddOut(BaseModel):
    """GET /plus の出力"""

    result: int | float = Field(description="足し算の結果")


app = FastAPI()

# CORS設定（公式ドキュメント：https://fastapi.tiangolo.com/ja/tutorial/cors/）
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost", "http://localhost:8501"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/plus", response_model=samples.AddOut)
async def add(query: samples.AddIn = Depends()):
    """足し算

    Parameters
    ----------
    a : int | float
        足される数\\
    b : int | float
        足す数
    """
    return {"result": query.a + query.b}
```

CORSの設定をしておかないと、8501ポートで開いているStreamlitからのリクエストを遮断する可能性があります。この辺りは公式ドキュメントをご覧ください

公式ドキュメント：[CORS (オリジン間リソース共有) - FastAPI](https://fastapi.tiangolo.com/ja/tutorial/cors/)

### 実行画面

フロントエンドとバックエンドをそれぞれ立ちあげて、アプリ画面からリクエストを実行してみます。

{{< img src="/posts/poms-01/app-plus.png" title="足し算機能" caption="-20+40=20" width="80%" position="center">}}

計算結果を表示できました。

## まとめ

StreamlitとFastAPIを使うことで、Pythonのみでも簡単にフロントエンドとバックエンドを分離したアプリを作成することができました。フロントにややこしい処理を実装する必要がないため、プログラムの管理もしやすかったです。また、React等を学んで、フロントのみ切り替えることも簡単なので、まずアプリを作るという点では非常に使いやすいですね。

今後は、上で紹介したモザイク機能に関する記事や、その他の機能を実装していこうかなと思っています。