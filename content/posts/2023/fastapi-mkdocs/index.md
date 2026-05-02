---
title: "FastAPI × MKDocs （ × Docker ）でAPIサーバとドキュメントページを同時に展開する"
date: 2023-11-01
description: "MKDocsでビルドした静的ファイルを、FastAPI の StaticFiles を使ってサーバ起動時に同時に展開する方法についてメモ書き。（Dockerのマルチステージを使ってmkdocsの自動ビルドも）"
summary: "MKDocsでビルドした静的ファイルを、FastAPI の StaticFiles を使ってサーバ起動時に同時に展開"
categories:
 - テック系
aliases:
 - /posts/fastapi-mkdocs/
tags:
 - Python
 - Pipenv
 - FastAPI
 - Docker
 - MKDocs
---


11月からアウトプット強化月間として継続的にブログ投稿していこうと思います。目標は週に2、3本で、基本的には技術系にしたいと思っています。途中で息切れした場合は関係ないネタも投稿するかもです。

強化月間1本目はFastAPI関連のネタです。以前投稿した「 [FastAPI×MKDocs（×GitHub Pages）でドキュメント生成](/posts/2023/poms-02)」で、FastAPIで生成したSwaggerドキュメントをMKDocs側に反映し、GitHubPagesで展開するという内容を紹介しました。

今回はAPIサーバを起動したときにAPIと同じポートにドキュメントページを展開するというものになります。ついでにDockerのマルチステージを使って、ソースのビルドも自動で行っています（この辺りについて詳細は触れません）。

正直GitHub Pagesで展開すればよいのですが、仕事の中でいろいろ事情があって今回の内容を試した経験があり、メモがてらに残していきます。

実際に作成されたページ例（同じlocalhost:8000に展開）↓

<div class="in_gallery">
{{< figure src="mkdocs.png" caption="ドキュメントページ" >}}
{{< figure src="fastapi.png" caption="FastAPIによるSwagger" >}}
</div>

## 基本構成

### ファイル構成

紹介するコードは以下リポジトリで見ることができます。

<a href="https://github.com/rakuichi4817/rakuapi/tree/blog-20231101"><img src="https://github-link-card.s3.ap-northeast-1.amazonaws.com/rakuichi4817/rakuapi.png" width="460px"></a>

ファイル構成は次の通りです。本記事で取り上げる内容にはコメントをつけています。

```text
rakuapi/
├── .devcontainer/
├── .vscode/
├── app/
│   └── main.py # FastAPIのコード
├── docs/
│   └── index.md
├── .gitignore
├── Dockerfile # ドキュメントページをビルドする
├── Pipfile
├── Pipfile.lock
├── README.md
└── mkdocs.yml # mkdocsの設定ファイル
```

### Dockerマルチステージの役割について

マルチステージビルドを利用して、極力本番環境に無駄なものを入れないようにしています。本番環境ビルダーや開発用については詳しく触れません。

mkdocs-builderステージでは、後述する**mkdocsのページビルド作業を自動で行い**、ビルドされた静的ファイルを本番環境ステージにコピーしています。

このようにすることで、

- 開発者自身がソースをビルド
- ビルド後のコードを管理

といった作業が必要がなくなります。

{{< mermaid >}}
flowchart TB
    base["ベースイメージ<br>（python:3.11-slim）"]
    devcontainer["開発用<br>（devcontainer用）"]
    builder["本番環境ビルダー<br>（Pythonライブラリのインストール）"]
    mkdocs-builder["ドキュメントビルダー<br>（MKDocsによるページビルド）"]
    app["本番環境"]

    base----> devcontainer
    base---> builder
    base----> mkdocs-builder
    base----> app
    builder-.Pythonライブラリソースのみコピー.-> app
    mkdocs-builder-.ビルドしたドキュメントソースのコピー.-> app
{{< /mermaid >}}

Dockerfileの内容をそのまま貼っておきます。もっとこうしたほうが良いよ！等のアドバイスや指摘はチャットからお願いします。

なお、Pythonのライブラリ管理にはPipenvを使っていますが、PipenvとDockerを組み合わせる際の話は過去記事の「[Pipenv、Docker（マルチステージビルド）、devcontainerで開発環境と本番環境を分ける](/posts/pipenv-docker/)」を参照ください。

