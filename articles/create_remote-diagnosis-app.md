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

筆者は普段、サーバーサイドでエンジニアをやっている。
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

オブジェクト指向プログラミングで、
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
筆者は今オブジェクト指向について、改めて詳しく学んでいる最中なので、
説明が不足している可能性がある。
下記の記事などがインタフェースや、オブジェクト指向について詳しく説明してくれているので、
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
