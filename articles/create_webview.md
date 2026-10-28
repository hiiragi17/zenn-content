---
title: "ReactNativeでWebviewを導入する。"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['reactnative']
published: false
---

# はじめに

React Nativeアプリ開発で「Webページを表示したい」と思ったことはありませんか？
今回は**WebViewの仕組みを理解する**ため、GitHubのログインページを例に表示してみます。

## WebViewとは？

WebViewとはスマートフォンのネイティブアプリ（iOS/Android OS）独自の統一ブラウザで、Google ChromeやSafariなどとは違う仕様でWebのデータを処理し、
ページを表示します。
**「アプリの中にミニブラウザを埋め込む」** というイメージです。

ReactNativeには、
**react-native-webview**
というものがあるので、これを使えば簡単にWebviewを使うことができます！

https://github.com/react-native-webview/react-native-webview

### WebViewでできること

- 外部Webサイトの表示
- 利用規約やプライバシーポリシーの表示
- Webベースのグラフやチャートの埋め込み

既存のWebページを開く際に自社アプリから離れてChromeやSafari（別のアプリ）を起動するよりも、
アプリ内ブラウザでそのまま同じアプリに滞在してもらうほうが滞在時間が伸び、
購買行動につながりやすいというメリットがあります。
ECサイトなどでよく使われています。

## 実践：GitHubのページを表示してみる

※ReactNativeのセットアップがされている、
シュミレーターがインストールされていることが前提条件になります。

### Step 1: インストール

```bash
npm install react-native-webview
```

### Step 2: 基本的な実装

- LoginFormコンポーネント(Webviewの設定)

```javascript
import React, { useRef, useState } from 'react';
import { View } from 'react-native';
import { WebView } from 'react-native-webview';

interface LoginFormProps {
  onLoginSuccess?: () => void;
  onClose?: () => void;
}

export const LoginForm: React.FC = ({
  onLoginSuccess,
  onClose,
}) => {
  const [isError, setIsError] = useState(false);

  // URLが変更されたときに呼ばれる
  const handleWebViewNavigationStateChange = (navState: any) => {
    console.log('現在のURL:', navState.url);
    
    // ログイン成功判定：特定のURLにリダイレクトされたか確認
    if (navState.url.includes('dashboard') || navState.url.includes('success')) {
      // ログイン成功！
      onLoginSuccess?.();
    }
  };

  return (
    
      <WebView
        source={{
          uri: 'https://github.com/login',
        }}
        startInLoadingState={true}
        onError={() => setIsError(true)}
        onNavigationStateChange={handleWebViewNavigationStateChange}
      />
    
  );
};
```

- ホーム画面（ログイン状態の管理）

```javascript
import { View, StyleSheet } from 'react-native';
import { LoginForm } from '@/components/ui/login-form';
import { useState } from 'react';

export default function HomeScreen() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  // ログイン完了後は別の画面を表示
  if (isLoggedIn) {
    return (
        {/* ログイン後のコンテンツをここに表示 */}
    );
  }

  // ログイン前はWebViewでGitHubログインページを表示
  return (
    
      <LoginForm
        onLoginSuccess={() => {
          console.log('ログイン成功！');
          setIsLoggedIn(true);
        }}
        onClose={() => {
          console.log('ログインをキャンセル');
        }}
      />
    
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  successContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  placeholder: {
    flex: 1,
  },
});
```

それだけで下記のようにログインページが表示されるようになります。

![GitHubログインページ](/images/webview/github_login.png)

## まとめ

WebViewは「アプリの中の小さなブラウザ」として、
適切に使えば開発を効率化できる便利な機能です。
ただし、セキュリティが重要な機能には使用せず、
適材適所で活用しましょう！

## 参考リンク

この記事は下記を参考にして書かせていただきました。

- [react-native-webview公式ドキュメント](https://github.com/react-native-webview/react-native-webview)

- [WebViewとは？　メリットとブラウザアプリとの違いをわかりやすく解説](https://backapp.co.jp/blog/1505)
