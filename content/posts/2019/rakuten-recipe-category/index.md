---
title: "楽天レシピのカテゴリID一覧"
date: 2019-11-01T12:00:00+09:00
description: "楽天レシピAPIのカテゴリについて"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/cover/woman-pc.jpg
categories:
 - 技術系備忘録
tags:
 - RakutenAPI
 - Web
series:
-
---

楽天レシピAPIを利用するときにカテゴリID一覧を作成したので一部載せておきます。
利用した内容についてはまた機会があれば載せます。
<!--more-->

## 楽天レシピのカテゴリIDについて

楽天レシピのページ：[https://recipe.rakuten.co.jp](https://recipe.rakuten.co.jp)
楽天ウェブサービス:APIのページ： [https://webservice.rakuten.co.jp/document/](https://webservice.rakuten.co.jp/document/)

上記の楽天APIを用いて楽天レシピにおけるカテゴリのIDを取得しました。
カテゴリは階層構造になっているのですが、カテゴリのトップページURLを見ると、親子関係にあるカテゴリIDが `-` で繋がれています。

例えば以下のような階層構造があるときに下の表のようになっています。

- 人気メニュー
  - ハンバーグ
    - ハンバーグステーキ

|category_full_id | category_name | category_url
|----------|----------|----------|
| 30 | 人気メニュー | https://recipe.rakuten.co.jp/category/30/ |
|30-300 | ハンバーグ | https://recipe.rakuten.co.jp/category/30-300/ |
|30-300-1130 | ハンバーグステーキ | https://recipe.rakuten.co.jp/category/30-300-1130/ |

## 楽天レシピのカテゴリ一覧

このページではカテゴリの量が多いので全ては載せていませんが、一覧を下記URLよりtsv形式でダウンロードできます。

<a href="https://github.com/rakuichi4817/RelatedDataForRakutenAPI/blob/master/RecipeAllCategoryData_20190630.tsv" target="_blank">Github: rakuichi4817/RelatedDataForRakutenAPI
</a>
