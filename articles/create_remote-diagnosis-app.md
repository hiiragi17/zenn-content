---
title: 'React・Next.js・TypeScriptのキャッチアップのために、荒唐無稽なリモート理由を診断してくれるアプリを作成しました。'
emoji: '🏠'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['react', 'nextjs', 'typescript', 'contest2025ts']
published: false
---

# 始めに

リモートワーク、それは昨今のWebエンジニアにとって、切っても切り離せないだろう。
リモートワークに関して、人によって考え方もバラバラだ。
それなのに、最近リモートワークを否定する企業も多くなった。
推進派だろうと、否定派だろうと、
それでもリモートワークは、
エンジニアにとって1つの手段であるはずだ。
だから、1つの対抗策として、このアプリを作成した。

https://remote-diagnosis-app.vercel.app/

このアプリは~~それっぽい~~リモート理由を診断してくれる
後はその理由を、Slack等で上司に連絡するだけだ。
これで、後は自由気儘なリモートワークへと洒落込めるわけさ。

:::message
**どうかこのアプリが、あなたの快適なリモートワークで役立ちますように。**
:::

:::message alert
**実際に出てきた理由を使って上司に怒られても責任は負えません。悪しからず。**
:::

# 主な使用技術

私は普段、サーバーサイドでエンジニアをやっている。
フロントはほぼ書いたことがない。
今回、フロントのキャッチアップのために、
下記の技術を用いた。
正直こんな~~クソ~~アプリにはオーバースペックだ。

- Next.js 15.1.8
- React 19.0.0
- TypeScript 5系
- Tailwind CSS 3.4.1
- Vercel Analytics 1.5.0
- Vercel OG 0.6.8

GitHubは下記のようになっている。

https://github.com/hiiragi17/remote-diagnosis-app

今回はじめて使った技術だらけすぎて、
学ぶべきことが多かったのだが、
そのなかでピックアップして、学んだことをまとめてみた。

# TypeScriptのインターフェースに関して

## そもそもインターフェースとは

オブジェクト指向プログラミング(OOP)で、
クラスや構造体がプログラム上でどんな処理を持つべきかのルールを定義したもの。

```Java
// Javaでのインターフェース（1995年）
public interface Drawable {
    void draw();
    void resize(int width, int height);
}

public class Circle implements Drawable {
    public void draw() {
        System.out.println("Drawing a circle");
    }
    
    public void resize(int width, int height) {
        // 実装
    }
}
```

Javaがinterfaceキーワードを導入し、現代的なインターフェースの概念を確立した。
その後も下記の言語でインターフェースが導入されている。

- C#（2000年）：Javaとほぼ同様の概念
- PHP（2005年）：PHP 5でインターフェース導入
- TypeScript（2012年）：JavaScriptに型システムとともに導入

従来のOOPにおけるインターフェースの目的は、

- 多重継承の問題解決
- 契約（Contract）の概念
- ポリモーフィズムの実現
の3つだ。

:::message
私は今オブジェクト指向について、改めて詳しく学んでいる最中なので、
説明が不足している可能性がある。
下記の記事などがインタフェースや、
オブジェクト指向について詳しく説明してくれているので、
ぜひ参考にしていただきたい。
:::

https://www.gixo.jp/blog/5159/

https://zenn.dev/cloud_ace/articles/29748ac0537c7f

オブジェクト指向プログラミングにおける「インターフェース」は、
異なるクラス間の通信のルールを定める契約のようなもの。
TypeScriptではそれを「こういう形であるべき」という型の契約書、
型安全性のためのツールとして拡張した。

## TypeScriptでの進化

TypeScriptでのインターフェースは、オブジェクトの「型」を定義する機能。
「このオブジェクトはこういう構造を持っていなければならない」という契約のようなもの。
今回のアプリのHeader部分のコードを見ていく。

