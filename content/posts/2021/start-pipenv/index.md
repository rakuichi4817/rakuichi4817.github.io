---
title: "脱AnacondaしてPipenvでPython環境作り"
date: 2021-09-26
description: "Pythonの環境作りをAnacondaからPipenvに変更しました"
aliases:
  - /posts/start-pipenv/
categories:
 - テック系
tags:
 - Python
 - Pipenv
 - 環境構築
---

大学4年からPythonを触りだしましたが、その時から一貫してAnacondaでPythonの環境を作成していました。しかし、脱AnacondaしてPipenvに乗り換えたので、簡単にまとめようかと思います。

## Anacondaを利用していた理由とやめた理由

特にこだわりがあってAnacondaを利用していたわけではなく、右も左も分からないときに最初に使ったのがAnacondaで、結果的にずっと利用していました。研究室に配属された新人向けに、Pythonの環境構築としてAnacondaの説明が載っていたのです。

配属されるまで、授業以外でプログラミングなどに触れず、環境構築すらよくわかってなかった自分にとっては、とりあえずAnacondaで良かったかなとは思います。研究で使う最低限のライブラリも最初から入ってますし、個人で研究などに使う分には今でもAnacondaでも良いのかなと思います。おそらく研究室の方針としてもそういった意図があり、入ってきた生徒にAnacondaによる環境構築を説明していたのだと思います。

