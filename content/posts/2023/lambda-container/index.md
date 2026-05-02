---
title: "Python（slim）コンテナをLambdaで動かすためのDockerFile"
date: 2023-12-19
description: "Pipenvでライブラリを管理しつつPythonのslim系イメージをベースにして、Lambda関数をコンテナ形式で作成します"
summary: "Pipenvでライブラリを管理しつつPythonのslim系イメージをベースにして、Lambda関数をコンテナ形式で作成します"
draft: false
categories:
 - テック系
tags:
 - Python
 - Pipenv
 - Docker
 - AWS
---

今更ですがコンテナイメージを使ってLambda関数を動かしてみます。普通に動かすだけであれば[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images-create.html)に従えば良いので、あえて記事にするほどのことでもないです。

しかし、今回は次の2点の要素があるため備忘録として残しておくことにしました。

- AWS公式が提供している公式イメージを使わずに「python3.◯◯-slim」系イメージをベースとする
- pipenvを使ってPythonライブラリを管理し、可能な限りマルチステージを活用する

## この記事で触れること・参考記事

なお、**本記事ではDockerFile部分のみ取り上げ、ECRにイメージをプッシュする部分やLambda関数を作成するところには触れません**。そのあたりは以下の記事を参考にすると良いと思います。私自身もこちらの記事を参考にしました。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://qiita.com/kyamamoto9120/items/f1cda89ffc7cb5254f17" data-iframely-url="//iframely.net/HjHw3qh?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

結論は[こちら](#結論検証作成した構成)。

## 公式ドキュメントを見る

公式ドキュメント内にもSlim系のイメージを使った場合のDockerfileの例を紹介してくれていますので、まずはこちらを見てみます。

```dockerfile
# Define custom function directory
ARG FUNCTION_DIR="/function"

FROM python:3.11 as build-image

# Include global arg in this stage of the build
ARG FUNCTION_DIR

# Copy function code
RUN mkdir -p ${FUNCTION_DIR}
COPY . ${FUNCTION_DIR}

# Install the function's dependencies
RUN pip install \
    --target ${FUNCTION_DIR} \
        awslambdaric

# Use a slim version of the base Python image to reduce the final image size
FROM python:3.11-slim

# Include global arg in this stage of the build
ARG FUNCTION_DIR
# Set working directory to function root directory
WORKDIR ${FUNCTION_DIR}

# Copy in the built dependencies
COPY --from=build-image ${FUNCTION_DIR} ${FUNCTION_DIR}

# Set runtime interface client as default command for the container runtime
ENTRYPOINT [ "/usr/local/bin/python", "-m", "awslambdaric" ]
# Pass the name of the function handler as an argument to the runtime
CMD [ "lambda_function.handler" ]
```

こちらを見た私が思うポイントは以下の点です。

- ビルドステージと本番ステージを分けている
- lambdaで動かすためのライブラリ「**awslambdaric**」を入れている
- ライブラリの導入先をLambda関数ハンドラーを含むモジュールと同じところにする
- `ENTRYPOINT` をDockerコンテナの起動時に実行させるランタイムインターフェイスクライアントを指定する
- `CMD` にLambdaで動かしたいメインモジュール（関数ハンドラー）を設定する

これらのポイントはLambdaで動かすための設定と解釈しても問題ないのかなと思います。AWSが提供しているLambda用のイメージはこのあたりをあらかじめ組み込んでいるのだと思います。逆に、提供されているイメージを使わない場合は、このあたりの設定をしないといけないということですね。

## 結論（検証作成した構成）

上記の内容を踏まえてPipenvでPythonライブラリを管理しつつ、Lambdaで動作可能な極力軽量なイメージを作成してみました。（最適ではないところもあるかもしれませんが、アドバイスがあれば右下のチャットから教えてください）

### ファイル構成

まずは今回作成したファイル構成を紹介します。示しているのは最低限のファイル構成になります。appフォルダの中にある「main.py」が実際にLambdaで動かすモジュール（関数ハンドラー）になります。

```text
root 
├── app
│   └── main.py # Lambdaで動かすメインモジュール
├── Dockerfile 
├── Pipfile
└── Pipfile.lock
```

「Pipfile」、「Pipfile.lock」の中身に関しては重要ではないので割愛しておきます。

### 関数ハンドラー

「main.py」の中身は以下のようにシンプルにしています。条件に関わらず起動したら文字列を返します。（ヴィッセル優勝嬉しいなぁ🐮🔶）

```python
def handler(event, context):
    return "ヴィッセル神戸が優勝だ！"
```

### DockerFile

こちらがメインのDockerFileです。「[公式ドキュメントを見る](#公式ドキュメントを見る)」で触れたポイントを踏まえつつ、Pipenvで管理したライブラリを導入するようにしています。



```dockerfile
#==================================
# ベースとなるステージ
#==================================
FROM python:3.11-slim AS base

ARG workdir="/lambda-docker"

RUN apt-get update && pip install --upgrade pip 
#==================================
# ビルダーステージ
#==================================
FROM base AS builder

RUN pip install pipenv 
# ライブラリ情報をコピーし、そちらを利用して仮想環境を作らずシステム側へのインストール
COPY ./Pipfile ./Pipfile.lock /
RUN pipenv sync --system

# lambda用のライブラリインストール
RUN mkdir -p ${workdir}
RUN pip install \
    --target ${workdir} \
    awslambdaric
#==================================
# Lambdaで動かすイメージ
#==================================
FROM base AS lambda
WORKDIR $workdir

# ビルダーからのソース受け取り
# pipenvで入れたものはこちらから入れる
COPY --from=builder /usr/local/lib/python3.11/site-packages ${workdir} 
COPY --from=builder ${workdir} ${workdir}

# 作成したソースのコピー
COPY ./app ${workdir}

ENTRYPOINT [ "/usr/local/bin/python", "-m", "awslambdaric" ]
CMD ["main.handler"]  
```

Pipenvを利用していますが仮想環境は作らず、ビルダーステージで直接システム側にインストールします。そして本番ステージへコピーする形です。

このあたりの内容は過去に取り上げたので気になる方はぜひ↓

{{< article link="/posts/2023/pipenv-docker/" >}}

## まとめ

これらの設定でビルドしたイメージをECRに登録し、Lambda関数を作成、テストすると問題なく起動することを確認できました。冒頭に記述した通り、このあたりの設定は別の記事を参考にしてください。

それにしてもコンテナの起動が早すぎてびっくりしました。処理時間の制限などはありますが非常に便利だなぁと感じました。