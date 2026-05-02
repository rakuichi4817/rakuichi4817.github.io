---
title: "Hugoのインストール方法（Windows）"
date: 2019-01-21
description: "Hugoのインストール方法について簡単にまとめます"
draft: false
aliases:
 - /posts/install-hugo/
categories:
- テック系
tags:
 - Hugo
 - 環境構築
---

Windows環境におけるHugoのインストール方法について紹介していきます。
<!--more-->
今回は、あまりこういった技術に慣れていない方向けに、丁寧めにまとめます。

HugoとはGo言語で書かれた静的サイトジェネレータです。
このサイトもhugoを用いて作成していますが、私が使っていて感じるメリットは以下の点です。

- blog形式の静的サイトを簡単に高速に作成できる
- テーマが豊富で、適応も簡単なため、こだわりがなければすぐにサイトを作成できる

以前までhtmlをゴリゴリ書いていた私にとっては画期的なサイトジェネレータでした。

## chocolateyを用いたインストール

chocolateyはLinuxでいう「apt」のようなパッケージ管理ツールです。
（いろんなパッケージをコマンドを打つことで入れることができます）
今回インストールするHugoはchocolateyを用いて簡単に入れることができます。

### chocolateyのインストール

まず**管理者用コマンドプロンプト**を開く。

- `Windowsキー(Windowsのマークが書かれたボタン)`と`r`の同時押し
- 表示された入力欄※1に `cmd` を入力し `ctrl`  `shift`  `Enter` を同時押し
- 「このアプリがデバイスに変更を加えることを許可しますか」というメッセージが出てくるが気にせずOKを押す

{{< figure src="prompt.png" title="ファイル名を指定して実行" alt="ファイル名を指定して実行">}}

出てきた画面（コマンドプロンプト）に、以下のコマンドをペーストすればインストールが始まります。始まらなければエンターをおしてください

```shell
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

表示の流れが終わればインストール完了です。
続いて、以下のコマンドでchocolateyをアップデートしておいてください。

```shell
choco upgrade chocolatey
```

### Hugoのインストール方法

chocolateyが入ってしまえば、Hugoは下記のコマンドで簡単にインストールすることができます。
インストールする際に、通常版の`hugo`と、「Sass/SCSS」対応の拡張版`hugo-extend`があります。私は利用するテーマの都合上、拡張版を選択しました。
※とりあえず拡張版を入れておけば、問題ないのかな？と思っています

```shell
choco install hugo-extended -confirm
```

インストールできたか確認するために、バージョン確認のコマンドを実行します。
以下のように`hugo version`と入力して、結果が帰ってきたらインストール完了です。

```shell
> hugo version
hugo v0.88.1-5BC54738+extended windows/amd64 BuildDate=2021-09-04T09:39:19Z VendorInfo=gohugoio
```

### 参考にしたページ

[chocolatey公式HP](https://chocolatey.org/install)  
[chocolatey公式HPパッケージ](https://chocolatey.org/packages/hugo)  
[Hugo公式HP](https://gohugo.io/)  
[Hugo公式HPインストール方法](https://gohugo.io/getting-started/installing#chocolatey-windows)  
