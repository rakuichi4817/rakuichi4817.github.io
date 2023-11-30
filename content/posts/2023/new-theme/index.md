---
title: "ブログのテーマを「blowfish」に変えてみた"
date: 2023-11-30
description: "アウトプット強化月間のはずがブログのテーマ差し替えに時間を取られました..."
summary: "アウトプット強化月間のはずがブログのテーマ差し替えに時間を取られました..."
draft: false
heroStyle:
categories:
 - テック系
tags:
 - hugo
---

11月はアウトプット強化月間にするぞ～と思っていたのですが、ブログ全体の調整に目が向き全然コンテンツを増やすことができませんでした...。ということで、このブログのテーマを一新したのでそちらを簡単に紹介しようと思います。

## HugoテーマのBlowfishへの変更

このブログは[Hugo](https://gohugo.io/)を利用して作成しており、利用するデザインテーマを「[Blowfish](https://blowfish.page/) 」に変更しました。前に採用していたテーマは「[Zzo](https://zzo-docs.vercel.app/)」だったのですが、更新が止まっていたのが主な理由で乗り換えを検討していました。

こちらのテーマ、何故かロゴが日本語でも書かれており興味を引かれました。

{{< github repo="nunocoracao/blowfish" >}}

リポジトリを見ても比較的動きがあり、テーマとしても以下の点に惹かれました。

- 記事一覧ページの表示の仕方をパラメータで調整できる
    - カード表示
    - リスト表示
- サムネイル画像をバックグラウンドに表示したり、シンプルに表示したりできる
- shortcodesも充実している。

## パラメータ調整が簡単でカスタマイズ性も高い

こちらのテーマは、簡単に・柔軟にデザインをカスタマイズできる点が強みの一つです。

例えば、記事の一覧をカードタイプや通常のリストタイプへの切り替え機能があります（下図参照）。設定ファイル（.toml）上で `true` と `false` を切り替えるだけで実現でき非常に簡単です。他にも様々な設定を切り替えることができます。

{{< gallery >}}
{{< figure src="home1.png" class="grid-w45">}}
{{< figure src="home2.png" class="grid-w45">}}
{{< /gallery >}}

またこちらのテーマ自体、CSSフレームワーク「Tailwind CSS 3.0」を利用しているので、自分でCSSを適用することも簡単にできます。実際このブログも、元テーマから少し手を加えています。

## サムネイル画像の違い

各記事単位でも設定値を書き換えることで、以下のようにサムネイル画像の表示を切り替えることができます。

{{< gallery >}}
{{< figure src="simple.png" caption="simple" class="grid-w50 md:grid-w33 xl:grid-w25">}}
{{< figure src="big.png" caption="big" class="grid-w50 md:grid-w33 xl:grid-w25">}}
{{< figure src="background.png" caption="background" class="grid-w50 md:grid-w33 xl:grid-w25">}}
{{< figure src="thumbAndBackground.png" caption="thumbAndBackground" class="grid-w50 md:grid-w33 xl:grid-w25">}}
{{< /gallery >}}

## shortcodesの充実

上記の写真を並べる表示も shortcodes のひとつなのですが、他にも色々あります。詳細は[公式ページ](https://blowfish.page/docs/shortcodes/)を見ていただければと思いますが、いくつか具体的な表示を載せておきます。

### タイプ表示

{{< typeit >}}
タイプするかのような表示もできるよ！
{{< /typeit >}}

### katexによる数式表示

tex形式の数式も作れます。

{{< katex >}}
\\(f(a,b,c) = (a^2+b^2+c^2)^3\\)

### グラフ表示


{{< chart >}}
type: 'bar',
data: {
  labels: ['神戸', '横浜FM', '広島', '浦和', '名古屋'],
  datasets: [{
    label: '2023年11月30日時点 勝点数',
    data: [68, 64, 55, 54, 51],
  }]
}
{{< /chart >}}

## 改めてコンテンツの追加を頑張る

ブログのテーマも刷新したことですし、改めて投稿を頑張っていこうと思います。あまり難易度をあげないよう細かい内容の投稿も増やすかもしれませんが、継続を重視していこうかと。年末も近づいてきたので、恒例の振り返りであったり買ってよかったものシリーズも投稿していく予定です。