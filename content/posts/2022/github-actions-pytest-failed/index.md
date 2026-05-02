---
title: "GitHub Actions・pytestで失敗した(Failed)時の挙動確認"
date: 2022-01-18
description: "シンプルにGitHub Actionsでpytestを動かし、テストが失敗した場合の挙動を見てみました。"
aliases:
 - /posts/github-actions-pytest-failed/
categories:
 - テック系
tags:
 - Python
 - pytest
 - GitHub
 - GitHub Actions
 - テスト
 - CI/CD
---

タイトルの通り、GitHub Actionsでpytestを実行し、テストが失敗（Failed）したときの挙動について確認してみました。

前回の記事「[GitHub Actions・pipenv・pytestで自動テストの練習]({{< ref "/posts/2022/github-actions-pytest/index.md" >}})」で、一連の流れの確認をしてみたのですが、テストが失敗のときはどうやって判断できるのだろうか？と思い、実際に試し、そのまとめになります（当たり前のことなのかもしれませんが、勉強なのであしからず）。

## 失敗するテストを作成

基本的な構成は「[GitHub Actions・pipenv・pytestで自動テストの練習]({{< ref "/posts/2022/github-actions-pytest/index.md" >}})」と全く同じにしています。失敗するテストの挙動を確認するために「pytest-failed」ブランチを作成して、そちらでテストの内容だけ、必ず失敗するように書き換えて試してみました。全体感などを確認したい方は上記記事を見てみてください。

今回利用した実際のリポジトリ：[https://github.com/rakuichi4817/study-actions/tree/pytest-failed](https://github.com/rakuichi4817/study-actions/tree/pytest-failed)

作成した必ず失敗するテストファイルは以下になります。`assert 2 == 3`となっているため、必ずテストは失敗になります。この状態で変更をコミット・プッシュすることで、GitHub Actions上で失敗するテストが実行可能になります。

```tests/test_sample.py
def test_failed():
    print("---failed test---")
    assert 2 == 3 
```

## GitHub Actionsの確認

プッシュするとエラーに関するメールが届きます。GitHub Actionsでは、処理に問題があるとメールが届く機能があります。下図のように問題があったworkflowへのリンクが貼られています。

{{< figure src="actions4.png" title="メール内容" caption="Pytestでエラーが出ていることを確認できる" alt="メール内容" width="50%" position="center">}}

GitHub Actions上で結果を確認してみます（あえてメールのリンクを踏んでいません）。

GitHubのリポジトリ上のActionsタブを見てみると、エラーを吐いていることが確認できます。ここでもworkflowがエラーとなっていることが判断できます。

{{< figure src="actions1.png" title="リポジトリのActionsタブ" caption="失敗しているテストでエラーが確認できる" alt="リポジトリのActionsタブ" width="100%" position="center">}}

ここで、今回のプッシュに該当する一番上のworkflowの詳細の確認をします。

StatusがFailureとなっていることがわかります。さらに「Annotations」が追加され、エラーに関する内容が書かれています。ただ、ここでは、「処理の中でエラーが発生したよ」くらいしかわかりません。軽く調べてみたところ、ここにエラー箇所を表示する方法もあるみたいです。

{{< figure src="actions2.png" title="エラーworkflowの詳細" caption="StatusがFailure" alt="リポジトリのActionsタブ" width="100%" position="center">}}

今回はそのまま「build」ジョブの中身を見て、エラーが出ているステップを確認します。

「Test with pytest」ステップでエラーが出ていることがわかり、そのまま詳細情報が表示されます（開いたときに自動で詳細も表示されました）。前節で作成した、失敗するテスト内容`assert 2 == 3`のところでFAILEDが出ていることがわかります。

{{< figure src="actions3.png" title="エラーjobの詳細" caption="Pytestでエラーが出ていることを確認できる" alt="エラーjobの詳細" width="90%" position="center">}}

## まとめ

今回は、GitHub Actionsでpytestが失敗したときにどういった挙動になるのかを確認しました。

pytestに失敗するとworkflowも失敗となり、お知らせメールが届くことを学べました。しかし、具体的なエラー内容はジョブの詳細を開くしかなく、Annotationsでは判断できません。このあたりは、カスタマイズすることでエラー箇所をAnnotationしたりもできるのかなと思っています。

次回以降、このあたりの勉強もできれば...。
