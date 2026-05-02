---
title: "飲食店レビューからポエムを撲滅！OpenAI の Structured Outputを使ったテキスト抽出"
date: 2024-11-22
description: "Structured Outputを使って、飲食店のレビューから料理と関係のないポエミーな内容を抽出するプログラムを紹介します。"
summary: "Structured Outputを使って、飲食店のレビューから料理と関係のないポエミーな内容を抽出するプログラムを紹介します。"
draft: false
heroStyle: "simple"
categories:
 - テック系
tags:
 - 生成AI
 - LLM
 - OpenAI
 - Python
---

**「人はなぜレビューを書くのだろう。感動を伝えたいから？それとも言葉で遊びたいから？私たちがレビューに求めるもの、それは詩ではない。確かな味、確かな情報、ただそれだけなのに。」**

とポエミーな表現で書き出してみました。皆さんも、食べログなどでレビューを見ていると、このようなポエミーな文章を見かけることがあるのではないでしょうか。

わたしは学生時代に食べログレビューを扱った研究をしていたのですが、目的に合わないポエムのような文章がノイズになることが多く、前処理に苦戦した覚えがあります。

ということで、飲食店のレビューからポエムを撲滅するために、OpenAIが提供しているStructured Outputを利用してテキスト抽出をの課題に取り組んでみます。（というよりは、Structured Outputの練習をしてみます）

ただ、公式のドキュメントが充実しているので細かい解説は省略します。めんどくさいので...。

## Structured Outputとは

Structured Outputは、OpenAIが提供するLLMの回答を特定のフォーマットや構造に従って提供するための機能です。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://platform.openai.com" data-iframely-url="//iframely.net/WYepPOj?card=small"></a></div></div><script async src="//iframely.net/embed.js"></script>

例えば「兵庫と大阪の県庁所在地を教えて」といった質問に対して、標準のチャット応答であれば「箇条書きで出力して」と付け加えることで、以下のような形式で出力させることができるでしょう。

```text
- 兵庫県: 神戸市
- 大阪府: 大阪市
```

Structured Outputを使うことで、以下のように構造化されたフォーマットで回答を得ることができます。

```json
[
    {
        "prefecture": "兵庫県",
        "capital": "神戸市"
    },
    {
        "prefecture": "大阪府",
        "capital": "大阪市"
    }
]
```

LLMをシステムに組み込むことを考えると、回答のフォーマットを統一できることは、とてもとてもうれしいです。どんな出力が返ってくるか決まっていないのに、プログラムで処理するのは難しいですからね。この機能は、現在（2024年11月22日現在） `gpt-4o` と `gpt-4o-mini` で利用可能なようです。

## ポエム抽出という課題でStructured Outputを試してみる

### 抽出する内容

今回は以下のテキストを入力として、Structured Outputを使ってポエムを抽出してみます。こちらのテキストはステーキ屋を想定してLLMで生成したレビュー文章です。

とてもポエミーな文章ですが、実際にこういうレビューはあるんですよね...。

> ある日、仕事に疲れた帰り道、「自分へのご褒美」が必要だと感じました。スマホで偶然見つけたこのステーキ屋さん。その写真越しに伝わる肉の輝きに惹かれ、気づけば予約ボタンを押していました。扉を開けた瞬間のあの香り――それは、疲れた心に優しく語りかけるような芳醇な肉の誘い。まるで、「今日という日をよく頑張ったね」と囁いてくれるかのようでした。
>
> 店内は、落ち着いた木の香りが漂う癒しの空間。壁には地元のアーティストが描いたらしい抽象画がかかり、どこか「静かなる情熱」を感じさせるようなインテリア。窓の外には小さな庭が広がり、ライトに照らされた植物たちが夜の静寂を美しく彩っていました。
>
>「この空間は、料理を味わうためだけに存在しているんだな」と思わせるような、静かでありながらも温かい雰囲気。
>
> 頼んだのはお店の一番人気、特選サーロインステーキ200g。運ばれてきた瞬間、肉の表面で輝く美しい焼き加減に目を奪われます。ナイフを入れると、予想以上に柔らかく滑らかに切れていき、そこから溢れる肉汁に思わず息をのみました。
>
> ナイフを入れた瞬間、肉の繊維がするするとほぐれていくのが見て取れます。一口頬張ると、まるで「肉の小宇宙」が広がるかのような感覚。噛むたびに、肉汁とともに溢れ出る深い旨味と甘み。それは、ただの食事ではなく、一種の芸術体験に近いものでした。付け合わせのポテトグラタンも絶品。ほのかに香るチーズが、肉の重厚さを優しく受け止め、次の一口をさらに待ち遠しいものにしてくれます。
>
> スタッフの方々の心遣いもまた感動的でした。「お肉がこの温かさで一番美味しいタイミングです」という一言に込められたプロフェッショナルの矜持。それが、食事全体の信頼感をさらに高めてくれました。
>
> 総評
>
> ただの食事ではなく、心を満たすひとときを提供してくれる、そんなステーキ屋さんでした。このお店を訪れた日を、私の「食の記憶」に残る大切な1ページとして忘れることはないでしょう。また、あの魔法の一皿に会いに行きたいと思います。

