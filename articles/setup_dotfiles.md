---
title: 'PC移行を楽にする！dotfilesで開発環境を管理しよう。'
emoji: '📁'
type: 'tech'
topics: ['dotfiles', 'zsh', 'Mac', 'Git', 'homebrew']
published: false
---

## はじめに

この度転職をしました。
大体会社のPCから支給されるPCを使うことになるのですが、
その際セットアップが面倒だな……と思ってしまいました。
特に私は直近で転職をしているので、
前もやったことをまた何度もやる必要がありました。
私はpecoの設定、Gitのエイリアス、ターミナルの透明化設定、
Raycastの設定などを行なっています。
後自分の名前が普通だと出てこないので、辞書設定とかも行う必要があります。
多分こだわっている人はもっとこだわっているし大変だと思います。
私もこれらを毎回手動で設定し直すのは本当に面倒だなと感じました。

そこで今回、**dotfiles**という仕組みのことを知ったため、
これを使って開発環境を管理することにしました。
結果、次回のPC移行時には**たった1つのコマンドで環境が復元できる**ようになるはずです。
今回の記事dotfilesとは何か、どうやって作るのかをまとめたいと思います。

## dotfilesとは？

**dotfiles**とは、設定ファイル（`.zshrc`、`.gitconfig`など）をGitHubで管理する仕組みです。

### なぜ「dotfiles」という名前？

Unixシステムでは、設定ファイルは通常「`.`（ドット）」で始まる名前になっています。例：

- `.zshrc` - zshの設定
- `.gitconfig` - Gitの設定
- `.vimrc` - Vimの設定

これらのファイルをまとめて管理するリポジトリなので、**dotfiles**と呼ばれています。

### dotfilesのメリット

1. **PC移行が超簡単**
    - 新しいPCでもコマンド1つで環境復元
2. **設定のバックアップ**
    - GitHubに保存されているので、PCが壊れても安心
3. **設定の共有**
    - チームメンバーや他のエンジニアと共有できる
4. **変更履歴が残る**
    - いつ、どんな設定を変更したかがわかる

## 今回作成したdotfilesの構成

私が実際に作成したdotfilesの構成はこちらです：

```text
dotfiles/
├── .gitconfig          # Git設定（エイリアス）
├── .zshrc              # zsh設定（peco、エイリアス）
├── Brewfile            # Homebrewパッケージリスト
├── setup.sh            # セットアップ自動化スクリプト
├── README.md           # 説明書
├── raycast/            # Raycast設定
│   └── config.rayconfig
└── .gitignore          # Git除外設定
```

### 含まれている設定

### 1. peco（コマンド履歴検索）

`Ctrl + R`で過去のコマンドを検索できる超便利ツール

### 2. Gitエイリアス

```bash
git st    # git status
git br    # git branch
git co    # git checkout
git ci    # git commit
```

### 3. ターミナル透明化

背景が見えるターミナルで作業効率UP

### 4. Raycast設定

ホットキーや拡張機能の設定

### 5. 辞書登録のバックアップ方法

macOSの辞書登録を新PCに移行する方法も記載

## セットアップ方法

### Step 1: dotfilesリポジトリを作成

まず、ローカルにdotfilesディレクトリを作ります。

```bash
mkdir ~/dotfiles
cd ~/dotfiles
```

### Step 2: 設定ファイルを作成

### .zshrc（peco設定を含む）

```bash
# Homebrew settings
eval "$(/opt/homebrew/bin/brew shellenv)"

# peco settings
# 過去に実行したコマンドを選択。ctrl-rにバインド
function peco-select-history() {
  BUFFER=$(\history -n -r 1 | peco --query "$LBUFFER")
  CURSOR=$#BUFFER
  zle clear-screen
}
zle -N peco-select-history
bindkey '^r' peco-select-history

# 過去に移動したことのあるディレクトリを選択。ctrl-uにバインド
function peco-cdr() {
  local destination="$(peco-get-destination-from-cdr)"
  if [ -n "$destination" ]; then
    BUFFER="cd $destination"
    zle accept-line
  else
    zle reset-prompt
  fi
}
zle -N peco-cdr
bindkey '^u' peco-cdr

# ブランチを簡単切り替え。git checkout lbで実行できる
alias -g lb='`git branch | peco --prompt "GIT BRANCH>" | head -n 1 | sed -e "s/^\*\s*//g"`'

# dockerコンテナに入る。deで実行できる
alias de='docker exec -it $(docker ps | peco | cut -d " " -f 1) /bin/bash'
```

### .gitconfig（Gitエイリアス）

