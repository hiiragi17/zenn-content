---
title: " Astro × TypeScript × GitHub Pagesで1日1スクラップサイトを構築してみた。"
emoji: "📃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['astro', 'typescript', 'GitHubPages']
published: false
---
# 始めに

皆さまは日々の小さなアウトプットをどのように行なっていますか。
毎日ちゃんとブログ記事を書いたりしていますか？
もしくはTwitter(現X)に書いたりしていますか？
Times文化があればそこに書いたりしていますか？
NotionやZennにまとめて書いたりしていますか？

折角なら何か学んだことをどこかにまとめたりしたいと考える人は多いでしょう。
しかし、Zennのような技術情報共有サービスに投稿するための準備は大変です。
そうではなく、サクッと自分だけがわかるような、
小さなアウトプットがしたいと考えていました。
そしてできるのならGitHubに草を生やせた方が、
モチベーションアップにも繋がります。
最初はZennのスクラップ機能をGitHubで管理できるなら、
やってみようと考えましたが、
スクラップはGitHub管理できないことを知り断念しました。
その後はこのTILをやってみようかと考えました。

https://qiita.com/nemui_/items/239335b4ed0c3c797add

しかしさらに調べると、
GitHub Pagesを使って技術メモを公開する方法を紹介しているページを見つけました。

https://karaage.hatenadiary.jp/entry/2017/09/06/073000

これが一番エンジニアっぽいのでは!?(安直)と考え、
GitHub Pagesで1日1スクラップサイトを作ってみることにしました。

# Astroを選んだ理由

1日1スクラップサイトの方向性としては、
「GitHub Pages + Markdown + 静的サイトジェネレーター」でした。
静的サイトジェネレーターには色々あるのですが、
今回はAstroを選びました。
TypeScriptで色々管理できることに魅力を感じたからです。
またAstroはブログのような静的サイトを作成することに特化しており、
Markdownを直書きOKかつ軽量、高速であることに惹かれました。

https://docs.astro.build/ja/getting-started/

> Astro は、速度を重視して設計されたオールインワン Web フレームワークです。お気に入りの UI コンポーネントとライブラリを利用して、どこからでもコンテンツを取得し、どこにでも展開できます。

# AstroのContent Collections APIの仕組み

## Content Collections APIとは

Content Collectionsは、
AstroでMarkdownやMDXファイルを型安全に管理するための仕組みです。
従来のファイルベースアプローチと異なり、スキーマ定義により厳密な型チェックが可能です。

### 基本的な仕組み

ディレクトリ構造

```textile
src/
├── content/
│   ├── config.ts          # スキーマ定義
│   └── blog/              # コレクション
│       ├── first-post.md
│       ├── second-post.md
│       └── ...
```

スキーマ定義

```typescript:config.ts
import { defineCollection, z } from 'astro:content';

// 'blog' という名前のコレクションを定義
const blogCollection = defineCollection({
  type: 'content',  // Markdownコンテンツ
  schema: z.object({ // フロントマターの型定義(Zodを使用)
    title: z.string(),
    description: z.string(),
    pubDate: z.date(),
    tags: z.array(z.string()).optional(), // 文字列の配列(オプショナル)
    draft: z.boolean().default(false),
  }),
});

export const collections = {
  blog: blogCollection,
};
```

- defineCollectionでblogという名前のコレクションを定義する。
- schemaで、各記事のMarkdownファイルに記述するフロントマターの項目
（titleやpubDateなど）とその型をzodを使って指定する。
これにより、型が間違っているとエラーで教えてくれる。

## getCollection()の内部動作

```typescript
import { getCollection } from 'astro:content';

const posts = await getCollection('blog');
```

- getCollection('blog')でblogコレクションの記事データをすべて取得する。

まとめるとContent Collections APIにより、下記の効果が得られます。

**開発時**

- 型安全性: スキーマベースの厳密な型チェック
- 自動補完: エディターでの開発効率向上
- エラー防止: 存在しないプロパティへのアクセスを防止

**ビルド時**

- スキーマ検証: 無効なデータでビルドが失敗
- 一貫性保証: すべてのコンテンツが同じ構造を持つことを保証

**実行時**

- パフォーマンス: 事前に検証されたデータの高速アクセス
- 信頼性: 型安全性により実行時エラーを防止

今後記事が増えることを考えると、
エラー防止になり、パフォーマンスが高いというのは、
とても魅力的です。

## GitHub Actionsでの自動デプロイ

下記のようにGitHub Actionsで自動デプロイを行なえるようにしています。

```YAML:deploy.yml
# name: ワークフローの名前（GitHub のActionsタブで表示される）
name: Deploy to GitHub Pages

# on: ワークフローのトリガーを定義
# push: コードがプッシュされたときに実行
# branches: ["main"]: mainブランチへのプッシュのみを対象
on:
  push:
    branches: ["main"]

# Job1:build(ビルドジョブ)
jobs:
  build:
    # GitHub が提供する仮想環境（Ubuntu Linux）を使用 
    # 他の選択肢：windows-latest, macos-latest
    runs-on: ubuntu-latest
    steps:
      # 公式アクションを使用してリポジトリのコードを仮想環境にダウンロード 
      # @v4は使用するアクションのバージョン
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          # Node.js version 20をインストール
          # Astroは Node.js 18+ が必要なので、最新の安定版を使用
          node-version: "20"
      #  依存関係のインストール
      - run: npm ci
      #  Astroサイトのビルド
        # package.jsonのscriptsセクションで定義されたbuildコマンドを実行
        # Astroの場合、通常はastro buildが実行される
      - run: npm run build
      # ビルドで生成された./distフォルダをアーティファクトとして保存
      # 次のジョブ（deploy）で使用するために一時保存
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  # Job2:deploy(デプロイジョブ)
  deploy:
    # このジョブはbuildジョブが成功した後に実行される
    needs: build
    runs-on: ubuntu-latest
    steps:
      # GitHub Pages専用の公式デプロイアクション
      # 前のジョブでアップロードされたアーティファクトを自動で取得し、GitHub Pagesにデプロイ
      - uses: actions/deploy-pages@v4
```

ポイントは、buildとdeployを分離していることです。
これにより、ビルドの成功を確認してからデプロイが実行されます。

そして完成したサイトがこちらです。

https://hiiragi17.github.io/daily-scraps/

# 余談

このブログを作るのに時間がかかってしまって、
まだそんなにアウトプットができていないので、
少し本末転倒感があるのですが。
多分もう少し早く作りたかったら、
Jekyllなどを使用する方が早いでしょう。
下記のサイトなどとても参考になります。
テーマも選べるのでブログっぽくなります。

https://qiita.com/madoreenu/items/b47833bf785562c77819

:::message
Astroでもテーマは選べます。
今回私が使用しなかっただけです。
:::

https://astro.build/themes/?search=

# 最後に

これから小さなアウトプットを毎日する習慣を付けられそうで楽しみです。
もう少しカスタマイズしたいなとは考えますが。
それではここまで読んでいただきありがとうございました。
誰かの参考になれば幸いです。

# 参考文献

この記事は以下の記事を参考にして執筆させていただきました。

https://docs.astro.build/ja/getting-started/