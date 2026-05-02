---
title: "認証機能付きJupyterHub（Native Authenticator）をDockerで立てつつ永続化"
date: 2023-12-20
description: "NativeAuthenticatorで認証機能をつけたJupyterHubをコンテナで立てつつ、各ユーザのサインイン情報と作成したファイルを永続化してみます。"
summary: "NativeAuthenticatorで認証機能をつけたJupyterHubをコンテナで立てつつ、各ユーザのサインイン情報と作成したファイルを永続化してみます。"
draft: false
categories:
 - テック系
tags:
 - Python
 - Docker
 - Jupyter
---


{{< alert icon="circle-info" cardColor="#f8f9fa">}}
今回Dockerコンテナを利用してJupyterHubサーバを立てていますが、JupyterHub上の各ユーザごとに専用コンテナを立ち上げることはしていません。
{{< /alert >}}

社内のPythonコミュニティやPythonを扱うハンズオン研修等で講師をすることがあるのですが、毎回環境構築から始めていると本題に入るまでに時間がかかることがあり課題に感じていました。環境構築も大事な要素ではあるのですが、主に以下の点からJupyterHubを使ってユーザ側に環境を提供しようと考えました。

- 参加者側のPCでは環境構築が難しい場合がある（セキュリティ的な制限）
- 環境構築を経験する必要がない状況がある
- ライブラリをこちらで導入して揃えておきたい

Colab（Colaboratory）を使えば良いのではというのもあるのですが、そちらはそちらで問題がありとりあえずJupyterHubでという形です。

実際はAWSのECS等を利用して立ち上げることを想定しているのでコンテナで環境を用意します。また、ユーザが作成したファイルなどの永続化と、認証情報の永続化を行いたいのでその部分にも触れていきます。

## Native Authenticatorを使った認証機能の付与

