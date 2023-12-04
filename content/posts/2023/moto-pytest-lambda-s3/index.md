---
title: "motoとpytestでlambdaにおけるS3接続のモックを使ったテスト"
date: 2023-12-04
description: "motoとpytestを用いて、S3上のファイルを取得するLambdaプログラムのローカルテストを試す"
summary: "motoとpytestを用いて、S3上のファイルを取得するLambdaプログラムのローカルテストを試してみます"
draft: false
heroStyle:
categories:
 - テック系
tags:
 - Python
 - Pytest
 - AWS
 - Boto3
 - Moto
---

[Moto](https://docs.getmoto.org/en/latest/)とPytestを用いて、S3上のファイルを取得するLambdaプログラムのローカルテストを試してみます。

PythonのテストツールとしてPytestは割と使っており、Lambda用のコードをpytestを使ってテストしたいと考えていました。

↓関連記事
{{< article link="/posts/2022/github-actions-pytest/" >}}

単純にLambdaのコンソール上でテストすると、実際にS3にアクセスして処理することになります。そこで、環境に依存せずLambdaのコードが正しいかをテストしたいと思い、Motoにたどり着きました。なので、今回のテストは単体テストの範囲なのかなと思います。（テストの知識が浅いので間違っているかもです。）

## Motoについて

まずはMotoについて簡単にまとめます。[公式ドキュメント](https://docs.getmoto.org/en/latest/)には以下のように書かれています

> A library that allows you to easily mock out tests based on AWS infrastructure.
>
> （翻訳）AWSインフラストラクチャに基づいてテストを簡単にモックアウトできるライブラリ。

プログラム内でAWSサービスに接続する部分（基本的にboto3）を、テストのときに仮想の接続先へと切り替えるものになります。このあたりのモックテストで有名なツールとしては「[LocalStack](https://github.com/localstack/localstack)」があると思いますが、一旦はMotoを採用しています。このあたりは以下の記事が参考になります。

<div class="iframely-embed"><div class="iframely-responsive" style="padding-bottom: 52.5%; padding-top: 120px; width: 80%; margin:auto"><a href="https://zenn.dev/alivelimb/articles/20220508-aws-python-testing" data-iframely-url="//iframely.net/CKWH9bK"></a></div></div><script async src="//iframely.net/embed.js"></script>

## 構成

次節以降で紹介していく、テスト対象コードとテスト用コードは以下のような構成になります（最低限のみ記述）。

```text
root
├── src
│   └── lambda_function.py # テスト対象コード
└── tests
    ├── __init__.py 
    ├── conftest.py    # 事前・事後処理用
    └── test_lambda.py # テストコード
```

利用しているバージョン等は以下のとおりです。

- Python: 3.11
    - boto3: 1.33.6
    - moto: 4.2.10

Motoに関してはインストール時に以下コマンドを実行しました。「essential」でなくても良かったのですが、とりあえずという形です。

```shell
pip install moto[essential]
```

## テスト対象となるLambdaのコード

今回テスト対象とするLambdaのPythonコード「src/lambda_function.py」は以下の通りです。

```python
import boto3


def lambda_handler(event, context):
    """デモハンドラー"""
    # イベント情報からバケット名・オブジェクトキー・リージョンを取得
    bucket_name = event["Records"][0]["s3"]["bucket"]["name"]
    key = event["Records"][0]["s3"]["object"]["key"]
    region = event["Records"][0]["awsRegion"]

    s3_client = boto3.client("s3", region_name=region)

    try:
        # ファイルの中身を取得し返す
        response = s3_client.get_object(Bucket=bucket_name, Key=key)
        body = response.get("Body")
        return body.read()
    except Exception as e:
        print(
            f"Error getting object {key} from bucket {bucket_name}. Make sure "
            + "they exist and your bucket is in the same region as this function."
        )
        raise e
```

S3にファイルが追加・更新されたことをトリガーに、イベント情報から対象のファイルの中身を取得して返す単純な処理です。このプログラムのテストをローカルで行っていきます。

## Moto × Pytestのコード

### conftest.py

Pytestの前処理、後処理を実現する fixture を使って、モックの設定とテスト用のダミーデータを投入する処理を書いていきます。このあたりはMotoの[公式ドキュメント](https://docs.getmoto.org/en/latest/docs/getting_started.html?highlight=pytest#example-on-usage)を参考にしました。

以下のコードを「conftest.py」として作成します。


```python
import os

import boto3
import pytest
from moto import mock_s3

dummy_reagion = "us-east-1"
dummy_bucket_name = "test_bucket"
dummy_key = "sample.txt"


@pytest.fixture(scope="function")
def aws_credentials():
    """ダミー接続情報"""
    os.environ["AWS_ACCESS_KEY_ID"] = "testing"
    os.environ["AWS_SECRET_ACCESS_KEY"] = "testing"
    os.environ["AWS_SECURITY_TOKEN"] = "testing"
    os.environ["AWS_SESSION_TOKEN"] = "testing"
    os.environ["AWS_DEFAULT_REGION"] = dummy_reagion


@pytest.fixture(scope="function")
def s3(aws_credentials):
    """テスト用のモックS3接続"""
    with mock_s3():
        yield boto3.client("s3", region_name=dummy_reagion)


@pytest.fixture
def create_bucket(s3):
    """テスト用バケットの作成"""
    boto3.client("s3").create_bucket(Bucket=dummy_bucket_name)


@pytest.fixture
def put_dummy_textfile(create_bucket):
    """テスト用のファイルを設置してテスト後に削除する"""
    s3_client = boto3.client("s3", region_name=dummy_reagion)
    s3_client.put_object(Bucket=dummy_bucket_name, Key=dummy_key, Body="test")
    yield
    s3_client.delete_objects(
        Bucket=dummy_bucket_name, Delete={"Objects": [{"Key": dummy_key}]}
    )
```

モックの書き方自体は色々あるのですが、Pytestのfixtureと組み合わせる書き方で挑戦した形です。

今回テストするコードは、S3のファイルを取得し読み込んだ結果を表示するものなので、前処理として「test」と書かれたファイルを設置しておきます。これが fixtureの`put_dummy_textfile()`の部分です。このfixtureの処理の流れとしては、

1. `s3` : 接続先のモック
2. `create_bucket`: モック先に仮想のバケット作成
3. `put_dummy_textfile`: 仮想バケットに仮想のテキストファイル設置

となります。

### test_lambda.py

つづいて、テストコードを見ていきます。本来の想定しているLambdaの処理としては、S3からイベント情報を受け取って処理をすることになっているので、ダミーのイベント情報として `dummy_event` を用意します。なお、今回こちらのダミー情報に関しては、プログラム内で操作する項目のみに絞っております。

```python 
import pytest

from src.lambda_function import lambda_handler

from .conftest import dummy_bucket_name, dummy_key, dummy_reagion

dummy_event = {
    "Records": [
        {
            "awsRegion": dummy_reagion,
            "s3": {
                "bucket": {
                    "name": dummy_bucket_name,
                },
                "object": {
                    "key": dummy_key,
                },
            },
        }
    ]
}


@pytest.mark.parametrize(
    "event, expected",
    [(dummy_event, "test")],
)
def test_lambda_handler(event, expected, put_dummy_textfile):
    # モックされた仮想S3よりテキストを取得して期待データと比較
    assert expected == lambda_handler(event, None).decode("utf-8")
```

fixtureである `put_dummy_textfile` によりS3の接続先がモックへと切り替わります。これはfixtureの`put_dummy_textfile` の中身を見たときに、段階的に読み込んでいるfixtureの `s3` で `with mock_s3()` と書いてある部分が効いています。

これにより関数 `lambda_hundler`の中でboto3経由でS3に接続しようとしている部分がモックされるので、S3への接続を行わずMoto側で仮想的に立てたS3へと繋がります。よって、fixtureで仮想的に事前投入したテキストファイル情報が取得される形になり、テストが合格します。

## まとめ

今回はMotoとPytestを使って、S3接続を処理に含むLambdaのPythonコードをローカルでテストする方法を紹介しました。全体的にかなり雑な説明ではあるので、もしかしたら詳細を追記していくかもしれません。

また、他のサービスのモック方法や結合テストのところの検討、LocalStackの調査等々、気が向いたときに取り組んでいきたいと思います。

## その他参考

- [pytest と moto で優勝する](https://blog.serverworks.co.jp/moto-pytest-dynamodb)