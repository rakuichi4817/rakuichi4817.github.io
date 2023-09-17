---
title: "Pipenv、Docker（マルチステージビルド）、devcontainerで開発環境と本番環境を分ける"
date: 2023-04-12T23:30:00
description: "PipenvでPythonのライブラリを管理しつつ、Dockerでマルチステージビルドを採用し、VSCodeのdevcontainerを利用して開発環境と本番環境を分ける方法についてメモ代わりにまとめます。サンプルアプリとしてStreamlitを採用しています"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/python.png
meta_image: images/cover/top-cover.png
categories:
 - 技術系備忘録
tags:
 - Python
 - Pipenv
 - Streamlit
 - Docker
 - 環境構築
series:
 - Python環境構築
 - アプリ開発
---

久しぶりの投稿です（n回目）。

最近、Pythonで環境を作るときにDockerで環境を作るようにしており、そこで利用している設定についてメモ代わりに書いておこうと思います。今まではPipenvの仮想環境だけで対応していたのですが、ホストマシンの影響を受けることが少なくなく、Dockerを使うようにしています。

## 実現したかった全体像

実装の参考にしたページは、本ページ下部の[参考](#参考)にまとめています。

本ページで紹介する構成で、自分が達成したかった内容は以下項目です。

- Pythonアプリ（サンプルはStreamlitで作る）の開発環境を作る
- PythonライブラリはPipenvで管理する
- Dockerのマルチステージビルドを利用して開発環境と本番環境を分ける
    - 極力イメージサイズを小さくしたい
    - 本番環境ではPipenvの仮想環境を作らない
    - 開発環境ではPipenvの仮想環境を作って開発する
- 開発環境にはVSCodeのDevContainerで接続する
    - 接続したコンテナ上でGitHubにsshアクセスしたい

Pythonのライブラリ管理にPoetryも検討しているのですが、まだ移行できずにいるのでPipenvを使っています。Pipenvは仮想環境を構築する際に仮想環境用のファイルを用意するので、既存環境とすみわけができて便利なのですが、本番環境では必要ありません。

Pipenvでは`--system`オプションをつけることで、システム側に直接書き込むことができるので、こちらをうまく活用して本番環境には最低限の要素のみ導入するようにします。

Gitに関しても開発環境のみ導入すれば問題ないので、Dockerのマルチステージビルド（Multi-stage builds）を利用して実現していきます。


## 実際のコード

一通り見たい方は以下のGitHubから確認していただければと思います。

<a href="https://github.com/rakuichi4817/docker-python-app-sample"><img src="https://github-link-card.s3.ap-northeast-1.amazonaws.com/rakuichi4817/docker-python-app-sample.png" width="460px"></a>

### Dockerfile

マルチステージビルドを使い、極力キャッシュを利用できるような構成にしたつもりですが、まだまだ無駄もあるかと思います。改善点があれば右下のチャットからぜひ教えていただければと思います。

構成の参考には[参考](#参考)に乗せている資料にとてもお世話になりました。

```dockerfile
# -----ベースイメージの指定-----
FROM python:3.10-slim AS base

ARG workdir="/workspace"
WORKDIR $workdir
RUN apt-get update && pip install --upgrade pip[label](https://speakerdeck.com/revcomm_inc/pythonnitotutenoyoriyoidockerfilewokao-eru)

# -----本番環境用ビルダー-----
FROM base as builder
RUN pip install pipenv 

# ライブラリをシステムへ直接書き込む
COPY Pipfile Pipfile.lock $workdir/
RUN pipenv sync --system
EXPOSE 8501

# -----本番APP用------
FROM base AS app

HEALTHCHECK CMD curl --fail <http://localhost:8501/_stcore/health>

# ビルダーで展開したライブラリをアプリ用コンテナにコピー
COPY --from=builder /usr/local/lib/python3.10/site-packages /usr/local/lib/python3.10/site-packages/
COPY --from=builder /usr/local/bin/streamlit /usr/local/bin/streamlit
COPY . $workdir

ENTRYPOINT streamlit run app/main.py --server.port=8501 --server.address=0.0.0.0

# -----開発用(.devcontainerが接続する用)-----
FROM base AS development
# devcontainer上では仮想環境を作って開発する
RUN apt-get install -y git \
    && pip install pipenv
```

### devcontainer.json

```devcontainer.json
{
    "name": "python-app",
    "build": {
        "context": "..",
        "dockerfile": "../Dockerfile",
        "target": "development"
    },
    "workspaceFolder": "/workspace",
    "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
    "customizations": {
        "vscode": {
            "extensions": [
                "njpwerner.autodocstring",
                "mhutchie.git-graph",
                "MS-CEINTL.vscode-language-pack-ja",
                "ms-python.python",
                "ms-python.vscode-pylance",
                "ionutvmi.path-autocomplete",
                "DavidAnson.vscode-markdownlint",
                "ms-toolsai.jupyter",
                "redhat.vscode-yaml",
                "bierner.markdown-preview-github-styles",
                "yzhang.markdown-all-in-one"
            ]
        }
    },
    "postCreateCommand": "pipenv install --dev"
}
```

`target`を指定することで、どのステージに対してアクセスするか指定できます。`extensions`で指定している拡張機能は個人的によく使うものを入れているだけなので、必要がなければ抜いてください。

`postCreateCommand`を用いて、VSCodeから開発環境へアクセスしたとき（最初にコンテナを作成した時）にPipenvの仮想環境を作成しています。開発段階では利用するライブラリを変えることもあるかと思うので、仮想環境を作成する形を残しています。

### 各環境へのアクセスの仕方

本番環境のビルドと実行は以下コマンドになります。`--target`オプションを利用することで、マルチステージビルドのどのステージをビルドするか指定できます。

```shell
# ビルド
$ docker build . --target app  -t python-sample-app
# アプリの起動
$ docker run -p 8501:8501 python-sample-app
```

開発環境へアクセスする際は、VSCodeのDevConteiner拡張で「Reopen In Container」を選択すればオッケーです。

## 参考

- Docker構成の参考（とても勉強になりました）
    <div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.1972%;"><iframe src="https://speakerdeck.com/player/996246a177ce4183b84a59b73c1293e6" style="top: 0; left: 0; width: 100%; height: 100%; position: absolute; border: 0;" allowfullscreen scrolling="no"></iframe></div>
- Streamlit×Dockerの参考
    <div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://laid-back-scientist.com/multi-stage-build" data-iframely-url="//iframely.net/4ailbrB?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>
- Pipenv×Dockerの参考
    <div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/kawasin73/items/3e58fc8a14af66ab7204" data-iframely-url="//iframely.net/aQOZ2uG?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>
- DevContainer上からSSHでGitHubにアクセスする
    <div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/SolKul/items/3103fdde94c09b044a3a" data-iframely-url="//iframely.net/IzPvLza?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>
