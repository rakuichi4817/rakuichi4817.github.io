---
title: "Windows(CPU)にAnacondaでTensorFlow2.1.0とKeras2.3.1の導入"
date: 2020-04-13T23:00:00+09:00
description: "Windows上にAnacondaでKerasの環境を作成します"
draft: false
aliases:
 - /posts/windows-cpu-keras/
categories:
 - テック系
tags:
 - Python
 - 環境構築
 - ML
---

{{< alert >}}
**【2021年9月18日追記】**
本記事ではcondaを用いて仮想環境を作成し、その内部でpipを使ってライブラリをインストールしています。しかし、2021年途中以降で脱Anacondaをしましたので、こちらの方法がおすすめというわけでもありません。
{{< /alert >}}

こんばんは、4月から始まった会社の研修も、Zoomによるリモート研修になってしまい、一日中パソコンとにらめっこしているRakuichiです。

会社ではJavaの勉強をするので、せっかく大学院で利用してきたPythonの感覚を鈍らせないためにも、Keras（TensorFlow）を用いたディープラーニングの勉強をすることにしました。

私は意地でもWindowsマンで、WindowsのCPU版での環境を作りましたので、今回はそれをまとめたいと思います。
他のネットブログ等にはインストール時のバージョンなどが書かれていないこともあるので、この記事ではしっかり記しておきたいと思います。

KerasとTensorFlowの説明に関しては省きますが、KerasはTensorFlowのAPIであり、TensorFlowが必須となります。

## 環境

記事作成時の私の環境は以下になります。
ラップトップでGPUがないので、CPU版での環境構築になります。
また、開発にはJupyter Labを利用しているためそちらの設定も行います。

- Windows10 Pro （CPUのみ）
- Anaconda3
- Jupyter Lab
- Python 3.6
- TensorFlow 2.1.0
- Keras 2.3.1

## インストールの手順

インストールの手順は以下になります。

1. Anacondaで仮想環境の作成
2. conda環境の切り替えとJupyter上のカーネルの登録
3. 「Visual C++ Build Tools 2019」のインストール
4. 「TensorFlow」のインストール
5. 「Keras」のインストール

私が個人的にハマったポイントは、3つ目のVisual C++ Build Tools 2019を入れてなかった点です。このため、TensorFlowのインストールはうまくいくのに、Pythonでインポートができない問題が発生しました。
（新しいPCに変えて入れるの忘れてましたが、こいつを入れていないと色んなもののインストール時に問題が発生します。）

### 1. Anacondaでの仮想環境の作成

KerasとTensorFlowには対応したPythonのバージョンが存在しているため、普段使っているPythonのバージョンでは動かない可能性があります。そのため、Anacondaの仮想環境機能を利用して、適したPythonバージョンの環境を作成します。

ここで、TensorFlowとKerasの公式サイトよりPythonの対応バージョンを見てみます。

> Python 3.5–3.7  
> [TensorFlow公式サイトより](https://www.tensorflow.org/install)

> KerasはPython 2.7-3.6に対応しています．  
> [Keras公式サイトより](https://keras.io/ja/#_2)

この両方を満たすPythonのバージョンのなかから、今回はバージョン __3.6__ で環境を作ります。

condaの仮想環境を作成するためにコマンドプロンプト上で以下のコマンドを入力します。意味としては、「keras-cpu」という名のAnaconda環境を作成し、「Pythonバージョンの3.6」と「Jupyter」をインストールすることになります。

```shell
conda create -n keras-cpu python=3.6 jupyter
```

### 2. conda環境の切り替えとJupyter上のカーネルの登録

作成した環境に切り替える方法と、Jupyter上で環境を切り替える方法について述べていきます。

コマンドプロンプト（コンソール上）でのconda環境の切り替えは以下のとおりです。

```shell
activate keras-cpu
```

コマンド実行後、`C:\Users\ユーザ名>` の前に `(keras-cpu) ` が入れば切り替え完了です。

続いて、Jupyter上でカーネルを切り替えるための設定をします。上記コマンドで環境を切り替えた状態で以下のコマンドを実行します。

```shell
ipython kernel install --user --name=keras-py3.6 --display-name=keras-py3.6
```

このコマンドにより、今適応されているconda環境がJupyter上で切り替えられるようになります。

「--display-name=」に続けて書かれたものが、Jupyter上で表示されます（多分）。

実際、登録後にJupyter Labを起動してみると、以下のようにPythonのKernelに、もともと存在していた「Python 3」と、新たに作成した「keras-py3.6」の2つが出てきます。

Keras等をインストールしたあとは、この新しく作成したKernelを利用して実行するようにしてください。

{{< figure src="featured-jupyterlab.png" caption="Jupyter Labの画面">}}

### 3. 「Visual C++ Build Tools 2019」のインストール

こちらをインストールしていない状態でTensorFlowとKerasをインストールすると、インストールはうまくいくものの、いざPythonでインポートするとエラーを吐いて動かない、ということになります。

[公式のTensorFlowのWindowsに関するページ](https://www.tensorflow.org/install/source_windows)を見ると、以下のように書かれています。こちらに従って、必要なものをインストールしてください。

> #### Visual C++ Build Tools 2019 をインストールする
> Visual C++ Build Tools 2019 をインストールします。これは Visual Studio 2019 に付属していますが、別々にインストールすることもできます。
> 1. Visual Studio のダウンロード サイトに移動します。
> 2. [再頒布可能パッケージおよびビルドツール](https://visualstudio.microsoft.com/ja/downloads/?rr=https%3A%2F%2Fwww.tensorflow.org%2Finstall%2Fsource_windows)を選択します。
> 3. 以下をダウンロードしてインストールします。
> - Microsoft Visual C++ 2019 再頒布可能ファイル
> - Microsoft Build Tools 2019
>
> 引用元： https://www.tensorflow.org/install/source_windows

### 4. TensorFlowのインストール

以上の手順が終わり次第、 `pip` を使ってTensorFlow（バージョン2.1.0）をインストールします。

TensorFlowのバージョン1系では、CPU版とGPU版が分けられて扱われていますが、最新の2系では公式ページ（以下で引用）に記されている通り、両方に対応しているため、気にすることなくインストールが可能です。

> #### TensorFlow 2 パッケージが利用可能
> tensorflow - 最新の安定版リリース、CPU および GPU サポート（Ubuntu、Windows 用）

インストールする際には、`activate keras-cpu` をした状態で以下のコマンドを実行してください。関連パッケージなども勝手にインストールされます。

```shell
pip install tensorflow
```

### 5. Kerasのインストール

TensorFlowのインストールが無事終われば、Keras（バージョン2.3.1）は、以下のコマンドで問題なくインストール可能です。

```shell
pip install keras
```

## 確認
すべての環境構築が終わればJupyter Labで「keras-py3.6」のKernelを選択し、
いかのプログラムでインポートができれば完了です。
おそらく「Keras」のインポート時には、バックエンドで何を使っているか（今回の場合はTensorFlow）が表示されると思います。

```python
import tensorflow
import keras
```

## まとめ

今回はCPUのWindows環境で、Anacondaを用いたTensorFlowとKerasの環境構築について説明しました。

比較的簡単にできるとは思いますが、Visual C++ Build Tools 2019をインストールしないといけないことが、落とし穴になっていたりするので注意していただければと思います。
