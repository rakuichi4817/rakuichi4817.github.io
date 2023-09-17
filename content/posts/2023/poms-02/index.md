---
title: "FastAPI×MKDocs（×GitHub Pages）でドキュメント生成"
date: 2023-02-21T00:45:00+09:00
description: "MKDocsで作成するドキュメントに、FastAPIで生成されるSwaggerを表示します"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/python.png
meta_image: "posts/poms-02/poms-documents.png"
categories:
 - 技術系備忘録
tags:
 - Python
 - FastAPI
 - MKDocs
 - GitHub
series:
 - アプリ開発
---

以前投稿した記事「[FastAPI×Streamlitでアプリ開発（Getリクエスト）](/posts/poms-01)」で、FastAPIとStreamlitを利用し、Pythonのみでバックエンドとフロントエンドを分離したアプリを作成しました。

どうせ練習するなら、合わせてドキュメントも作ってしまおうと思い、PythonでMarkdownを静的サイトへ変換できる静的サイトジェネレータ [**MKDocs**](https://www.mkdocs.org/) を利用してみることにしました。

リポジトリは以下になります。

<a href="https://github.com/rakuichi4817/poms"><img src="https://github-link-card.s3.ap-northeast-1.amazonaws.com/rakuichi4817/poms.png" width="460px"></a>

なお、本記事の方法を使って実際に作成したドキュメントが以下になります。（[リンク](<https://rakuichi4817.github.io/poms/backend-swagger/>)）

{{< img src="/posts/poms-02/poms-documents.png" title="実際のページ" caption="MKDocsで作成され、Swaggerドキュメントも閲覧可能" width="80%" position="center">}}


## ライブラリのインストール

今回利用するライブラリは以下の4つになります。

- [fastapi](https://fastapi.tiangolo.com/ja/)：API作成フレームワーク
- [mkdocs](https://www.mkdocs.org/)：MKDocsのメインライブラリ
- [mkdocs-material](https://squidfunk.github.io/mkdocs-material/)：生成するサイトのテーマを追加するライブラリ
- [mkdocs-render-swagger-plugin](https://github.com/bharel/mkdocs-render-swagger-plugin)：APIドキュメントをMKDocs上で閲覧するためのライブラリ

私が作成している環境では`pipenv` を使ってライブラリ管理していますので、リポジトリでは「Pipfile」にライブラリ情報がまとまってます。

## FastAPIでSwaggerドキュメントのjsonを出力

FastAPIで作成したAPIサーバは、起動したときにSwaggerドキュメントを閲覧できます。今回はこちらをサーバを起動せずとも閲覧できるよう、サーバを起動したときにAPI情報として「openapi.json」を出力しておきます。そしてその「openapi.json」をMKDocsで閲覧していきます。

参考にしたのはFastAPIのGitHub issueでのやり取りです。（[issueへ](https://github.com/tiangolo/fastapi/issues/2712)）

最小構成でコードを書くなら以下のようになります。各処理の内容はコメントで書いていますが、サーバ起動時にのみ動く関数を作成します。

```python
import json

from fastapi import FastAPI


app = FastAPI()


# サーバ起動時に発火する処理
@app.on_event("startup")
def save_openapi_json():
    # openapi定義情報の取得
    openapi_data = app.openapi()
    # ファイル出力
    with open("openapi.json", "w") as file:
        json.dump(openapi_data, file)
```

これで一度APIサーバを起動すると、自動的に「openapi.json」が出力されるようになります。

## MKDocsでSwaggerドキュメントを閲覧する

### mkdocs.ymlにプラグインを定義

MKDocsでは「mkdocs.yml」に設定内容を書き込みます。私が上記リポジトリで実際に作成した内容が以下になります。

```yml
site_name: POMS
site_author: Rakuichi
copyright: Rakuichi

site_dir: docs
docs_dir: docsrc
theme:
  name: material
  locale: ja
  features:
    - toc.integrate
  palette:
    primary: brown
plugins:
  - search
  - render_swagger
```

それぞれの設定について簡単に説明しておきます。

- site_name, site_author, copyright：サイトの基本情報
- site_dir：ビルドしたHTMLソースを入れるディレクトリ
- docs_dir：ビルド元になるMarkdownファイル等が入ったディレクトリ
- theme：デザインテーマ（詳細は「mkdocs-material」の公式ドキュメント参照）
  - name：利用するテーマ名
  - locale：言語
  - features：利用するテーマに存在する設定
  - palette：カラーテーマ
- plugins：利用するプラグイン
  - `search`：ドキュメント内検索機能
  - `render_swagger`：SwaggerUIの表示機能


プラグインの設定をするところの `render_swagger` によって、MKDocsで生成するサイト内でSwaggerドキュメントを表示できるようにします。

### 実際にMarkdown内に埋め込む

MKDocsではMarkdownがサイトのもとになります。今回「backend-swagger.md」というファイルを作成し、同階層に「openapi.json」も置いておきます。その状態で「backend-swagger.md」内に以下内容を記述します。

`!!swagger <対象ファイル>!!` とすることで、Swaggerドキュメントを表示することができます。

```markdown
# APIドキュメント

!!swagger openapi.json!!
```

これでMKDocsでもSwaggerドキュメントが表示されるようになります。試しに閲覧する場合は以下コマンドでWebドキュメントを起動します。

```shell
mkdocs serve
```

実際に表示されるのが、この記事の冒頭の画像のような形です。

## GitHub Pagesで公開する

簡単な説明にはなりますが、以上でページの作成はできました。あとはGitHub Pagesを利用して、作成したウェブドキュメントを`mkdocs serve`せずとも見れるようにしてみます。

「mkdocs.yml」にてビルド先を指定していますので、以下コマンドを実行すると「docs」ディレクトリにHTMLやcssが出力されます。

```shell
mkdocs build
```

あとはpushし、GitHubのリポジトリ上でGitHub Pagesの公開設定を行うだけです。GitHub Pagesでは「docs」ディレクトリを対象として選択可能なので簡単に設定できるかと思います。

参考：[GitHub Pagesの公式ドキュメント](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages)

## まとめ

かなり簡単な説明にはなりますが、Pythonだけでバックエンドも、フロントエンドも、ドキュメントも作ってしまうということができました。しかもGitHubですべて管理できるというのも気に入っています。

おそらくこれがベストプラクティスというわけではないと思います。しかし、個人的にいろいろできることがありそうだなと感じており、引き続き関連要素の学習を続けていくつもりです。