---
title: "Windows（Python Launcher）のPipenvでバージョンを指定して仮想環境を作ると失敗する問題の解決法"
date: 2023-09-17
description: "Windows上でPipenvを使用し、Pythonバージョンを指定して仮想環境を作成しようとしたとき、Python Launcher管理下にある指定Pythonバージョンを見つけられず、エラー「Warning: Python ○○ was not found on your system...」が出る問題の解決法をメモしておきます。"

categories:
 - テック系
tags:
 - Python
 - Pipenv
 - 環境構築
aliases:
 - /posts/pipenv-pylauncher/
---

Windows上でPipenvを使用し、Pythonバージョンを指定して仮想環境を作成しようとしたとき、Python Launcher管理下にある指定Pythonバージョンを見つけられず、エラー「Warning: Python ○○ was not found on your system...」が出る問題の対策を紹介します。

## 状況と原因

問題が起きる環境は以下の通りです。

- Windows11
- Python公式のインストーラを使用、環境変数にパスは通さずPythonランチャー経由で実行管理（以下バージョンのインストール）
    - Python3.11
    - Python3.10

この状況でPipenvを使って仮想環境を作成していきます。Python3.11側にPipenvを入れている状態で、Python3.10の仮想環境を作ります。通常のPipenvと同じようにコマンドを実行するとエラーになります。

```shell
$ py -m pipenv --python 3.10

Warning: Python 3.10 was not found on your system...
Neither 'pyenv' nor 'asdf' could be found to install Python.
You can specify specific versions of Python with:
$ pipenv --python path\to\python
```

Python3.10を入れているのにPipenvが見つけられていません。原因は、**「Pythonランチャー経由でPythonを実行しており、各バージョンのPython自体（Python.exe）の環境変数を通していない」** からとなります。

Pythonインストーラーでインストールするときに環境変数に通してしまえば解決はするのですが、Windows上では複数バージョンの管理が怪しくなるのでしたくありません。

## 解決法

解決方法は仮想環境を作成する際に直接Pythonの実行ファイルのパスを通してあげると通ります。もしかしたらほかにも方法があるかもしれませんので、解決策の1つだと思ってもらえれば。

まずはPythonランチャーが管理しているPythonリストを表示します。

```shell
$ py --list-paths
 -V:3.11 *        C:\Users\rakuichi\AppData\Local\Programs\Python\Python311\python.exe
 -V:3.10          C:\Users\rakuichi\AppData\Local\Programs\Python\Python310\python.exe
```

今回は3.10を使って仮想環境を作るので以下のようなコマンドになります。

```shell
py -m pipenv --python=C:\Users\rakuichi\AppData\Local\Programs\Python\Python310\python.exe
```

これで無事仮想環境を作成することができます。既存のPipfileより環境を再現したい場合は、上記コマンドに `install` または `sync`をつけましょう。

## 課題

連携されたPipfileから環境を再現する際、普通であればいちいちPythonのパスを指定しなくても良いです。しかし、今回のような環境では以下のような手順になってしまいます。

1. 連携されたPipfileを確認しPythonバージョンを確認
2. バージョンのパスを確認
3. パスを指定しながら環境再現コマンドを実行

少し手間だと感じます。

## 結論

やっぱりDockerを使いましょう。
