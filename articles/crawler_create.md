---
title: 'pyppeteerを使用してクローラーを作成した話。'
emoji: '🤖'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['Python', 'pyppeteer', 'クローラー']
published: false
---

# 始めに

今回業務でのとある案件で、Pythonによるクローラー作成を行いました。元々スクレイピングは個人開発の時にしたことがあったのですが、クローラーに関してはあまり知らないことも多かったため、折角なのでまとめてみたいと思います。

# そもそもクローラーとは？

> Web クローラーは、Web サイトを体系的に検索し、そのコンテンツをインデックス登録する自動化されたプログラムまたはボットです。主に検索エンジン向けにページをインデックス登録するために使用されるほか、クーポンや比較ショッピングアプリ、SEO や RSS アグリゲーションなどにも使用されます。

参考サイト：

https://www.akamai.com/ja/glossary/what-is-a-web-crawler

つい最近輪読会にも参加している、「システム設計の面接試験」の9章にも出てきており、ロボットやスパイダーとしても知られているらしいです。こちらの本では、大量のページをクロールし、情報をどのように保存するかの設計が問われており、重複したページをどのようにするかなど、かなり細かい設計まで問われていましたが。今回私が実装したクローラーは、とあるサイトの写真のURLとリンク先のURLを保存するような、シンプルなものでした。

ともかくインターネット上でウェブページを見て、情報を保存するようなものをクローラーと定義して良いのでしょう。

# 実際の実装に関して

## 元々はBeautifulSoupのみで実装されていた

元々他にクローラーが作成されており、最初はそれを見ながら実装していました。その時使用されていたのがBeautifulSoupでした。

> Beautiful SopuはHTMLやXMLから狙ったデータを抽出するためのライブラリです。公式ドキュメントの冒頭の説明を見るとこれはHTMLやXMLのパーサーそのものではなく、パーサーをラップして扱いやすくするライブラリのようです。

参照： https://qiita.com/Chanmoro/items/db51658b073acddea4ac

Beautiful Soupは前述の通りHTMLコンテンツなら、特に問題なくパース処理を行えます。しかしJavaScript等で作成された、動的なコンテンツの場合、これだけではうまくいきません。最後に表示されるHTMLとダウンロードされるHTMLが別物になってしまうからです。最初このことになかなか気が付かなかったのですが、下記のようにHTML構造をログに出力させて、ようやく分かりました。

```python
# ページ全体のHTML構造をログに出力
with open("page_structure.html", "w", encoding="utf-8") as f:
    f.write(str(soup))
```

## requests_htmlを使用しようとしたが、上手くいかなかった

この問題を解決するために、SeleniumやPlaywrightなどのヘッドレスブラウザを使用して、JavaScript実行後のHTMLを取得する方法がありました。

https://zenn.dev/aiq_dev/articles/b13a88fe4a7bc7

https://qiita.com/tomomi-kawashita/items/1a1e03d5ee590823b92e

ただ設定するためにDockerコンテナに、

- Chrome (またはChromium) ブラウザ
- ChromeDriverなどのコンポーネントが必要であり、少し大変そうだなと思い、他に方法がないか検索してrequests-html ライブラリというのを見つけました。

> requests-htmlは以下のように、requestやBeautifulSoup(bs4)に依存したライブラリです。つまり、内部でこれらのライブラリを利用しています。なかでもpyppeteerにより「インターフェースなしのブラウザ（ヘッドレス）」を使うことでブラウザの表示内容を取得できます。

参照 https://gammasoft.jp/blog/how-to-download-web-page-created-javascript/

しかしこちらの記事のようなエラーが頻発し、

https://absg.hatenablog.com/entry/2020/10/20/201926

ヘッドレスブラウザの依存ライブラリをインストールするようDockerファイルに記載したのですが、結局メモリ不足になったりして上手くいきませんでした。

## pyppeteerを使用したところ上手くいった！

そこで既にrequests-htmlに使用されていた、pyppeteerに着目しました。

> Puppeteer は Chromium をヘッドレスモードでプログラムが操作することを目的としたパッケージで、pyppeteer とは、 puppeteer (パッペッティア: 人形遣い) と呼ばれるGoogle製のJavaScriptのパッケージを、Pythonに移植したものです。

https://zenn.dev/mook_jp/articles/pyppeteer-simple-tutorial

さらにこの記事を参考に、pyppeteerを使用して、動的コンテンツを含むHTMLを取得し、BeautifulSopuを利用して、HTMLをパースするようにしました。

https://zenn.dev/haruka/articles/9ada7e0236463a

その後は必要な情報を抽出し、データを保管した後、DBにbulk insertするだけでした。

## Dockerイメージによるもの？

今回作成した、pyppeterr用に分離した、Dockerfile.pyppeterrは下記のように、alpineを使用したものでした。

```py
FROM python:3.12-alpine

RUN apk update && apk --no-cache add \
    chromium \
    chromium-chromedriver \
    nss \
    freetype \
    harfbuzz \
    ca-certificates \
    ttf-freefont \
    jpeg-dev \
    zlib-dev \
    libstdc++ \
    && rm -rf /var/cache/apk/*

RUN which chromium || echo "chromium not found" && \
    which chromium-browser || echo "chromium-browser not found" && \
    ls -la /usr/bin/chrom* || echo "No chromium binaries found in /usr/bin"

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PYPPETEER_NO_SANDBOX=1

ENV PYPPETEER_CHROMIUM_EXECUTABLE=/usr/bin/chromium

WORKDIR /usr/local/app

COPY src/requirements.txt /usr/local/app/

RUN apk --no-cache add build-base libffi-dev gcc musl-dev && \
    python -m pip install --upgrade pip && \
    python -m pip install -r /usr/local/app/requirements.txt && \
    python -m pip install pyppeteer requests-html lxml beautifulsoup4 html5lib && \
    python -m pip cache purge && \
    apk del build-base

COPY src .

CMD ["python", "main.py"]
```

> -alpineがついたimageは、Alpine Linuxをベースに構築されたものです。Alpine Linuxは、コンテナで利用することを想定して設計されたOSで、極めて軽量です。Alpineのベースイメージは5MB未満と非常に小さく収まっています。コンテナを可能な限り最小で、最速で構築したい場合に採用します。

参照： https://www.ted027.com/post/docker-debian-difference/

今回の案件では元々この仕様のDockerfileだったため、特に変更しなかったのですが、この仕様のために、もしかしたらrequests_htmlでメモリが足りなかったり、互換性を持つパッケージがなくて上手く動作しなかった可能性もあるなと思いました。

こういう記事もあるぐらいなのですが、

https://engineering.nifty.co.jp/blog/26586

まあ案件で上手いこと使えているし、多分考えがあるからalpineを使用していると思うのですが。むしろDockerイメージに色々なものがあることを、あまり理解してなかったので、いい勉強になりました。

# まとめ

クローラーを初めて実装したため、色々躓くところはあったのですが、知らなかったことを色々知れたため、自分の備忘録としてまとめておきました。誰かの参考になれば幸いです。

# 参考文献

この記事は下記の記事を参考にして、書かせていただきました。

https://www.akamai.com/ja/glossary/what-is-a-web-crawler

https://qiita.com/Chanmoro/items/db51658b073acddea4ac

https://zenn.dev/aiq_dev/articles/b13a88fe4a7bc7

https://qiita.com/tomomi-kawashita/items/1a1e03d5ee590823b92e

https://gammasoft.jp/blog/how-to-download-web-page-created-javascript/

https://zenn.dev/mook_jp/articles/pyppeteer-simple-tutorial

https://zenn.dev/haruka/articles/9ada7e0236463a