研修等で参加者に利用していただくことを想定しているので、認証機能をつけてユーザ毎に環境を分ける必要があります。そこで、JupyterHub公式が用意している[Native Authenticator](https://github.com/jupyterhub/nativeauthenticator) Plugin を利用します。

基本的な導入方法は[公式ドキュメント](https://native-authenticator.readthedocs.io/en/stable/quickstart.html#installation)に詳しく書かれていますので割愛します。作業は大きく分けて2つです。

- `pip install jupyterhub-nativeauthenticator`でプラグインの導入
- Jupyterhubのconfigファイルに色々書き込む

今回は前提となるJupyterHub環境の導入からプラグインの導入、サーバの立ち上げまですべてDockerfileで管理するのでそちらを紹介します。細かい挙動の説明等は以下のサイトが参考になります。

<div class="iframely-embed"><div class="iframely-responsive" style="padding-bottom: 52.5%; padding-top: 120px;"><a href="https://qiita.com/dyamaguc/items/db1da3084e36029f20cc" data-iframely-url="//iframely.net/OfWEWZc"></a></div></div><script async src="//iframely.net/embed.js"></script>

## 結論

今回の構成のポイントを改めて以下にまとめます。

- JupyterHubサーバを1つのコンテナで立てる
- Native Authenticatorを使ってシンプルなサインイン機能を追加する
- コンテナで立ち上げるのでデータの永続化を考える

### ファイル構成

データの永続化に関してはバインドマウントを利用します。AWSで展開する場合はEFSなどを利用することになると思います（そちらもおいおいまとめるかもです）。以下ファイル構成における「instance」フォルダに永続化するファイルが置かれていきます。

```text
root 
├── config
│   └── jupyterhub_config.py # JupyterHubのサービス設定ファイル
├── instance # 自動で生成されるディレクトリ（永続化データ）
│   ├── db   # サインイン情報などが保存されているSQLiteDBの保存先
│   └── home # コンテナ上のLinuxユーザのユーザディレクトリ保存先
├── docker-compose.yml
└── Dockerfile 
```

### jupyterhub_config.py

jupyterhubの設定ファイルは以下のとおりです。

```python
import os
from subprocess import check_call

import nativeauthenticator


# ユーザ定義
c = get_config()


def pre_spawn_hook(spawner):
    """ユーザ毎のサーバ立ち上げ時に行う処理"""
    username = spawner.user.name
    try:
        check_call(["useradd", "-ms", "/bin/bash", username])
    except Exception as e:
        print(f"{e}")
c.Spawner.pre_spawn_hook = pre_spawn_hook

c.JupyterHub.authenticator_class = "native" # 認証方法
c.JupyterHub.admin_access = True # 管理者アクセスの許可
c.JupyterHub.template_paths = [f"{os.path.dirname(nativeauthenticator.__file__)}/templates/"] 

c.Authenticator.admin_users = {"rakuichi"} # 管理者として扱うユーザ名
c.JupyterHub.db_url = "sqlite:///instance/db/jupyterhub.sqlite" # ユーザ情報などが保存されたDBの接続先

```

`pre_spawn_hook`はJupyterHub上のユーザがJupyter環境を立ち上げる際に実行される関数です。今回の例ではコンテナ上（Linux）のユーザを作成しています。このあたりの説明は公式ドキュメントや紹介した記事の中にあります。

データの永続化の観点から抑えておくべきポイントは、**JupyterHub上のユーザが自分の環境を立ち上げたときに表示されるカレントディレクトリが、Linux上のユーザディレクトリであるということ。また、JupyterHubユーザのサインイン情報はDBに保存されるということです**。このことから、ユーザディレクトリとDB（今回はSQLiteのファイル）ディレクトリを永続化すれば、コンテナ再起動しても問題ないと考えられます。

今回であれば、**ユーザディレクトリは `/home/<ユーザ名>` に作成され、DBファイルは `c.JupyterHub.db_url = "sqlite:///instance/db/jupyterhub.sqlite"` で指定しているので、これらをDocker側でバインドマウント**しておきます。

なお、Linuxユーザ＝JupyterHubユーザではないのかという疑問が出てきそうですが、紹介した記事で以下のように説明されています。

> NativeAuthenticatorだとユーザ名＆パスワード情報はOSのユーザ情報を見に行かないです。

### Dockerfileとdocker-compose.yml

Dockerfile構築にあたり大事なことは以下の点です

- JupyterHubを動かすための環境を揃える
- 認証のためのプラグイン導入
- データ永続化

なお、今回はマルチステージ等の最適化は行っていません。

まずはDockerfileからです。（今回バインドマウントの設定をするために docker-composeも利用しています）

```Dockerfile
FROM python:3.12-slim

ARG workdir="/jupyterhub"
WORKDIR ${workdir}

RUN apt-get update && pip install --upgrade pip

# nodejs(npm)等前提ライブラリのインストール
RUN apt-get install -y nodejs npm sudo \
    && npm install -g configurable-http-proxy

# jupyterhub用ライブラリの導入
RUN pip install jupyterhub jupyterlab notebook jupyterhub-nativeauthenticator

# jupyterhub設定ファイルのコピー    
COPY config  ${workdir}/config

# 起動
EXPOSE 8000
CMD ["sudo", "jupyterhub", "-f", "config/jupyterhub_config.py"]
```

あまり特別なことはしていません。シンプルに前提ツールを導入し、用意した設定ファイルを利用してJupyterHubサーバを立ち上げています。

続いて、docker-compose.ymlになります。こちらでバインドマウントするフォルダを指定しています。

```yaml
version: "3"
services:
  jupyterhub:
    build:
      context: ./
    ports:
      - "8000:8000"
    volumes:
      - type: bind
        source: ./instance/home
        target: /home
      - type: bind
        source: ./instance/db
        target: /jupyterhub/instance/db
    
```

初回起動時は以下コマンドで実行できます。

```shell
docker-compose up --build
```

## まとめ

簡単ではありますが、NativeAuthenticatorで認証機能をつけたJupyterHubをコンテナで立てつつ、各ユーザのサインイン情報と作成したファイルを永続化する方法を紹介しました。永続化に関して大事なのは

- ユーザディレクトリを永続化すること
- JupyterHub上のサインイン情報などが保存されたDBを永続化すること

になるので、このあたりを抑えておけばAWS上でも問題なく永続化できそうです。

かなり簡単な紹介になっているので、わからないこと等あれば、右下のチャットから質問ください。