自分語りしている内容はもちろん、全体的にかなり詩的な表現をしていることがわかります。

このテキストから**料理以外のポエム的な表現を抽出してみます**。「ナイフを入れた瞬間、肉の繊維がするするとほぐれていくのが見て取れます。」などの料理に関する内容は抽出対象外とします。料理のおいしさを知りたい場合、この内容は非常に有用な情報ですからね。

### コード全容

早速ですが、作成したプログラムを一気にドンと。今回はAzure OpenAI経由で利用しています。

```python
import os
from typing import Literal

from openai import AzureOpenAI
from openai.types.chat import (
    ChatCompletionMessageParam,
    ChatCompletionSystemMessageParam,
    ChatCompletionUserMessageParam,
)
from pydantic import BaseModel, Field
from pydantic_settings import BaseSettings, SettingsConfigDict

SYSTEM_MESSAGE = """
あなたは飲食店のレビューから、ポエム的な内容を抽出するAIアシスタントです。

与えられるテキスト情報は、飲食店に対してその飲食店を訪れたユーザが投稿したレビューです。
レビューの内容は、飲食店の評価や感想、食事内容、サービス内容などが含まれます。
しかし、その中には、飲食店の評価とは直接関係のない、投稿者の背景や昔話、詩的な表現が含まれることがあります。
食べた料理に対して詩的な表現がされている場合、それはポエムではありますが有用な情報なので抽出はしないでください。
"""


class Settings(BaseSettings):
    # 各種設定値を読み込む
    model_config = SettingsConfigDict(
        env_file=os.path.join(os.path.dirname(__file__), ".env")
    )

    aoai_endpoint: str = ""
    aoai_api_version: str = ""
    aoai_api_key: str = ""
    deployment_name: str = "gpt-4o"


class PoemMeassage(BaseModel):
    """抽出されたポエムの情報"""

    sentence: str = Field(title="テキストに含まれていたポエム的な内容")
    reason: str = Field(title="ポエムと判断された理由")
    sentence_type: Literal["自分のこと", "お店のこと"] = Field(
        title="内容が自分の話か、お店の話かを判定してください"
    )


class ExtractPoemResponse(BaseModel):
    """テキストに含まれるポエムのリスト"""

    poem_messages: list[PoemMeassage] = Field(title="抽出されたポエムのリスト")


settings = Settings()


openai_client = AzureOpenAI(
    azure_endpoint=settings.aoai_endpoint,
    api_version=settings.aoai_api_version,
    api_key=settings.aoai_api_key,
)


def extract_poems_from_review(review: str) -> list[PoemMeassage]:
    """レビューテキストからポエムを抽出する

    Args:
        review (str): 抽出対象のレビューテキスト

    Returns:
        list[PoemMeassage]: 抽出されたポエムリスト
    """

    system_message = ChatCompletionSystemMessageParam(
        role="system", content=SYSTEM_MESSAGE
    )
    query = ChatCompletionUserMessageParam(role="user", content=review)
    messages: list[ChatCompletionMessageParam] = [system_message, query]

    response = openai_client.beta.chat.completions.parse(
        model=settings.deployment_name,
        messages=messages,
        temperature=0,
        response_format=ExtractPoemResponse,
    )

    parse_sentence_result = response.choices[0].message.parsed

    if parse_sentence_result is None:
        raise ValueError("ポエムの抽出に失敗しました")
    return parse_sentence_result.poem_messages
```

### フォーマットの指定

Azure OpenAIの接続情報などを定義している部分は説明を割愛します。

