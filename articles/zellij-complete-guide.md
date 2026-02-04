---
title: "tmux使いが全員Zellijに乗り換える日が来た"
emoji: "🔥"
type: "tech"
topics: ["Zellij", "ターミナル", "tmux", "Rust", "開発環境"]
published: false
---

## tmuxの苦行、覚えてるか

tmux。あいつとの付き合いはもう長い。

`.tmux.conf`を何時間かけて書いた？プレフィックスキーを`Ctrl+b`から`Ctrl+a`に変えて、ペイン分割のキーバインドを設定して、ステータスバーの色を変えて、プラグインマネージャ入れて、resurrectで永続化して。

で、新しいマシンに移ったらまた最初からだ。

俺はある日気づいた。**ターミナルマルチプレクサーの設定に人生を費やしている**。本末転倒もいいところだ。コードを書く時間より`.tmux.conf`を弄る時間のほうが長い週があった。病気だろ、これは。

そんなときに出会ったのがZellijだ。

## Zellijって何

3行で言う。

- Rust製のターミナルマルチプレクサー
- tmuxの「何もかも自分で設定しろ」を「最初から全部入り」に変えたやつ
- 画面の下にキーバインドが常時表示されるから、マニュアル不要

以上。もう使いたくなっただろ。

## tmux vs Zellij、体験で語る

比較表とか星の数とか、そういうのはいい。体験で語る。

### 初日の体験

**tmux初日**: `man tmux`を開く。3000行ある。閉じる。Qiitaで「tmux 入門」を検索する。記事が50個出てくる。どれを読めばいいかわからない。とりあえず`Ctrl+b %`でペインを分割することだけ覚えて帰る。

**Zellij初日**: `zellij`と打つ。画面の下にキーバインドが全部書いてある。`Ctrl+P`押したらペインモードになった。`N`押したらペインが増えた。5分で理解した。帰り道ビールが美味い。

### 設定ファイル

**tmux**: `.tmux.conf`は育てるものだ。ネットで拾ってきた設定を継ぎ足して、数年かけて「自分だけの最強設定」を作り上げる。盆栽だよ盆栽。

**Zellij**: 設定ファイルなしで起動できる。デフォルトが既に良い。必要になったら`~/.config/zellij/config.kdl`に書き足す。**最初から快適**という概念がここにある。

### Neovimとの共存

**tmux**: `Ctrl+O`がtmuxとNeovimで競合する。解決策を調べると「キーバインドを変えろ」。どっちの？両方？もう嫌だ。

**Zellij**: `Ctrl+G`でロックモード。Zellijのキーバインドが全部無効になる。Neovim使い終わったらもう一回`Ctrl+G`。以上。**天才の発明だろこれ**。

### ペインの扱い

tmuxにはない機能がZellijにはある。**フローティングペイン**だ。

通常のペインの上に、浮かぶペインを出せる。`Ctrl+P` → `W`で出る。一時的にログを見たいとき、ちょっとコマンドを打ちたいとき、メインの作業を崩さずに作業できる。

これを知ったとき、俺は「なんで今までなかったんだ」と思った。

## インストール

```bash
# macOS（これだけ）
brew install zellij

# Arch Linux
sudo pacman -S zellij

# それ以外のLinux
wget https://github.com/zellij-org/zellij/releases/latest/download/zellij-x86_64-unknown-linux-musl.tar.gz
tar -xzf zellij-x86_64-unknown-linux-musl.tar.gz
sudo mv zellij /usr/local/bin/

# Rustユーザー
cargo install zellij
```

確認。

```bash
zellij --version
```

バージョンが出たら勝ちだ。`zellij`と打って起動してくれ。

## 最初に覚える操作5つ

全部覚えなくていい。この5つだけでいい。

### 1. ペイン分割 🪟

```
Ctrl+P → ペインモードに入る
  N → 新しいペイン（自動配置）
  ↓ → 下に分割
  → → 右に分割
  X → ペインを閉じる
```

ペイン間の移動は`Alt+矢印キー`。これはノーマルモードのまま使える。

### 2. タブ 📑

```
Ctrl+T → タブモードに入る
  N → 新しいタブ
  ← / → → タブ間移動
  R → タブ名を変更
  X → タブを閉じる
```

ノーマルモードで`Alt+数字`を押してもタブを切り替えられる。こっちのほうが速い。

### 3. セッション 💾

セッションはZellijの実行単位だ。デタッチすれば、裏で動き続ける。

```bash
# 名前をつけて起動
zellij -s my-project

# デタッチ（セッション内で）
# Ctrl+O → D

# セッション一覧
zellij ls

# 再接続
zellij a my-project
```

SSHが切れても、セッションは生きてる。再接続すれば元通りだ。

### 4. フローティングペイン 🎈

```
Ctrl+P → W → フローティングペインの表示/非表示をトグル
```

