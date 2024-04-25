---
title: APSchedulerを用いてFastAPIでジョブの定期実行
date: 2024-04-25
description: "APSchedulerを用いてFastAPIでジョブを定期実行してみました"
summary: "APSchedulerを用いてFastAPIでジョブを定期実行してみました"
draft: false
heroStyle: "simple"
categories:
 - テック系
tags:
 - Python
 - FastAPI
 - APScheduler
---

ひさしぶりの投稿です。2024年に入ってから転職したり引っ越したりと、公私ともにバタバタしていて投稿が疎かになっていました（毎度おなじみの言い訳から入るやつ）。

5月から新たにお世話になるチームでは、各メンバーが積極的にブログを書いていました。それを見てやる気が出たので、私も投稿頑張っていきます。

久しぶりの今回は、FastAPIネタです。タイトルの通り、FastAPIで立てているサーバ上で、定期実行するようなジョブを仕込むというネタです。利用したライブラリは **「[APScheduler](https://github.com/agronholm/apscheduler)」** です。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://dev.classmethod.jp/articles/apscheduler-fastapi/" data-iframely-url="//iframely.net/vZfWwih?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

基本的には上記リンクを参考にしましたが、FastAPIのアップデートに伴いスタートアップイベント周りの推奨実装方法が変わっていました。本記事では現在推奨されている方法を適用しています。

## 試しに動かしたコードのリンク

実際に動かしたコードはGithubにおいているのでそちらのリンク。

GitHubリポジトリリンク：<https://github.com/rakuichi4817/rakuapi/tree/feature/scheduler>

## Pythonライブラリ等

今回の記事で利用しているライブラリやらのバージョンを記載しておきます。

- Python: 3.11
    - fastapi: 0.110.1
    - apscheduler: 3.10.4

## 実装の紹介

### FastAPIのLifespan Eventsの使い方

基本的な流れとしてはAPIサーバ立ち上げ時に、APSchedulerでスケジューラを起動してジョブを定期実行する形です。ですので、まずはFastAPI側で立ち上げ時に実行する処理を仕込まないといけません。

参考にした記事では、`@app.on_event("startup")` を使って立ち上げ時に実行する関数を定義していましたが、**FastAPIの公式ドキュメントでは[Lifespan Events](https://fastapi.tiangolo.com/advanced/events/)を使う形を推奨**していました。ということで今回はこちらを使っていきます。

以下に簡単なコード例を示します。

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    print("startup")
    yield
    print("shutdown")

app = FastAPI(lifespan=lifespan)
```

細かい説明は[公式ドキュメント](https://fastapi.tiangolo.com/advanced/events/)を見ていただいたほうが良いと思いますので要点だけ。

`lifespan` 関数内で、`yield` があると思いますが、その前後の処理がそのままスタートアップ処理とシャットダウン時の処理となります。pytestのfixture等で同じような使い方をするので、経験者はイメージしやすいかと思います。

定義した関数を `FastAPI` クラスのインスタンス化時に引数として渡すだけで実装完了です。

### APSchedulerをLifespanに組み込む

今回は非同期関数をジョブとして指定するので、[`AsyncIOScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/asyncio.html)を使用していきます。

簡単に使うだけであればシンプルなコードなので、早速上記の`lifespan`と組み合わせてみます。なお、関数 `tick()` などは以下stackoverflowの内容を参考にしています。

<https://stackoverflow.com/questions/68217756/apscheduler-to-call-an-async-function>

```python
import asyncio
from contextlib import asynccontextmanager
from datetime import datetime

from apscheduler.schedulers.asyncio import AsyncIOScheduler
from fastapi import FastAPI


async def tick():
    """サンプル関数

    Notes
    -----
    現在時刻の表示と3秒のスリープ
    """
    print(f"Tick! The async time is {datetime.now()}")
    await asyncio.sleep(3)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """サーバ起動前後のイベントを指定

    Parameters
    ----------
    app : FastAPI
        アプリインスタンス
    """
    scheduler = AsyncIOScheduler()
    scheduler.add_job(tick, "interval", seconds=1, max_instances=2)
    scheduler.start()
    yield


app = FastAPI(lifespan=lifespan)
```

今回の例であれば、定期実行するジョブが `tick()` 関数になります。現在時刻を表示して、非同期的に3秒間待つシンプルな処理です。

このジョブを `AsyncIOScheduler` インスタンスに追加（ `add_job` ）します。追加するときにトリガーを指定できるのですが、今回は次の3つの指定をしています。

- `"interverl"`: 一定間隔で実行する
- `seconds=1`: 1秒間隔で実行
- `max_instances=2`: 最大2つまで同時に実行する

最後に、スケジューラを `start()` します。これでサーバを起動すると以下のような出力となります。

```log
INFO:     Will watch for changes in these directories: ['/rakuapi']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [1164] using WatchFiles
INFO:     Started server process [1171]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
Tick! The async time is 2024-04-25 07:37:46.483872
Tick! The async time is 2024-04-25 07:37:47.484277
Execution of job "tick (trigger: interval[0:00:01], next run at: 2024-04-25 07:37:48 UTC)" skipped: maximum number of running instances reached (2)
Execution of job "tick (trigger: interval[0:00:01], next run at: 2024-04-25 07:37:49 UTC)" skipped: maximum number of running instances reached (2)
Tick! The async time is 2024-04-25 07:37:50.484717
Tick! The async time is 2024-04-25 07:37:51.483725
Execution of job "tick (trigger: interval[0:00:01], next run at: 2024-04-25 07:37:52 UTC)" skipped: maximum number of running instances reached (2)
Execution of job "tick (trigger: interval[0:00:01], next run at: 2024-04-25 07:37:53 UTC)" skipped: maximum number of running instances reached (2)
Tick! The async time is 2024-04-25 07:37:54.484009
Tick! The async time is 2024-04-25 07:37:55.484479
```

今回非同期処理を採用しているので、2つ目のジョブは1つ目のジョブの完了を待たずに1秒間隔で実行されています。

しかし3秒目の実行タイミングでは、「Execution...」と出ていると思います。これは`max_instances=2`の制限に引っかかっていることを表しています。1秒間隔でジョブを実行しようとしているものの、ジョブがすでに2つ立ち上がっており完了していない（3秒待ちがある）ため、実行がスキップされています。

## まとめ

今回は、APSchedulerを使ってFastAPIで定期ジョブを実行する方法を紹介しました。非常にシンプルな使い方でジョブを定義できるので便利でした。とはいえ、AWS上であればLambdaとかを使えば解決する内容ではあると思います。このあたりはケースバイケースで良いものを選択する必要がありますね。