```ts:Header.tsx
interface HeaderProps {
  title?: string;
  subtitle?: string;
  showMainTitle?: boolean;
}

export default function Header({ 
  title, 
  subtitle, 
  showMainTitle = true 
}: HeaderProps) {
  return (
    <header className="text-center py-8">
      {showMainTitle && (
        <>
          <h1 className="text-5xl font-bold text-gray-800 mb-4">
            ○○なので、
          </h1>
          <h2 className="text-5xl font-bold text-indigo-600 mb-2">
            リモートします。
          </h2>
        </>
      )}
      
      {title && (
        <h1 className="text-4xl font-bold text-gray-800 mb-2">
          {title}
        </h1>
      )}
      
      {subtitle && (
        <p className="text-lg text-gray-600 italic">
          {subtitle}
        </p>
      )}
      
      {showMainTitle && (
        <p className="text-lg text-gray-600 italic">
          〜 今日のリモート理由診断アプリ 〜
        </p>
      )}
    </header>
  );
}
```

このHeaderPropsインターフェースは、
Headerコンポーネントが受け取るプロパティ（props）の型を定義している。

```typescript
interface HeaderProps {
  title?: string;
  subtitle?: string;
  showMainTitle?: boolean;
}
```

### プロパティの詳細

title?: string - ?アイコンは「オプショナル」を意味し、この値は渡す・渡されないどちらでも良い
subtitle?: string - 同様にオプショナルな文字列
showMainTitle?: boolean - オプショナルなブール値（true/false）

### 実際の使用例

```typescript
export default function Header({ 
  title, 
  subtitle, 
  showMainTitle = true 
}: HeaderProps) {
```

ここでHeaderPropsインターフェースを使って、関数の引数の型を指定している。
これにより：

型安全性：間違った型の値を渡すとエラーになる
補完機能：エディタでプロパティ名が自動補完される
ドキュメント化：どんなプロパティを使えるかが明確になる

TypeScriptが開発時にこれらのエラーを検出してくれるため、バグを未然に防げる。
インターフェースは「型の契約書」として、
コードの安全性と可読性を大幅に向上させる重要な機能。

TypeScriptで定義されたインターフェースは、
コンパイルチェックに活用された後、
JavaScriptコードを生成する過程で消される。
インターフェースがJavaScript実行時に影響することはない。

```typescript
// TypeScriptのコンパイル前
interface User {
  name: string;
  age: number;
}

const user: User = { name: "太郎", age: 25 };

// JavaScriptへのコンパイル後
// インターフェースは完全に消える！
const user = { name: "太郎", age: 25 };
```

# 型推論に関して

TypeScriptで開発者が明示的に型を書かなくても、
コードの文脈から自動的に型を推測する機能。

今回書いたコードだと下記が値する。

```ts:ResultClient.tsx
export default function Header({ 
  title, 
  subtitle, 
  showMainTitle = true  // ← ここで型推論が発生
}: HeaderProps) {
```

もし完全に明示的な書き方をするなら、下記になる。

```ts
export default function Header({ 
  title, 
  subtitle, 
  showMainTitle: boolean = true 
}: HeaderProps) {
  // ...
}
```

この `:boolean`を省略できるのが、型推論だ。
TypeScriptで型推論が働く理由は、
すでに、`HeaderProps`インターフェースでbooleanと定義されていること、
デフォルト値による型推論ができているからだ。

```ts
showMainTitle = true  // trueはboolean型なので、showMainTitleもboolean型と推論
```

TypeScriptの型推論を活用することで、コードがスッキリして可読性が向上する。
今回はそんなにコードを記載していないので、あまりわからないが、
コード量が増えてくると、そのあり難みを感じるに違いない。
とても便利な機能だ。

# Next.jsの学び

ここから先はTypeScriptではなく、Next.jsでの学びを記載していく。

## 動的メタタグ生成でのエラー

まずは下記のコードの例を見て欲しい。

