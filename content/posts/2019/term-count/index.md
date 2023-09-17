---
title: "Pythonで分かち書きされたテキストに対する単語出現回数をカウント"
date: 2019-10-27T12:00:00+09:00
description: "外部パッケージを使わずに単語の出現回数をカウントします"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/python.png
categories:
 - 技術系備忘録
tags:
 - Python
 - NLP
series:
-
---

Scikit-learnなどの外部パッケージを使わずに文書中の単語の出現回数を求めたいときのプログラムです。
<!--more-->

## 分かち書きされたテキスト

ヴィッセル神戸のチャント（応援歌）の中でも、私が好きな「アウェイマーチ」をサンプルに試してみます。

> どこまでも行こうぜ 勝利を信じて
> 熱き友の想い 胸に宿して
> 行こう 勝利へ トモニ戦え ラーララ ララララ
> 歌声響かせろ（神戸！） 遠く神戸まで（神戸！）
> さぁみんなで帰ろう 神戸に帰ろう 勝利この手に

以上のテキストに対して、分かち書きを行うと以下のようになります。
分かち書きとは、形態素と呼ばれる意味を持つ最小単位に分割することを言います。

```text
どこ まで も 行こ う ぜ 勝利 を 信じ て 熱き 友 の 想い 胸 に 宿し て 行こう 勝利 へ トモニ 戦え ラーララ ララララ 歌声 響かせろ （ 神戸 ！ ） 遠く 神戸 まで （ 神戸 ！ ） さぁ みんな で 帰ろ う 神戸 に 帰ろ う 勝利 この 手 に
```

## 標準ライブラリのcollections.Counterを利用する

`collections.Counter()`にリストなどを与えると、`Counter`オブジェクトが生成されます。`Counter`は辞書型`dict`のサブクラス（同じような形）で、keyに要素、valueに出現回数を持ちます。

```python
from collections import Counter

wakati_text = "分かち書きされた上記のテキストを入れる"

morph_counts = Counter([morph for morph in wakati_text.split(" ")])
print(morph_counts)
```

出力は以下のようになります。

```text
Counter({'神戸': 4, 'う': 3, '勝利': 3, 'に': 3, 'まで': 2, 'て': 2, '（': 2, '！': 2, '）': 2, '帰ろ': 2, 'どこ': 1, 'も': 1, '行こ': 1, 'ぜ': 1, 'を': 1, '信じ': 1, '熱き': 1, '友': 1, 'の': 1, '想い': 1, '胸': 1, '宿し': 1, '行こう': 1, 'へ': 1, 'トモニ': 1, '戦え': 1, 'ラーララ': 1, 'ララララ': 1, '歌声': 1, '響かせろ': 1, '遠く': 1, 'さぁ': 1, 'みんな': 1, 'で': 1, 'この': 1, '手': 1})
```

## 標準ライブラリのcollections.defaultdictを利用する

`collections.defaultdict()`を用いてカウントする方法もあります。
こちらの方法であれば、柔軟に処理を付け加えることができるので、汎用性は高いかもしれません。

`defaultdict`については、こちらの記事（[Qiita: Python defaultdict の使い方](https://qiita.com/xza/items/72a1b07fcf64d1f4bdb7)）が参考になるかと。

```python
from collections import defaultdict

# int型で初期化
morph_counts = defaultdict(int)

for morph in wakati_text.split(" "):
    morph_counts[morph] = morph_counts[morph] + 1
print(morph_counts)
```

## まとめ

今回はストップワードなどを全く設定していないので、
記号などが入っていますが、省く場合は以下の2通りの処理が考えられます。

1. 分かち書きの段階で品詞などにより残す単語を絞る
2. 辞書型に入力するときにストップワード（正規表現）を用いて絞る

ストップワードを簡易的に使うのであれば[SlothLibのテキストページ](http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt)を使うといいと思います。見やすくするにはワードクラウドとかを使ってみるのもいいですね。

## 関連情報

- [SlothLib: Webサーチ研究のためのプログラミングライブラリ](https://dbsj.org/wp-content/uploads/journal/vol6/no1/ohshima.pdf)
