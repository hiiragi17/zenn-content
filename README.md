# zenn-content

My zenn articles is here.

<a href="https://zenn.dev/hiiragi" target="_blank"><img alt="Zenn" src="https://img.shields.io/badge/Zenn-3EA8FF.svg?&style=for-the-badge&logo=Zenn&logoColor=white" /></a>

## Zenn latest posts

<!-- BLOG-POST-LIST:START -->
- [今更人に聞けないPythonの基本の用語を、PHPと比較しながらまとめてみる。](https://zenn.dev/arsaga/articles/e4fe73447495c4)
- [輪読会でのファシリテーターのススメ。](https://zenn.dev/arsaga/articles/605db9323f40cb)
<!-- BLOG-POST-LIST:END -->

## Local Development

### Install dependencies

```bash
npm ci
```

### Add articles

```bash
npx zenn new:article --slug 記事のスラッグ --title タイトル --type tech --emoji ✨
```

- `--slag`: 記事のスラッグを指定する。`articles/[slug].md`でファイルが生成される。詳細は[コチラ](https://zenn.dev/zenn/articles/what-is-slug)
- `--title`: 記事のタイトル
- `--type`: 記事のタイプ。 `tech` | `idea`
- `--emoji`: アイキャッチ用の絵文字を1つ指定

### commit format

```
<type>(<scope>): <subject>

e.g. 以下のようなコミットメッセージを求めるようになります
docs(zenn-github-repo): add articles
```

### Add book

```bash
npx zenn new:book
```

### preview

```bash
npm start
```

### Reference repositories
https://github.com/jonghyo/zenn.dev