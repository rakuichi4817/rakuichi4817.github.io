---
title: "hugoでshortcodesを用いてtableにcssを適応"
date: 2019-10-03T12:00:06+09:00
description: "shortcodesを用いてtableになんとかcssを適応します"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/hugo-small.png
categories:
- 技術系備忘録
tags:
 - Hugo
 - Web
series:
-
---

久しぶりにHugoの投稿をしようと思います。
今回の内容は、**「マークダウン形式で書いたtableに対して、cssを適応させる」** です。

## 通常のマークダウンで記入

例えば、マークダウン形式でテーブルを書くと、以下のようになります。
{{< highlight md >}}
| 選手                     | 背番号 | 国       |
| ------------------------ | ------ | -------- |
| 古橋 亨梧                | 16     | 日本     |
| アンドレス・イニエスタ   | 8      | スペイン |
| トーマス・フェルマーレン | 4      | ベルギー |
{{< /highlight  >}}

__↓サイト上での表示↓__

| 選手                     | 背番号 | 国       |
| ------------------------ | ------ | -------- |
| 古橋 亨梧                | 16     | 日本     |
| アンドレス・イニエスタ   | 8      | スペイン |
| トーマス・フェルマーレン | 4      | ベルギー |

表示されるテーブルのデザインは、利用しているテンプレートに乗っ取る形になります。

## 自作shortcodesで任意のcssを適応

自分で制作したcssをマークダウンで記入したテーブルに反映してみた結果が以下になります。
苦肉の策みたいなやり方なので、他に良いやり方もあるかもしれません。

まず、以下のようなshortcodeファイルを作成します。

```mdtable.html
{{if (.Get "class")}}
    <div class = "{{ .Get "class" }}">
{{ else }}
    <div class = "simple-table">
{{ end }}
{{.Inner | markdownify }}
</div>
```

続いて、cssを以下のように作成します。

```css
div.simple-table table {
    width: 100%;
    border-top: 1px solid #e2e2e2;
    border-left: 1px solid #e2e2e2;
    background: #ffffff;
    margin-bottom:15px;
    margin-left: auto;
        margin-right: auto;
        color:#000000;
}

div.simple-table table th{
    border-right: 1px solid #e2e2e2;
    border-bottom: 1px solid #e2e2e2;
        padding:10px 0;
    text-align: center;
}
div.simple-table table td {
    font-size: 0.9em;
    border-right: 1px solid #e2e2e2;
    border-bottom: 1px solid #e2e2e2;
    text-align: center;
    padding:10px 0;
}
div.simple-table table th {
    background-color: #f3f0f0;
}
```

そして、以下のようにマークダウンを記入します。

```md
{{</* mdtable class = "simple-table" */>}}
| 選手                     | 背番号 | 国       |
| ------------------------ | ------ | -------- |
| 古橋  亨梧               | 16     | 日本     |
| アンドレス・イニエスタ   | 8      | スペイン |
| トーマス・フェルマーレン | 4      | ベルギー |
{{</* mdtable */>}}
```

これらを適応すると、以下のような表示になります。

{{< mdtable class = "simple-table" >}}
| 選手                     | 背番号 | 国       |
| ------------------------ | ------ | -------- |
| 古橋 亨梧                | 16     | 日本     |
| アンドレス・イニエスタ   | 8      | スペイン |
| トーマス・フェルマーレン | 4      | ベルギー |
{{< /mdtable >}}

### ざっくりとした説明

最初は`{{replace}}` を使って`{{ {.Inner | markdownify }}`で、マークダウン形式からマークアップに変換し、受け取った`<table>`の中身を書き換える方法を考えていました。
しかし、あまりうまく動かず...
ですので回避策として、shortcodes側で`<table>`の外側にクラスを適応した `<div>`をかまして、cssで対応させる手法をとりました。処理の流れ等の詳しい説明は今回は割愛します。

## 参考ページ
[Hugo shortcodesドキュメント](https://gohugo.io/content-management/shortcodes/)

## 関連ページ
[独自のショートコードを作成する | まくまくHugo/Goノート](https://maku77.github.io/hugo/shortcode/create-shortcode.html)
