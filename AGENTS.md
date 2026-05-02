# AGENTS.md

## このリポジトリについて

- Hugoで構築した個人ブログ（RakuBlog）
- 利用テーマ: [Blowfish](https://blowfish.page/ja/)（Hugoテーマ）
- GitHub Pagesで公開
- ブログURL: [https://rakuichi4817.github.io/](https://rakuichi4817.github.io/)
- ブログ内容: 個人のブログとして、技術的な記事や日常の出来事などを投稿しています
- 記事は主に `content/posts/` 配下で管理（例: `content/posts/2026/05/<slug>/index.md`）
- 記事内で使う画像は、記事と同じディレクトリに配置する（Leaf Bundle運用）

## ルール

- 利用する言語は日本語とする
  - コミットメッセージや記事内容、レビューコメントなどすべて日本語で行う
- `themes/` 配下は編集しない（テーマ本体は直接変更しない）
- テーマのカスタマイズは `layouts/` 配下のオーバーライド、または `assets/css/custom.css` で行う
- サイト全体の設定変更は `config/_default/` 配下を編集する

## コンテンツ追加・更新の手順

- 新規記事は `content/posts/年/月/slug/index.md` を作成して追加する
- Front Matter には最低限 `title` `date` `draft` を設定する
- 公開時は `draft: false` にする
- カテゴリやタグは既存記事の表記揺れに合わせて統一する（日本語表記を維持）
- 画像は記事ディレクトリに配置し、相対パスで参照する（例: `![説明](image.png)`）

## 記事内容のチェックリスト

- 文法と表記に問題がないか
- web文書として読みやすい文書校正になっているか
- 読みやすい流れになっているか
- Markdownが崩れていないか