```text
[user]
	name = YOUR_NAME_HERE
	email = YOUR_EMAIL_HERE

[alias]
	st = status
	br = branch
	co = checkout
	ci = commit
```

### Brewfile（パッケージリスト）

```ruby
# CLI Tools
brew "peco"
brew "tig"
brew "git"

# GUI Applications
cask "raycast"
```

### Step 3: セットアップスクリプトを作成

`setup.sh`を作成して、新しいPCでの設定を自動化します。

```bash
#!/bin/bash

set -e

echo "🚀 Starting dotfiles setup..."

# Homebrewのインストールチェック
if ! command -v brew &> /dev/null; then
    echo "📦 Installing Homebrew..."
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

    echo "⚙️  Setting up Homebrew PATH..."
    echo >> ~/.zprofile
    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
else
    echo "✅ Homebrew is already installed"
fi

# パッケージのインストール
echo "📦 Installing packages from Brewfile..."
brew bundle --file=~/dotfiles/Brewfile

# シンボリックリンクの作成
echo "🔗 Creating symbolic links..."

# バックアップディレクトリの作成
BACKUP_DIR=~/dotfiles_backup_$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# 既存ファイルがある場合はバックアップ
if [ -f ~/.zshrc ]; then
    echo "📋 Backing up existing .zshrc to $BACKUP_DIR"
    cp ~/.zshrc "$BACKUP_DIR/.zshrc"
fi

if [ -f ~/.gitconfig ]; then
    echo "📋 Backing up existing .gitconfig to $BACKUP_DIR"
    cp ~/.gitconfig "$BACKUP_DIR/.gitconfig"
fi

# シンボリックリンクを作成
ln -sf ~/dotfiles/.zshrc ~/.zshrc
ln -sf ~/dotfiles/.gitconfig ~/.gitconfig

echo ""
echo "✨ Setup completed!"
echo ""
echo "📝 Next steps:"
echo "  1. Edit ~/.gitconfig to add your name and email"
echo "  2. Run 'source ~/.zshrc' to apply changes"
echo "  3. (Optional) Import Raycast settings if you have one"
echo ""
```

実行権限を付与：

```bash
chmod +x setup.sh
```

### Step 4: GitHubにプッシュ

```bash
cd ~/dotfiles
git init
git add .
git commit -m "Initial commit"

# GitHubでリポジトリを作成してから
git remote add origin git@github.com:YOUR_USERNAME/dotfiles.git
git branch -M main
git push -u origin main
```

## 新しいPCでの復元方法

新しいPCでは、たったこれだけを行えば良いです。

```bash
# 1. リポジトリをクローン
git clone https://github.com/YOUR_USERNAME/dotfiles.git ~/dotfiles

# 2. セットアップスクリプトを実行
cd ~/dotfiles
./setup.sh

# 3. Git設定を自分の情報に変更
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 4. zshrcを再読み込み
source ~/.zshrc
```

これだけで、peco、Gitエイリアス、その他の設定がすべて復元されます。

## 便利な使い方

### pecoでコマンド履歴検索

```bash
# Ctrl + R を押す
# → 過去のコマンドが一覧表示される
# → 文字を入力して絞り込み
# → Enterで実行
```

### pecoでディレクトリ移動

```bash
# Ctrl + U を押す
# → 過去に訪れたディレクトリが一覧表示
# → 文字を入力して絞り込み
# → Enterで移動
```

### Gitブランチを簡単切り替え

```bash
git checkout lb

# → ブランチ一覧が表示される
# → 文字を入力して絞り込み
# → Enterで切り替え
```

GitHubで「dotfiles」と検索すると、いろんな人の設定が見られるので、
是非他のエンジニアの方の設定も見てみてください！

私のdotfilesのリポジトリは下記です。

https://github.com/hiiragi17/dotfiles

## まとめ

dotfilesは、一度作ってしまえば一生使える資産です。
最初は面倒に感じるかもしれませんが、次のPC移行時の楽さを考えると、
作っておいて損はないかなと思います。
是非皆様も作ってみてください。

それでは最後まで読んでいただき、ありがとうございました！
この記事が誰かの参考になれば幸いです。

## 参考リンク

この記事は下記を参考にして書かせていただきました。

- [GitHub dotfiles](https://dotfiles.github.io/)
- [peco](https://github.com/peco/peco)
- [pecoを使ったらターミナルの操作が劇的に効率化できた話](https://qiita.com/keisukee/items/9b815e56a173a281f42f)
- [【透明化】ターミナルの環境設定変更しないと絶対に損します>_<](https://qiita.com/chisaki0606/items/28e4df4bb778e8bdaae8)