```ts:app/page.tsx
'use client'; // クライアントコンポーネント

import { useState } from 'react';

export async function generateMetadata() {
  return {
    title: 'マイページ'
  };
}

export default function Page() {
  const [count, setCount] = useState(0);
  return <div>カウント: {count}</div>;
}
```

どこがおかしいか、わかるだろうか。
多分大抵の人がわかるだろうが、これは下記のコードが間違っている。

```ts
// ❌ エラー！クライアントコンポーネントではgenerateMetadataは使えない
export async function generateMetadata() {
  return {
    title: 'マイページ'
  };
}
```

初心者がやりがちなエラーらしいが、
私もそのことがわからず、中々エラーから抜け出せなかった。
その理由としてはそもそも、
`use client`という概念を理解できていなかったことが挙げられる。

## まず理解すべき：サーバーサイドとクライアントサイドの違い

### サーバーサイド（Server Side）

```text
┌─────────────┐    HTMLを生成    ┌─────────────┐
│   サーバー   │ ──────────────→ │ ブラウザ     │
│ (Node.js)   │                │ (Chrome等)  │
└─────────────┘                └─────────────┘

```

- **実行場所**: サーバー（Node.js環境）
- **実行タイミング**: ページがリクエストされたとき
- **できること**: データベースアクセス、ファイル操作、環境変数アクセス
- **できないこと**: ブラウザAPI（`window`、`document`、イベントハンドラー）

### クライアントサイド（Client Side）

```text
┌─────────────┐    JavaScript   ┌─────────────┐
│   サーバー   │ ──────────────→ │ ブラウザ     │
│             │                │ (実行される) │
└─────────────┘                └─────────────┘

```

- **実行場所**: ブラウザ
- **実行タイミング**: ページが読み込まれた後
- **できること**: ブラウザAPI、ユーザーインタラクション、状態管理
- **できないこと**: サーバー専用のAPI、環境変数アクセス

そして`use client;`は「このコンポーネントはブラウザで実行してください」という指示だ。

ここで改めてメタタグはいつ生成されていないといけないだろうか？

## メタタグ生成の実行タイミング

メタタグ生成は**サーバーサイド**で実行される必要がある。
データの流れとしては、
URL → サーバーコンポーネント → メタタグ生成 → HTML送信 → クライアントコンポーネント実行
となる。

`use client;`のファイルでは`generateMetadata`を定義できず、
仮にクライアントサイドでタイトルを変更しても、
初期のHTMLには正しいメタタグが含まれていないことになってしまう。

ブラウザに送信 → SEOクローラーが正しいメタタグを読み取る
という流れになるため、サーバーサイドでのメタタグ生成は
SEO対策において必須となる。

そのため今回の実装では、結果ページに`generateMetadata`を置き、
use clientコンポーネントを、ResultClient.tsxに分離した。
そのためうまく動的なメタタグを作成できるようになった。

```ts:page.tsx
import type { Metadata } from 'next';
import { Suspense } from 'react';
import ResultClient from './ResultClient';

type Props = {
  searchParams: Promise<{ result?: string }>
}

export async function generateMetadata({ searchParams }: Props): Promise<Metadata> {
  const params = await searchParams;
  const result = params.result || '○○なので';
  const title = `${result}、リモートします。`;
  const description = '今日のリモート理由を診断するアプリ';
  const ogImageUrl = `/api/og?result=${encodeURIComponent(result)}`;
```