Python経由でStructured Outputを利用する際には、[Pydantic](https://docs.pydantic.dev/latest/)を使って受け取りたいフォーマットを指定します。

Pydanticで定義したオブジェクトスキーマを`openai_client.beta.chat.completions.parse()`の`response_format`引数に指定することで、指定したフォーマットで結果を受け取ることができます。

今回は以下のようにスキーマを定義しました。 各属性の型ヒントや、`Field` 内の引数 `title` の内容に従って回答を出力してくれるようになります。`sentence_type` のように、あらかじめ用意した選択肢（ `typing.Literal` ）から出力を選ばせることもできます。

```python
class PoemMeassage(BaseModel):
    """抽出されたポエムの情報"""

    sentence: str = Field(title="テキストに含まれていたポエム的な内容")
    reason: str = Field(title="ポエムと判断された理由")
    sentence_type: Literal["自分のこと", "お店のこと"] = Field(
        title="内容が自分の話か、お店の話かを判定してください"
    )


class ExtractPoemResponse(BaseModel):
    """テキストに含まれるポエムのリスト"""

    poem_messages: list[PoemMeassage] = Field(title="抽出されたポエムのリスト")
```

このように指定することで、LLMからの回答をPythonオブジェクトに変換して受け取ることができるので非常に便利です。また、出力させたい内容を明確にすることができるのもメリットの1つかと思います。

### システムメッセージについて

システムメッセージは割と適当に作っていますが、**料理のことに関してはポエミーに書いても抽出しないように注意書きを入れています**。

出力する内容に関しては、Pydanticのフォーマットに従って出力されるため、システムメッセージで細かく指定する必要はない印象です。（どちらに丁寧な説明を書くかは悩みポイントです）

### `openai.types.chat` を使って型をはっきりさせる

`openai.types.chat` には、チャットのメッセージを表すためのクラスが用意されています。これを使うことで、どのロールで送るメッセージかをはっきりさせることができます。

意外とWeb上の記事などでは、辞書型でそのまま突っ込んでる方が多いのです。しかし、**可読性の観点から、個人的には以下のように書くことをオススメしています。**

```python

from openai.types.chat import (
    ChatCompletionSystemMessageParam,
    ChatCompletionUserMessageParam,
)

system_message = ChatCompletionSystemMessageParam(
    role="system", content=SYSTEM_MESSAGE
)
query = ChatCompletionUserMessageParam(role="user", content=review)
```

### 抽出結果

実際にリクエストを投げるところは難しくないので、実際に`extract_poems_from_review()`を利用して、上記のテキストから抽出した結果を見てみます。

```python
[
    PoemMeassage(
        sentence="ある日、仕事に疲れた帰り道、「自分へのご褒美」が必要だと感じました。",
        reason="自分の感情や状況を詩的に表現しているため。",
        sentence_type="自分のこと",
    ),
    PoemMeassage(
        sentence="扉を開けた瞬間のあの香り――それは、疲れた心に優しく語りかけるような芳醇な肉の誘い。まるで、「今日という日をよく頑張ったね」と囁いてくれるかのようでした。",
        reason="香りを擬人化し、感情に訴えかける表現が詩的であるため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="店内は、落ち着いた木の香りが漂う癒しの空間。壁には地元のアーティストが描いたらしい抽象画がかかり、どこか「静かなる情熱」を感じさせるようなインテリア。",
        reason="店内の雰囲気を詩的に表現しているため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="窓の外には小さな庭が広がり、ライトに照らされた植物たちが夜の静寂を美しく彩っていました。",
        reason="景色を詩的に描写しているため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="「この空間は、料理を味わうためだけに存在しているんだな」と思わせるような、静かでありながらも温かい雰囲気。",
        reason="空間の雰囲気を詩的に表現しているため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="まるで「肉の小宇宙」が広がるかのような感覚。",
        reason="料理の体験を詩的に表現しているため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="それは、ただの食事ではなく、一種の芸術体験に近いものでした。",
        reason="食事を芸術に例えて詩的に表現しているため。",
        sentence_type="お店のこと",
    ),
    PoemMeassage(
        sentence="このお店を訪れた日を、私の「食の記憶」に残る大切な1ページとして忘れることはないでしょう。",
        reason="訪問の体験を詩的に表現しているため。",
        sentence_type="自分のこと",
    ),
]

```

結構いい感じでとれてそうですね。ポエム的な表現が抽出されていることがわかります。スキーマに従って、対象の文章、選ばれた理由、自分のことを書いているのか、お店のことを書いているのかが出力されています。

[フォーマットの指定」でも触れましたが、Structured Outputはフォーマットの強制以外にも、出力する内容を制御しやすいところがメリットだと思っています。通常のメッセージ回答だと、出力してほしい内容をシステムメッセージに詰めこんだ際に求める情報が落ちることが結構ある印象でした。

## まとめ

今回は、OpenAIのStructured Outputを使って、飲食店のレビューからポエムを抽出するプログラムを紹介しました。半分遊びで試してみましたが、なかなか面白い結果が出たのでもう少し遊んでみようかなと思いました。

ちなみに、o1モデルはまだStructured Outputに対応していないので、対応されるのが楽しみです。