メインの作業中に「ちょっとgit statusだけ見たい」みたいなとき、最高に便利だ。普通のペインの上に浮かぶ。用が済んだら同じコマンドで消せる。

### 5. ロックモード 🔒

```
Ctrl+G → ロックモード ON/OFF
```

Neovim使うとき、これを押せ。Zellijのキーバインドが全部無効になる。Neovimの`Ctrl+P`（補完）とか`Ctrl+O`（ジャンプリスト）が問題なく使える。終わったらもう一回`Ctrl+G`。

## レイアウトが神

ここからがZellijの真骨頂だ。

レイアウトファイルを書いておけば、`zellij --layout`で一発で開発環境が立ち上がる。tmuxで`tmuxinator`使ってた人、あれの上位互換だと思ってくれ。

### KDLって何

Zellijの設定ファイルはKDL（KDL Document Language）で書く。YAMLより読みやすい。インデントで死なない。

### 実例: Web開発レイアウト

```kdl
// ~/.config/zellij/layouts/web-dev.kdl

layout {
    pane split_direction="vertical" {
        pane size="60%" {
            name "editor"
        }
        pane split_direction="horizontal" size="40%" {
            pane {
                name "server"
                command "npm"
                args "run" "dev"
            }
            pane {
                name "terminal"
            }
        }
    }
}
```

これで何が起きるか。

```
┌──────────────────┬────────────┐
│                  │   server   │
│     editor       │  npm run   │
│                  │    dev     │
│                  ├────────────┤
│                  │  terminal  │
└──────────────────┴────────────┘
```

起動は一行。

```bash
zellij --layout web-dev
```

エディタが開いて、devサーバーが勝手に立ち上がって、ターミナルも用意されてる。**毎朝これで始まる**。最高だ。

### 実例: フルスタック開発

```kdl
layout {
    tab name="Frontend" {
        pane split_direction="vertical" {
            pane name="code" size="60%"
            pane name="dev-server" {
                command "npm"
                args "run" "dev"
            }
        }
    }
    tab name="Backend" {
        pane split_direction="vertical" {
            pane name="code" size="60%"
            pane name="api" {
                command "go"
                args "run" "main.go"
            }
        }
    }
    tab name="Git" {
        pane
    }
}
```

`Alt+1`でフロントエンド、`Alt+2`でバックエンド、`Alt+3`でGit。プロジェクトの全体像が一瞬で手に入る。

## Neovimとの共存、詳しく

さっきロックモードの話をしたけど、もう少し詳しくやる。

### 競合するキー

| キー | Zellij | Neovim |
|------|--------|--------|
| `Ctrl+O` | セッションモード | ジャンプリスト（戻る） |
| `Ctrl+N` | リサイズモード | 補完候補（次） |
| `Ctrl+P` | ペインモード | 補完候補（前） |

### 解決策は4つある

**① ロックモード（おすすめ）**

`Ctrl+G`で切り替え。シンプルで確実。

**② Altキーに逃がす**

```kdl
// ~/.config/zellij/config.kdl
keybinds {
    normal {
        bind "Alt p" { SwitchToMode "pane"; }
        bind "Alt t" { SwitchToMode "tab"; }
        bind "Alt n" { SwitchToMode "resize"; }
        bind "Alt o" { SwitchToMode "session"; }
    }
}
```

Ctrlの競合が全部消える。俺はこれが一番好きだ。

**③ tmuxモード**

`Ctrl+B`でtmux互換モードに入れる。tmuxの筋肉記憶がある人向け。

**④ Neovimプラグイン**

`zellij-nav.nvim`を入れると、Neovimのバッファ移動とZellijのペイン移動がシームレスになる。`Ctrl+H/J/K/L`でNeovim↔Zellijを行き来できる。

## 設定ファイルの基本

必要になったら書けばいい。場所はここ。

```
~/.config/zellij/config.kdl
```

よく使う設定だけ書いておく。

```kdl
// マウス操作を有効に
mouse_mode true

// macOSのコピー
copy_command "pbcopy"

// デフォルトのレイアウトをコンパクトに
default_layout "compact"

// ペインの枠線を消す（スッキリする）
pane_frames false

// セッションを自動保存
session_serialization true

// スクロールバッファ
scroll_buffer_size 10000
```

## まとめ

tmuxは偉大だった。GNU Screenから引き継いだ伝統があり、枯れた技術としての安定感がある。

でもZellijは「なぜ今までこうじゃなかったのか」を突きつけてくる。

- 設定ファイルなしで使える
- キーバインドが画面に書いてある
- フローティングペインがある
- レイアウトファイルで開発環境を一発構築
- ロックモードでNeovimと共存

tmuxの`.tmux.conf`を書く時間で、Zellijならもうコードを書き始めてる。

試してみてくれ。`brew install zellij`、それだけだ。

```bash
brew install zellij && zellij
```

5分後には「なんで早く乗り換えなかったんだ」と思ってるはずだ。

俺がそうだった。