```ts:ResultClient.tsx
'use client';

import { useRouter } from 'next/navigation';
import { useEffect } from 'react';
import Header from '../components/Header';
import Footer from '../components/Footer';

interface ResultClientProps {
  result: string | undefined;
}

export default function ResultClient({ result }: ResultClientProps) {
  const router = useRouter();

  // 動的にページタイトルを変更
  useEffect(() => {
    if (result) {
      document.title = `${result}、リモートします。`;
    }
  }, [result]);

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
      <Header 
        title="診断結果"
        subtitle="〜 今日のあなたのリモート理由 〜"
        showMainTitle={false}
      />

      {/* メインコンテンツ */}
      <main className="flex flex-col items-center justify-center px-4">
        {/* 結果表示カード */}
        <div className="bg-white rounded-xl shadow-lg p-8 max-w-md w-full mb-8">
          <div className="text-center">
            {/* 結果アイコン */}
            <div className="text-6xl mb-6">🎉</div>
            
            {result && (
              <>
                {/* 結果テキスト */}
                <div className="bg-indigo-50 rounded-lg p-6 mb-6">
                  <p className="text-3xl font-bold text-indigo-800 mb-2">
                    {result}、
                  </p>
                  <p className="text-3xl font-bold text-gray-800">
                    リモートします。
                  </p>
                </div>

                {/* 励ましメッセージ */}
                <div className="mb-6">
                  <p className="text-gray-600">
                    素晴らしい理由ですね！✨
                    <br />
                    今日も快適なリモートワークを
                    <br />
                    お楽しみください 🏠
                  </p>
                </div>
              </>
            )}

            {/* シェアボタン */}
            {result && (
              <div className="mb-2">
                <button
                  onClick={() => {
                    const text = `${result}、リモートします。`;
                    const hashtag = '#OOなのでリモートします。';
                    const url = window.location.href;
                    const tweetText = `${text} ${hashtag}`;
                    const tweetUrl = `https://twitter.com/intent/tweet?text=${encodeURIComponent(tweetText)}&url=${encodeURIComponent(url)}`;
                    window.open(tweetUrl, '_blank');
                  }}
                  className="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg transition-colors duration-200"
                >
                  🐦 Xでシェアする
                </button>
              </div>
            )}

            {/* ボタンエリア */}
            <div className="space-y-2">
              <button
                onClick={() => router.push('/')}
                className="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg transition-colors duration-200"
              >
                🏠 TOPページに戻る
              </button>
            </div>
          </div>
        </div>

        {/* 追加メッセージ */}
        <div className="max-w-md text-center">
          <p className="text-gray-600 text-sm mb-4">
            貴方にぴったりな理由が出るまで診断してみてくださいね！
          </p>
          <p className="text-xs text-gray-500">
            ※この理由はランダムに生成されています
          </p>
        </div>
      </main>

      <Footer />
    </div>
  );
}
```

### 参考：何ができる・できないの比較表

| 機能 | サーバーコンポーネント | クライアントコンポーネント |
| --- | --- | --- |
| `useState`、`useEffect` | ❌ | ✅ |
| イベントハンドラー（`onClick`等） | ❌ | ✅ |
| ブラウザAPI（`window`、`localStorage`） | ❌ | ✅ |
| データベースアクセス | ✅ | ❌ |
| 環境変数（サーバー専用） | ✅ | ❌ |
| `generateMetadata` | ✅ | ❌ |

よく聞いてみると、当たり前のことを言っているだけだが、
サーバーサイドの言語を書いていると、
サーバーサイドであることが当たり前なので。
Next.jsの2つの世界に関して、
何となくの理解で進めてしまい、酷い目にあったのだろう。
もし単純な性的サイトであれば、メタタグは固定で良いが。
動的コンテンツ + SNSシェア + (SEO)の場合は、
サーバーサイドでの動的メタタグ生成が、必須であることが学べてよかった。

# 最後に

そんなに難しくないアプリではあるが、
基本的な内容を学びには良い経験になった。
今後もこの技術スタックで、色々なアプリを作成してみたい。
それではここまで読んでいただきありがとうございました。
誰かの参考になれば幸いです。

# 参考文献

この記事は以下の記事を参考にして執筆させていただきました。

https://www.gixo.jp/blog/5159/

https://zenn.dev/cloud_ace/articles/29748ac0537c7f

https://typescriptbook.jp/reference/object-oriented/interface

https://typescriptbook.jp/

https://typescriptbook.jp/reference/values-types-variables/type-inference
