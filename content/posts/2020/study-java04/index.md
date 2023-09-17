---
title: "【Java勉強-第4回】キーボード入力と文字列の連結"
date: 2020-04-20T18:26:00+09:00
description: "Javaでのキーボード入力と文字列の連結についてまとめました"
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


Java勉強記録第3回の今回は、キーボード入力と、文字列の連結についてまとめていきます。文字列の連結の仕方を工夫しないと、データ量が増えたときに処理が遅くなったりしますからね。

## キーボード入力の受付

Javaのキーボード入力の受付には、__java.util__ パッケージの __Scanner__ クラスを用います。

今回の入力には以下の3つのメソッドを利用しています。

- nextInt()：整数型の入力
- nextDouble()：少数型の入力
- nextLine()：文字列の入力

ここで、気をつけないといけないことは`nextInt()`や`nextDouble()`で数字を読み込んだ際に、改行文字（Enterを押すため）が残る点です。

そのまま`nextLine()`を使うと、その改行文字を読み込んで処理が終わってしまいます。ですので、`nextLine()`を間に一度入れることで空読みを行い、この問題を解決します。

```java
import java.util.Scanner;

public class Keybord{

    public static void main(String[] args){

        // Scannerクラスの呼び出し
        Scanner scan = new Scanner(System.in);
        // 数値の入力
        System.out.print("int型を入力：");
        int num1 = scan.nextInt();
        System.out.print("duble型を入力：");
        double num2 = scan.nextDouble();

        // 空読み
        scan.nextLine();
        // 文字列の入力
        System.out.print("String型を入力：");
        String strin = scan.nextLine();

        // 入力の確認
        System.out.println("【入力の確認】");
        System.out.println("int型：" + num1);
        System.out.println("double型：" + num2);
        System.out.println("String型：" + strin);
    }
}
```

このプログラムを実行した結果が以下のとおりです。うまく機能していることを確認できました。

```text
int型を入力：1
duble型を入力：10.0
String型を入力：vissel
【入力の確認】
int型：1
double型：10.0
String型：vissel
```

## 文字列の連結

文字列の連結には下記の通りいくつか種類があります。

- プラス演算子
- concat()
- join()
- StringBuffer
- StringBuilder
  
プラス演算子や、concat()の他にも、StringBufferやStringBuilderといったクラスを利用することで連結することができます。StringBufferやStringBuilderの2つの違いに関しては以下の記事を参考にしていただければ良いと思います。

今回はとりあえず一般的に処理が早い`StringBuilder`を利用します。

参考サイト：[文字列操作 スレッドセーフ vs パフォーマンス StringBuilder](http://www.javainthebox.net/laboratory/J2SE1.5/TinyTips/StringBuilder/StringBuilder.html)

プラス演算子や、concat()による連結処理は重いみたいです。特にプラス演算子でつなげるとめちゃくちゃ遅いみたいですね。今回、時間は計測していませんが、以下のQiitaの記事を見るとプラス演算子がいかに遅いかがわかると思います。

参考サイト： [【Java】文字列結合の速度比較](https://qiita.com/nkojima/items/0098dccbe4a593bc0306)

### プラス演算子

つなげたい変数や文字列を`+`でつなぎます。

```java
public class StringJoin {

    public static void main(String[] args) {
        // 文字列の連結
        String s1, s2, s3, sjoin;
        s1 = "ヴィッセル神戸";
        s2 = "天皇杯";
        s3 = "優勝";

        System.out.printf("s1 = %s\n", s1);
        System.out.printf("s2 = %s\n", s2);
        System.out.printf("s3 = %s\n", s3);

        // プラスで
        sjoin = s1 + s2 + s3;
        System.out.println("\nプラス演算子");
        System.out.println(sjoin);
    }
}
```

出力結果は以下のとおりです。

```text
s1 = ヴィッセル神戸
s2 = 天皇杯
s3 = 優勝

プラス演算子
ヴィッセル神戸天皇杯優勝
```

### concat()

`s1`に対して`s2`を連結させたい場合は`s1.concat(s2)`とします。

```java
public class StringJoin2 {

    public static void main(String[] args) {
        // 文字列の連結
        String s1, s2, s3, sjoin2;
        s1 = "ヴィッセル神戸";
        s2 = "天皇杯";
        s3 = "優勝";

        // concat
        sjoin2 = s1.concat(s2);
        sjoin2 = sjoin2.concat(s3);
        System.out.println("\nconcat");
        System.out.println(sjoin2);
    }

}
```

出力結果は以下のとおりです。

```text
concat
ヴィッセル神戸天皇杯優勝
```

### join()

`join()`を用いることで、複数の文字列型を一つにつなげることができます。第一引数に設定した値が、連結する文字型の間に入ります。

```java
public class StringJoin3 {

    public static void main(String[] args) {
        // 文字列の連結
        String s1, s2, s3;
        s1 = "ヴィッセル神戸";
        s2 = "天皇杯";
        s3 = "優勝";

        // joinメソッド
        String sjoin3 = String.join("-", s1, s2, s3);
        System.out.println("\njoin");
        System.out.println(sjoin3);
    }

}

```

出力結果は以下のとおりです。

```text
join
ヴィッセル神戸-天皇杯-優勝
```

### StringBuilder

StringBuilderクラスを用いて、`append()`していくことで連結できます。

```java
public class StringJoin4 {

    public static void main(String[] args) {
        // 文字列の連結
        String s1, s2, s3;
        s1 = "ヴィッセル神戸";
        s2 = "天皇杯";
        s3 = "優勝";

        // StringBuilder append
        StringBuilder sjoin4 = new StringBuilder();

        sjoin4.append(s1);
        sjoin4.append(s2);
        sjoin4.append(s3);

        System.out.println("\nStringBuilder append");
        System.out.println(sjoin4);
    }

}

```

出力結果は以下のとおりです。

```text
StringBuilder append
ヴィッセル神戸天皇杯優勝
```

## まとめ

キーボードからの入力の受け取りと、文字列の連結についてまとめました。正直な感想は、「Pythonの方がええなぁ」です（笑）。

とはいっても、システムを作る時にJavaの良さが生きるときもあると思うので、しっかり理解していきたいと思います。