しかし、最近は自分の作ったプログラムを他の人の環境で動かしてもらうことが多くなりました。そのときに、「じゃあAnacondaを入れてcondaで仮想環境を作って」というわけにもいかず、condaとpipを混ぜてしまう危険性※（最近は問題ないとか）もあったりと、多くの問題が発生しました。  
*※ わりと有名だと思っている記事：[condaとpip：混ぜるな危険](https://onoz000.hatenablog.com/entry/2018/02/11/142347)*

何より、そもそも会社で使うには有償なんですよね。（まとめてくださっている記事：[Anaconda パッケージリポジトリが「大規模な」商用利用では有償になっていた](https://qiita.com/tfukumori/items/f8fc2c53077b234384fc)）

さらに、自分自身で使うときも複数の環境でコードを実行することが増え、そのたびにAnacondaで環境を作るのがめんどくさくなっていました。そこで、思い切って環境の作り方を変えようということで、こちらの記事「[Anacondaが有償化されて困っている人に贈る、Pythonのパッケージ管理](https://qiita.com/c60evaporator/items/ba41cef4b37465c39948)」を参考にして、Pipenvで環境を作ることにしました。

Pipenvを選んだ理由は**仮想環境を作って開発環境を揃える事ができそうだから**、というのが一番の理由です。私も自分の状況を見ながら、上の記事を参考にして決めました。Pipenvに変えてしばらくしますが、非常に満足しています。

## 脱AnacondaとPipenvの導入

### 脱Anaconda

まず、Anacondaで作った環境を削除します。基本的にはこちらの[公式ドキュメント](https://docs.anaconda.com/anaconda/install/uninstall/)を参考にしていただければと思います。

ステップは以下の2つです。

1. `anaconda-clean`で関連ファイルをまとめて削除
2. コントロールパネルからPythonの削除

Anacondaをインストールすると、様々な環境情報が色んな所に保存されます。それらをまとめて削除してくれる`anaconda-clean`を導入します。下記コマンドで、導入、実行を行うとホームディレクトリに`.anaconda_backup`というディレクトリが作成されます。必要がなければこちらのファイルも消して良いと思います。

```shell
conda install anaconda-clean
anaconda-clean --yes
```

続いて、コントロールパネルから「Python 3.x.x（Anaconda 3.x.x）」のアンインストールを実行すれば、脱Anacondaの完了です。

### PythonとPipenvの導入

まずは純正のPythonをインストールします。私は[Python.jp](https://www.python.jp/index.html)でまとめてくれている以下のダウンロードリンクから最新版をダウンロードしました。ダウンロードできたらインストーラを起動します。
　※ダウンロードリンク：[https://pythonlinks.python.jp/ja/index.html](https://pythonlinks.python.jp/ja/index.html)

インストーラを起動したら「Add Python 3.〇 to PATH」にチェックを入れるようにします。インストールが完了したら、`pip`で`pipenv`をインストールします。

```shell
pip install pipenv
```

続いて、環境変数を設定していきます。デフォルトでは、Pipenvで環境を作る際に、その環境ファイルがホームディレクトリに作成されてしまいます。こちらの設定をすることによって、実行したカレントディレクトリ（プロジェクト）配下に作成することができます。

環境変数名に`PIPENV_VENV_IN_PROJECT`、変数値に`true`を設定すればOKです。

{{< figure src="envval.png" caption="環境変数の設定" alt="環境変数の設定" >}}

これで最低限の設定は完了になります。

## 試しに仮想環境を作ってみる

試しに仮想環境を作ってみます。先程のPythonの導入でPython3.9を入れていたとします。この状態で、Python3.8の仮想環境を作ってみます。手順としては以下の通りになります。

1. Python3.8をインストール（「Add Python 3.〇 to PATH」にチェックを入れない）
2. プロジェクトディレクトリの作成
3. pipenvで仮想環境を作成

ここで、PipenvでPythonのバージョンを指定して環境構築する際の注意事項について、以下にまとめておきます（Python3.8で仮想環境を構築する前提）。

{{< alert>}}

**注意事項**

- Python3.9がすでに入っていても、Python3.8も別に入れる必要がある
- すでに導入されているPythonがバージョン3.8系である場合は入れる必要はない
- Pythonのバージョンは3.△.〇が存在するが、△が違う場合は別のものとしてインストールされ、〇が違うものに関しては、最新版で上書きされる
    - Python3.9.1とPython3.8.1は別でインストール
    - Python3.9.1とPython3.9.2を入れると3.9.2しか残らない
{{< /alert >}}

### 利用したいバージョンのPythonの導入（今回は3.8）

まず、Python3.8のインストールを行います。このとき、基本となるPythonとかぶらないよう、**「Add Python 3.〇 to PATH」にチェックを入れない**ようにしておきます。インストールが完了したら下記コマンドで出力を確認します。下記の例ではバージョン3.9と3.8が導入されており、`py`で実行すると、基本的には3.9が走ることを確認できます。

```shell
> py --list-paths
Installed Pythons found by C:\WINDOWS\py.exe Launcher for Windows
 -3.9-64        C:\Users\<username>\AppData\Local\Programs\Python\Python39\python.exe *
 -3.8-64        C:\Users\<username>\AppData\Local\Programs\Python\Python38\python.exe
```

### プロジェクトディレクトリの作成と仮想環境作成

今回は「check-pipenv」というディレクトリを作成し、その配下で開発をすすめるという前提で仮想環境を作成します。以下のコマンドで簡単に作成することができます。

```shell
mkdir check-pipenv
cd check-pipenv
py -m pipenv --python 3.8
```

エクスプローラで確認すると以下のように「.venv」と「Pipfile」が生成されていることがわかります。「.venv」は指定バージョンのPythonであったり、ライブラリの本体が保存されています。一方「Pipfile」には仮想環境の情報がテキストで保存されています。作成した仮想環境のを他の環境で再現する際は、「.venv」は共有する必要はなく、「Pipfile」さえあれば、`pipenv`で環境を再現することができます。

{{< figure src="pipenv.png" caption="作成されるファイル" alt="作成されるファイル" >}}

Pythonのバージョンが3.8として認識されているかを確認したい場合は、`pipenv shell`を実行して、仮想環境に入ってから`python --version`を実行するか、`pipenv run python --version`を実行することで確認できます。

```shell
pipenv shell
python --version
```

または

```shell
pipenv run python --version
```

仮想環境でPythonを実行したい場合も以上の2種類どちらかの方法で実行することができます。

### 仮想環境にライブラリを入れる

作成した仮想環境上にライブラリを入れる場合は、プロジェクトディレクトリ上で`pipenv install 〇〇`とすればOKです。試しに「numpy」を入れた例を示します。

```shell
>pipenv install numpy
Installing numpy...
Adding numpy to Pipfile's [packages]...
Installation Succeeded
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
 Locking...Building requirements...
Resolving dependencies...
Success!
Updated Pipfile.lock (456e4b)!
Installing dependencies from Pipfile.lock (456e4b)...
  ================================ 0/0 - 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

このコマンドを実行した後、「Pipfile」の中身に「numpy」の情報が書き加えられ、「Pipfile.lock」というファイルも作成されると思います。「.lock」がついているものについては次のセクションで紹介します。

### Pipfile（.lock）を用いて仮想環境の再現

今までの手順で、仮想環境の作成とその環境にライブラリを入れる方法がわかったと思います。では、この作成した仮想環境を他の場所、他の人に再現してもらうための手順を紹介します。手順としては非常に簡単で、「Pipfile」または「Pipenv.lock」を共有して、`Pipenv`のコマンドを走らせるだけです。

この時、「.lock」がついているものをもとに環境を構築すると、完全にライブラリのバージョンが揃えられてインストールされます。それぞれのコマンドは以下のとおりです。

```shell
# Pipfile.lockを利用する場合
py -m pipenv sync
# Pipfileを利用する場合
py -m pipenv install
```

こちらのコマンドを実行すると「.venv」ディレクトリが作成されていることが確認できます。これで、仮想環境の再現が完了となります。

## まとめ

今回はPythonの環境構築をAnacondaからPipenvに変えたことについて、簡単にですがまとめました。このあたりのわかりやすい記事は他にも色々上がっているので、自分の備忘録としてという形です。

今回の記事では触れていない複数の環境でjupyterを利用するときのカーネル設定の話や、vscodeで開発する際の環境指定の話などは、おいおい投稿しようかなと思います。

また、環境を揃えるという意味では、Dockerを使って土台から環境を揃えることも行っています。例えば、言語処理100本ノックの実行環境をDockerで作ってしまう、ということをしていたりします。ただ、若干学習コストが高くなるので、周りの人にすすめる際には難しいなぁと思っていたりもします。

