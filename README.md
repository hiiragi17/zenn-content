# zenn-content

My zenn articles is here.

<a href="https://zenn.dev/hiiragi" target="_blank"><img alt="Zenn" src="https://img.shields.io/badge/Zenn-3EA8FF.svg?&style=for-the-badge&logo=Zenn&logoColor=white" /></a>

## Zenn latest posts

<!-- BLOG-POST-LIST:START -->
- [Findyさんの抽選に当たったので、「O'Reilly 学び放題プログラム」を体験してみた。](https://zenn.dev/hiiragi/articles/oreilly_online_learning)
- [複雑な削除制限をLaravelのPolicyで統一管理する。](https://zenn.dev/hiiragi/articles/delete_setting_policy)
- [React・Next.js・TypeScriptのキャッチアップのために、荒唐無稽なリモート理由を診断してくれるアプリを作成しました。](https://zenn.dev/hiiragi/articles/create_remote-diagnosis-app)
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

### lint
```bash
npx markdownlint-cli2 articles/delete_setting_policy.md
npx markdownlint-cli2 --fix articles/delete_setting_policy.md
```
```bash
npx textlint articles/delete_setting_policy.md
npx textlint -- fix articles/delete_setting_policy.md
```

### Reference repositories
https://github.com/jonghyo/zenn.dev