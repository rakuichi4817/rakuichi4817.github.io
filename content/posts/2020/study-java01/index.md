---
title: "【Java勉強-第1回】EclipceでHello World!"
date: 2020-01-13T21:00:00+09:00
description: "Eclipceを使ってJavaでHello Worldを出力してみます"
draft: false

categories:
 - テック系
tags:
 - Java
series:
- Java入門
series_order: 1
---

新年あけましておめでとうございます。
<!--more-->

4月から入社する会社でJavaを利用するので、
先駆けて勉強していこうと思い、せっかくなので勉強の記録をまとめていこうと思います。
あくまで記録なのでミスがあれば右下よりコメントお願いしますm(_ \_)m

今回は第1回ということで、Hello Worldを出力するまでを書いていきます。

## JavaとEclipceの導入

Javaの開発環境として、統合開発環境のEclipce
（[https://mergedoc.osdn.jp/](https://mergedoc.osdn.jp/)）を利用しています。
最新版は安定しない可能性があるので、今回は「__Eclipce 4.8 Photon__」を利用します。
JavaのインストールとEclipceのインストールに関しては以下のページを参考にしました。

### 参考ページ

Javaのインストール（Windows）：[https://eng-entrance.com/java-install-jdk-windows](https://eng-entrance.com/java-install-jdk-windows)
Javaのインストール（Mac）：[https://eng-entrance.com/java-install-jdk-mac](https://eng-entrance.com/java-install-jdk-mac)

## Eclipceでいちからの「Hello World」

Eclipceが起動できた前提で、「Hello World」を出力するまでの記録です。
あまり深く考えずに以下の通りに実行すれば「Hello World」を出力できると思います。

### 1. プロジェクトの作成

プロジェクトを作成することで、
ソースコードや関連パッケージを一つにまとめて管理することができるようです。

- 左上の「新規」をクリック
- 「Javaプロジェクト」の選択後「次へ」

{{< figure src="step02.jpg" caption="新規プロジェクトの作成" >}}

- プロジェクト名「JavaPractice」にして「完了」
{{< figure src="step02.jpg" caption="プロジェクト名の入力" >}}

- パッケージ・エクスプローラーのタブに「JavaPractice」ができる
{{< figure src="step03.png" caption="作成されたプロジェクトの確認">}}

### 2. パッケージの作成

- 「JavaPractice」で右クリック➝「新規」➝「パッケージ」をクリック
{{< figure src="step04.png" caption="パッケージの作成">}}

- 名前のところに「practice01」と入力し完了をクリック
{{< figure src="step05.png" caption="パッケージ名の入力">}}

### 3. クラスの作成

いよいよJavaファイルの作成です。

- 作成した「practice01」を右クリック➝「新規」➝「クラス」をクリック
{{< figure src="step06.png" caption="新規クラスの作成">}}

- 名前に「HelloWorld」と入力し「public static void main(Staing[] args)」にチェックを入れる➝「完了」\
クラスの名の頭文字は大文字にする（単語が複数続く場合は頭文字を大文字にしてつなげる）
{{< figure src="step07.png" caption="新規クラスの設定">}}

- 以下のように表示される\
このウィンドウでは右側にソースコードがあります
{{< figure src="step08.png" caption="クラス作成後の表示">}}

### 4. Hello Worldの出力

Hello Worldを出力するためのプログラムを書きます。

- 以下のようにソースコードを書く
細かい説明は抜きにしますが、`print` ではなく `println` を用いることで末尾に自動で改行が入ります

```java
package practice01;

public class HelloWorld {
    public static void main(String[] args) {
        // printlnは改行が末尾に入る
        System.out.println("Hello World");
    }
}
```

- 実行ボタンを選択し左下のコンソールに「Hello World」が出力されているか確認する\
このときうまく実行できない場合は「アウトライン」タブ内の、
「HelloWorld」クラスを右クリックし「実行」をクリックする。
{{< figure src="step09.png" caption="プログラムの実行">}}

## まとめと感想

今回は、Eclipseを使ってファイル作成から、
定番の「Hello World」を出力するまでに取り組みました。
普段Pythonしか触ってないので久しぶりに `{ }` を見てC言語をやってた頃を懐かしみました笑。
これからもぼちぼち頑張っていこうと思います！\
次回は __「変数」__ に取り組んでいく予定です。

## 参考サイト

[サーチマン佐藤のJava](https://searchman.info/java_eclipse/1090.html)
