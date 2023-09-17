---
title: "【Java勉強-第3回】簡単なif・else文"
date: 2020-02-28T16:26:00+09:00
description: "Javaで簡単なif・else文を試してみました"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/java.png
categories:
 - 技術系備忘録
tags:
 - Java
series:
- Java入門
---

Java勉強記録第3回の今回は、簡単なif・else文を試していこうと思います。

今回は本当に触る程度で、今後実課題に取り組む時に（そんなときはこなさそう）、色々詳しく見ていこうかと思います。

極力こまめに投稿しようとした結果、一回の内容が薄くなりがち...

## if・else文

まず実例からということで、文字列型変数 `team` と `team2` にそれぞれ「ヴィッセル神戸」と「vissel kobe」を格納。
「ヴィッセル神戸」のときに、「最高です！」、そうでない時に、「!」を語尾に追加して出力するif・else文を作成しました。

```java
public class PracticeIf {
    public static void main(String[] args) {
        // 当てはまる時
        String team = "ヴィッセル神戸";

        if(team=="ヴィッセル神戸") {
            System.out.println(team + "最高です！");
        }
        else {
            System.out.println(team + "!");
        }

        // 当てはまらない時
        String team2 = "vissel kobe";

        if(team2=="ヴィッセル神戸") {
            System.out.println(team2 + "最高です！");
        }
        else {
            System.out.println(team2 + "!");
        }
    }
}
```

このプログラムを実行した結果が以下のとおりです。

```text
--出力結果--
ヴィッセル神戸最高です！
vissel kobe!
```

このようにちゃんと条件分岐されていることがわかります。
ここでif・else文の基本的な使い方について見てみると以下のようになります。

```java
if(条件式) {
    Trueの場合の処理
}
else {
    Falseの場合の処理
}
```

ifの直後のカッコ内の条件式がTrueかFalseかで処理が分かれる最も簡単なif文です。
このあたりはC言語とそんな変わらないですね。

条件式については、その他の言語と同様の比較演算子を用いる事ができます。
比較演算子を用いた例を以下に示します。

これらは内容が正しければTrue、そうでなければFalseを返します。

{{< mdtable class = "simple-table" >}}
| 条件式 | 内容 |
|----------|----------|
| a == b | aとbが同じ |
| a != b | aとbが違う |
| a > b | aがbより大きい |
| a >= b  | aがb以上  |
| a < b | aがbより小さい |
| a <= b  | aがb以下  |

{{< /mdtable >}}

## else if文でさらなる分岐

続いて、`else if`を用いてifの条件式でFalseになった時に、さらなる条件式に当てはまるかどうかで処理を分岐させます。

具体的なプログラムで見てみましょう。

```java
public class ForIf {
    public static void main(String[] args) {
        String tournament = "天皇杯"; // "ゼロックス"と"Jリーグ"の場合も検証

        if(tournament=="天皇杯") {
            System.out.println(tournament + "優勝！");
        }
        else if(tournament=="ゼロックス") {
            System.out.println(tournament + "優勝！");
        }
        else {
            System.out.println(tournament + "今年こそ優勝！");
        }
    }
}
```

この結果は以下のようになります。

```text
天皇杯優勝！
ゼロックス優勝！
Jリーグ今年こそ優勝！
```

## まとめ

今回は、超単純なif文についてまとめました。
入れ子とかinstanceof演算子等、色々他にもすべきことが大量にあるのですが、
また次の機会ということで...。
次回はfor文と配列についてかなと思います。
