---
title: miseのインストールとPython環境の用意
date: 2024-05-28
description: "とりあえずmiseをインストールしてPythonの環境を雑に作ってみた"
summary: "とりあえずmiseをインストールしてPythonの環境を雑に作ってみた"
draft: false
heroStyle: "simple"
categories:
 - テック系
tags:
 - mise
 - Python
 - 環境構築
---

最近は開発環境を作るとき、基本的にコンテナを作ってそのうえで開発、という手段を取っていました。しかし、サクッとなにか作る際にいちいちコンテナ作って～というのも微妙にしんどいので、**mise** とやらを試してみることにしました。

## miseとは

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mise.jdx.dev/" data-iframely-url="//iframely.net/3LUKiEG"></a></div></div><script async src="//iframely.net/embed.js"></script>

公式サイトによると、以下のような特徴があるようです。

> **Dev Tools**
> 
> Mise は多言語ツールのバージョンマネージャです。これは、asdf、nvm、pyenv、rbenvなどのツールを置き換えます。
>
> **Environments**
> 
> MISE を使用すると、異なるプロジェクトディレクトリ内の環境変数のセットを切り替えることができます。direnv を置き換えることができます。
> 
> **Tasks**
> 
> mise は、make や npm スクリプトを置き換えることができるタスクランナーです。

私はPythonで開発することが多いですが、Python自体のバージョンの管理が少し煩わしいイメージだったので、miseで管理するのが良さそうだなぁと感じました。

5月からジョインしたチームではasdfを使っているのですが、タスクランナーとしてはmakeを使っています。miseではmakeの役割も担えると書いているので、本記事では触れませんがゆくゆくは触っていきたいと思います。

ちなみにasdfの比較は公式ドキュメントで触れられているのでぜひご覧ください↓

[Comparison to asdf](https://mise.jdx.dev/dev-tools/comparison-to-asdf.html)

## miseの環境構築

環境構築は[公式ドキュメント](https://mise.jdx.dev/getting-started.html)ほぼそのままなので、備忘録程度に。

### 前提

- wsl2
- Ubuntu24.04

### インストール

まずはインストールから

```shell
$ curl https://mise.run | sh
$ ~/.local/bin/mise --version
2024.5.23 linux-x64 (2466638 2024-05-27)
```

### pathを通す

以下コマンド実行後シェルの再起動。

```shell
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
```

### ツールを入れる

試しにPython3.11を入れてみます。雑にグローバル設定で入れます。入れたあと実行ファイルの場所も確認してみます。

```shell
$ mise use -g python@3.11
$ which python
/home/rakuichi/.local/share/mise/installs/python/3.11/bin/python
```

asdfは `asdf option add python` としてから、利用するバージョンをインストールする流れなので、シンプルになっているのかな？と思います。

## tomlファイルでプロジェクト用設定を記入して環境構築

miseでは設定ファイルとして基本的に「.mise.toml」を用意します（設定ファイルの参照順は[公式ドキュメント](https://mise.jdx.dev/configuration.html)上に記載されています）。asdfの「.tool-versions」と違い、様々な設定が書き込めるぜ！と[公式](https://mise.jdx.dev/configuration.html)はアピールしております。

今回は簡単に以下の要件を満たすような設定ファイルを作って、環境を作ってみようと思います。

- Python3.12
- Pythonは「.venv」上に環境を作る

ということで、いきなり設定ファイルを。

```toml
[tools]
python = {version='3.12', virtualenv='.venv'}

[env]
_.python.venv = { path = ".venv", create = true }
```

この設定ファイルがあるディレクトリ上で `mise i` すれば環境が作られます。「.venv」周りの設定が少し冗長なんですが、`[env]` 以下を書かないと、自動で「.venv」を作ってくれませんでした。

もしかしたらもっときれいに書ける方法があるかもしれないので、分かり次第修正しておきます。

## まとめ

簡単にmiseの導入とPython環境の構築をしてみたので、そちらをまとめてみました。まだまだ本格的な使い方はできていないので、タスクの実行等も触れてみようと思います。Streamlitで簡易アプリ作る前提で触ってみようかしら🤔

気が向いたらポエム的な記事も上げていきたい...