```Dockerfile
# ----------ベースイメージの指定----------
FROM python:3.11-slim AS base

ARG workdir="/rakuapi"
WORKDIR $workdir

RUN apt-get update && pip install --upgrade pip 

# ----------開発用(.devcontainerが接続する用----------
FROM base AS devcontainer
# devcontainer上では仮想環境を作って開発する
RUN apt-get install -y git \
    && pip install pipenv

# ----------本番環境用ビルダー----------
FROM base as builder
RUN pip install pipenv 

# ライブラリをシステムへ直接書き込む
COPY Pipfile Pipfile.lock $workdir/
RUN pipenv sync --system 

# ----------ドキュメントビルダー----------
FROM base as mkdocs-builder
# mkdocs用ライブラリを入れる
RUN pip install mkdocs mkdocs-material mkdocs-render-swagger-plugin mkdocs-awesome-pages-plugin
WORKDIR /build
COPY ./docs /build/docs
COPY mkdocs.yml /build/
RUN mkdocs build

# ----------本番APP用----------
FROM base AS app

# ビルダーで展開したライブラリをアプリ用コンテナにコピー
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages/
COPY --from=builder /usr/local/bin/uvicorn /usr/local/bin/uvicorn
COPY --from=mkdocs-builder ./build/site /rakuapi/site
COPY ./app $workdir/app

EXPOSE 8000
HEALTHCHECK CMD curl --fail <http://localhost:8000/_stcore/health>
ENTRYPOINT ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 各ファイル詳細

簡単に各ファイルの内容を紹介しておきます。

### mkdocs.yml：MKDocs

まずMKDocsの設定ファイルであるmkdocs.ymlの説明をします。といっても、今回の内容を満たすことだけを考えるとあまり設定する必要はありません。GitHubのほうを見てもらうといろいろ書いていますが、本記事において必要な部分のみを抜粋します。

```mkdocs.yml
site_name: rakuapi
site_description: FastAPI×MKDocsのサンプル
site_author: Rakuichi

docs_dir: docs
```

ドキュメントページのもとになるソースファイルを配置するディレクトリを `docs_dir` で指定します。今回の場合「docs」になるため上記の設定となります。**mkdocsでビルドした静的ファイルはデフォルトで「site」に保存される**ため、明示的な指定はしていません（もちろん明示的に指定することもできます）。

{{< alert icon="info">}}
私が用意したDockerfileでは、この「site」ディレクトリのみを「mkdocs-builder」ステージから「app」ステージへコピーしていることがわかると思います。
{{< /alert >}}

### app/main.py：FastAPI

続いてFastAPI側のコードを紹介します。FastAPIで静的ファイルを展開する方法は公式ドキュメントに載っています。

公式ドキュメント：<https://fastapi.tiangolo.com/ja/tutorial/static-files/>

私のコードでは以下のように「site」ディレクトリが存在しているかどうかで、静的ファイルをマウントするかどうかを分けています。理由は開発環境で「site」ディレクトリが存在しないため、分岐がない場合エラーとなってしまうのを防ぐためです。

```main.py
import os

from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

site_dir = "site"
app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}


if os.path.exists(site_dir):
    # サイトページが存在していたらマウントする
    app.mount("/devdocs", StaticFiles(directory=site_dir, html=True), name="site")
```

これだけ用意すれば、あとは最初に用意したDockerコンテナを立ち上げると、ページ上部に乗せた画像のようにAPIと同じlocalhost:8000上でビルドしたページを確認することができます。

## まとめ

今回は、FastAPIの機能を使ってMKDocsでビルドしたページを展開する方法について紹介しました。なんだか書いてる割合の多くがDocker部分になった気がしますが...。ビルドを自動化する所はGitHub Actionsを使ってしまえばいいので、GitHub Pagesに展開する場合に応用することもできると思います。

FastAPIを使って静的ファイルを表示する方法自体はいろいろなものに応用できるので、NextJS等でビルドした後のソースとかに置き換えることも可能です。フロントエンドの勉強もしていきたい...。
