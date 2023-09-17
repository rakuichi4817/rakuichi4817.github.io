---
title: "Ubuntu 18.04.4 LTSにElasticsearch、Logstash、Kibanaを導入"
date: 2020-06-21T09:30:00+09:00
description: "UbuntuにElasticstackを導入してみます"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
image: images/tech/elastic.png
categories:
 - 技術系備忘録
tags:
 - Elastic
 - 環境構築
series:
- 
---

今回はUbuntuにElastic Stackを導入する手順について記録も兼ねて書いていこうと思います。
<!--more-->

**Elastic Stack**は、Elasticsearch、Kibana、Beats、Logstashの4つのツールの総称です。[公式サイト](https://www.elastic.co/jp/elastic-stack)に書かれている各ツールの説明は以下のとおりです。

- Elasticsearch
  >特定のIPアドレスによるアクティビティを確認する、あるいはトランザクションリクエストのスパイクを分析する、または半径2キロ以内にある牛丼店を探す... データを使って解くあらゆる"課題"は、最終的に検索の課題です。Elasticsearchなら、データを手軽に、スケーラブルに格納、検索、分析することができます。
- Kibana
  >Kibanaでデータを美しく可視化して、探索をはじめてみましょう。ワッフルチャートやヒートマップ、時系列分析など、豊富な機能が揃います。多様なデータソースに対応する事前構成済みのダッシュボードを使って、KPIに注目したプレゼンテーションを作成したり、1つの画面ですべてのデプロイを管理したりすることもできます。
- Beats
  >Beatsは、データシッピングに特化した無料かつオープンなプラットフォームです。何百、何千ものマシンからLogstashやElasticsearchにデータを転送できます。
- Logstash
  >Logstashは、オープンソースのサーバーサイドデータ処理パイプラインです。膨大な数のソースから同時にデータを取り込み、変換して、お好みの格納庫（スタッシュ）に送信します。
  
この中でもElasticsearch、Kibana、Logstashの3つをインストールしていきます。

## 環境

今回導入する環境と、インストールしたバージョンは以下のとおりです。

- Ubuntu 18.04.4 LTS
- Java 1.8.0_252
- Elasticsearch 7.8.0
- Kibana 7.8.0
- Logstash 7.8.0

## インストールの手順

インストールの手順は以下になります。

1. Javaのインストール
2. Elasticsearchのインストール
3. Kibanaのインストール
4. Logstashのインストール

### 1. Javaのインストール

ElasticStackはJavaを入れておかないと動かないので入れておきます。

```shell
add-apt-repository ppa:webupd8team/java
sudo apt install openjdk-8-jre-headless
```

確認

```shell
$ java -version
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (build 1.8.0_252-8u252-b09-1~18.04-b09)
OpenJDK 64-Bit Server VM (build 25.252-b09, mixed mode)
```

### 2. Elasticsearch

#### 2.1. インストールと自動起動設定

Javaが正しくインストールできたら、下記コマンドでElasticsearchのインストールをします。

```shell
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update && sudo apt install elasticsearch
```

自動で立ち上がるように設定します。

```shell
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```

#### 2.2. 外部接続用に設定の変更

外部PCからアクセスするための設定を行います。まずはじめにポートの開放を行います。

```shell
sudo ufw allow 9200
sudo ufw allow 9300
```

下記コマンドで、Elasticsearchの設定を書き換えます。

```shell
sudo nano /etc/elasticsearch/elasticsearch.yml
```

以下の内容を加えます。

```elasticsearch.yml
network.bind_host: 0
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
```

#### 2.3. 再起動、状態と接続の確認

設定を書き換えたら、Elasticsearchの再起動を行います。

再起動後、起動状態の確認も行います。**Active**のところに「active」と書かれていれば正しく動いています。

```shell
$ sudo systemctl restart elasticsearch.service
$ sudo systemctl status elasticsearch.service

 ●elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: ena
   Active: active (running) since Sun 2020-06-21 15:50:49 JST; 2h 1min ago
     Docs: https://www.elastic.co
 Main PID: 3447 (java)
    Tasks: 110 (limit: 4915)
   CGroup: /system.slice/elasticsearch.service
           ├─3447 /usr/share/elasticsearch/jdk/bin/java -Xshare:auto -Des.networkaddress.cach
           └─3642 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/contro
```

<サーバのIPアドレス>:9200で外部からアクセス可能か確認することができます。接続すると以下のような内容が表示されるはずです。

```json
{
  "name" : "<サーバ名>",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "<cluster_uuid>",
  "version" : {
    "number" : "7.8.0",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "<build_hash>",
    "build_date" : "2020-06-14T19:35:50.234439Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 3. Kibana

#### 3.1. インストールと自動起動設定

インストールと自動起動設定は下記コマンドで可能です。

```shell
sudo apt install kibana
```

Kibanaの自動起動設定

```shell
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
```

#### 3.2. 外部接続用に設定の変更

Elasticsearchと同様に外部PCからアクセスするための設定を行います。

まずはじめにポートの開放を行います。

```shell
sudo ufw allow 5451
```

下記コマンドで、Kibanaの設定を書き換えます。

```shell
sudo nano /etc/kibana/kibana.yml
```

以下のよう一部を書き換えます。

```kibana.yml
server .host : "0.0.0.0"
```

#### 3.3. 再起動、状態と接続の確認

設定を書き換えたら、Kibanaの再起動を行います。

再起動後、起動状態の確認も行います。**Active**のところに「active」と書かれていれば正しく動いています。

```shell
$ sudo systemctl restart kibana.service
$ sudo systemctl status kibana.service

 ● kibana.service - Kibana
   Loaded: loaded (/etc/systemd/system/kibana.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-06-21 15:42:32 JST; 3h 5min ago
 Main PID: 860 (node)
    Tasks: 11 (limit: 4915)
   CGroup: /system.slice/kibana.service
           └─860 /usr/share/kibana/bin/../node/bin/node /usr/share/kibana/bin/../src/cli
```

ここまでくれば、「<サーバのIPアドレス>:5601」でKibanaの画面を確認できます。

{{< figure src="kibana.png" caption="Kibana画面"  width="90%">}}

### 4. Logstash

#### 4.1. インストールと自動起動設定

インストールと自動起動設定は下記コマンドで可能です。

```shell
sudo apt install logstash
```

自動起動設定

```shell
sudo systemctl daemon-reload
sudo systemctl enable logstash.service
```

#### 4.2. Logstashの設定書き換え

Web上の解説サイトでは `/etc/logstash/conf.d/` 配下に `.conf` ファイルを置くと、設定が読み込まれてデータの投入ができると書いているところもありますが、どうやら現在のバージョンでは、自ら明示的に書いてやらないといけないようです。

変更部分はLogstash本体の設定ファイル`/etc/logstash/logstash.yml`内の`path.config:`の部分を以下のように書き換えます。

```shell
sudo nano /etc/logstash/logstash.yml
```

```logstash.yml
# 元の状態
# path.config:
# ↓書き直した後
path.config: /etc/logstash/conf.d
```

## まとめ

今回は、Elasticstackのインストール方法についてまとめました。

今後、自分が持っているテキストデータなどをElasticでインデクシングして、活用していこうと思います。
