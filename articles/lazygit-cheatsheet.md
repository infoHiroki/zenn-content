---
title: "git addとgit commitを手打ちしてる全人類に告ぐ"
emoji: "⚡"
type: "tech"
topics: ["Git", "LazyGit", "ターミナル", "開発環境", "効率化"]
published: true
---

## 😮‍💨 告白する

告白する。俺はGitが苦手だった。

いや、苦手というか、**怖かった**。🫣 `git rebase`という文字列を見ると胃がキュッとなる。`git reset --hard`に至っては、もはやホラーだ。取り返しのつかないことをしそうで、毎回手が震えていたのである。

で、結局どうなるかというと、`git add .`と`git commit -m "fix"`を一日に何十回も打つだけの人生になる。ブランチ？マージ？コンフリクト？🙈 見なかったことにする。まるで健康診断の結果を封筒ごとゴミ箱に入れる中年のようではないか。

そんな俺を救ったのがLazyGitだった。✨

## 🤔 LazyGitとは何か

ターミナルで動くGitクライアントである。こういうやつ。👇

![LazyGitでコミット&プッシュ](https://github.com/jesseduffield/lazygit/raw/assets/demo/commit_and_push-compressed.gif)

左にブランチ、真ん中にファイル、右にdiff。**全部が一画面に収まっている**。📺

SourceTreeやGitKrakenのようなGUIアプリに似ているが、こちらはターミナルで完結する。マウスに手を伸ばさなくていい。キーボードだけで全部できる。⌨️

これの何がいいかというと、**Gitの全体像が常に見えている**ということだ。🗺️ コマンドラインだと「今どのブランチにいるんだっけ」「何がステージされてるんだっけ」と不安になる。LazyGitなら全部見えてる。不安が消える。精神衛生上、非常によろしい。🧘

## 📥 インストールする

```bash
# macOS 🍎
brew install lazygit

# Arch Linux 🐧
sudo pacman -S lazygit

# Ubuntu / Debian 🐧
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*')
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
tar xf lazygit.tar.gz lazygit
sudo install lazygit /usr/local/bin

# Go ユーザー 🐹
go install github.com/jesseduffield/lazygit@latest
```

Gitリポジトリの中で`lazygit`と打ってくれ。それだけでいい。✅

画面が出たら勝ちだ。🎉

## 🖐️ 最初に覚える5つの操作

全キーバインドを覚えようとしてはいけない。📖 それは辞書を最初から読もうとするのと同じで、正気の沙汰ではない。

この5つだけでいい。日常の90%はこれで回る。

### 1️⃣ ステージング ⚡

ファイルパネルで**スペースキー**を押す。以上である。🫡

赤いファイル（未ステージ）が緑（ステージ済み）になる。💚 もう一回押せば戻る。トグルだ。

全部まとめてステージしたいなら`a`を押す。

`git add .`と打つ必要は、もうない。永遠に。👋

### 2️⃣ コミット 💾

`c`を押す。メッセージ入力欄が出る。書いてEnter。おしまい。🎬

![コミット&プッシュの実演](https://github.com/jesseduffield/lazygit/raw/assets/demo/commit_and_push-compressed.gif)

直前のコミットを修正したい？`C`（大文字）でamendだ。

![古いコミットの修正](https://github.com/jesseduffield/lazygit/raw/assets/demo/amend_old_commit-compressed.gif)

「あ、コミットメッセージにtypoが……」😱 大丈夫。`C`で直せる。人生にもこういう機能があればいいのに。

### 3️⃣ プッシュ / プル 🚀

```
p → プッシュ ⬆️
P → プル ⬇️（大文字）
f → フェッチ 🔄
```

3つだけだ。小文字と大文字の違いに注意してくれ。☝️

### 4️⃣ ブランチ操作 🌿

ブランチパネル（`2`キーで移動）で 👇

| キー | 何が起きる |
|------|-----------|
| `スペース` | チェックアウト（切り替え） 🔀 |
| `n` | 新規ブランチ作成 🆕 |
| `d` | ブランチ削除 🗑️ |
| `M` | マージ 🤝 |
| `r` | ブランチ名変更 ✏️ |

`git checkout -b feature/new-thing`なんて二度と打たなくていい。`n`を押して名前を入れるだけ。😌 指が楽になる。腱鞘炎のリスクが減る。

### 5️⃣ アンドゥ 🔄

`z`を押す。**直前の操作が取り消される**。

![アンドゥの実演](https://github.com/jesseduffield/lazygit/raw/assets/demo/undo-compressed.gif)

これがどれだけ偉大かわかるだろうか。🤯 `git reset`とか`git reflog`とか調べなくていい。`z`を押すだけで元に戻る。

Ctrl+Zの感覚でGitを使える。人類がGitに対して抱いていた恐怖の大半は、この`z`で解消されるのではないか。🙏✨

## 🎮 パネルの構造

画面は5つのパネルに分かれている。📊

| キー | パネル | 何があるか |
|------|--------|-----------|
| `1` | ステータス | 今いるブランチとか 📍 |
| `2` | ブランチ | ブランチ一覧 🌿 |
| `3` | ファイル | 変更ファイル 📁 |
| `4` | コミット履歴 | ログ 📜 |
| `5` | スタッシュ | 一時保存 📦 |

`Tab`で順番に移動もできるが、数字キーで飛ぶほうが圧倒的に速い。⚡ 迷ったら`?`を押せ。ヘルプが出る。🆘

## 📋 チートシート全部盛り

壁に貼っとけ。🖨️ デスクトップの壁紙にしてもいい。

### 📁 ファイル操作

| キー | 何が起きる |
|------|-----------|
| `スペース` | ステージ / アンステージ 🔄 |
| `a` | 全部ステージ 📥 |
| `d` | 変更を破棄 ⚠️💀 |
| `e` | エディタで開く 📝 |
| `o` | デフォルトアプリで開く 🔓 |

### 💾 コミット

| キー | 何が起きる |
|------|-----------|
| `c` | コミット ✅ |
| `C` | amend（直前のコミット修正） ✏️ |
| `m` | コミットメッセージ編集 📝 |

### 🌿 ブランチ

| キー | 何が起きる |
|------|-----------|
| `スペース` | チェックアウト 🔀 |
| `n` | 新規作成 🆕 |
| `d` | 削除 🗑️ |
| `M` | マージ 🤝 |
| `r` | 名前変更 ✏️ |

### 🌐 リモート

| キー | 何が起きる |
|------|-----------|
| `p` | プッシュ ⬆️ |
| `P` | プル ⬇️ |
| `f` | フェッチ 🔄 |

### 📦 スタッシュ

| キー | 何が起きる |
|------|-----------|
| `s` | スタッシュに保存 💼 |
| `a` | 適用（保持） 📤 |
| `p` | 適用して削除 🎯 |
| `d` | 削除 🗑️ |

### 🛠️ その他

| キー | 何が起きる |
|------|-----------|
| `z` | アンドゥ 🔥🔥🔥 |
| `?` | ヘルプ 🆘 |
| `/` | 検索 🔍 |
| `D` | diff表示切替 👀 |
| `Ctrl+r` | リフレッシュ 🔄 |
| `q` | 終了 👋 |

## 🧙 知っておくと幸せになれるテクニック

### 🎯 部分ステージング

ファイル全体じゃなくて、**変更の一部だけステージしたい**。そういうことがある。むしろ頻繁にある。

![行単位のステージング](https://github.com/jesseduffield/lazygit/raw/assets/demo/stage_lines-compressed.gif)

ファイルを選んでEnterを押すとdiffが見える。👀 そこで行単位やhunk単位で`スペース`を押せば、その部分だけステージできる。

`git add -p`でやるのとは比較にならない速さである。⚡ あのインタラクティブモードで`y`とか`n`とか`s`とか打ってた日々は何だったのか。🫠

### 🏗️ インタラクティブrebase

コミット履歴パネルで`e`を押すとインタラクティブrebaseに入れる。

![インタラクティブrebase](https://github.com/jesseduffield/lazygit/raw/assets/demo/interactive_rebase-compressed.gif)

コミットの順番入れ替え、squash、fixup、**全部ビジュアルでできる**。✨ `git rebase -i HEAD~5`と打って、vimで`pick`を`squash`に書き換える苦行から解放される。

あの苦行はなんだったのだろう、と思う。たぶん修行だったのだろう。🧘 しかし俺たちはもう十分修行した。

### 🔧 コンフリクト解決

マージでコンフリクトが起きたら、LazyGitがコンフリクト箇所をハイライトしてくれる。🎨 矢印キーで移動して、どっちを採用するか選ぶだけ。

テキストエディタで`<<<<<<<`を目で探す時代は終わりだ。👁️ あの不毛な目grep、もうやらなくていい。

### 🍒 チェリーピック

別ブランチのコミットを1個だけ持ってきたい。そういうことがある。

![チェリーピック](https://github.com/jesseduffield/lazygit/raw/assets/demo/cherry_pick-compressed.gif)

LazyGitなら、コミットを選んで持ってくるだけ。🎯 `git cherry-pick`のハッシュ値をコピペする作業から解放される。

## ⚙️ おすすめ設定

`~/.config/lazygit/config.yml` に書く。📝

```yaml
# 🖱️ マウスを有効に（たまに使いたいとき用）
gui:
  mouseEvents: true
  showBottomLine: false

# ✏️ エディタをNeovimに
os:
  editPreset: "nvim"

# 🇯🇵 日本語コミットメッセージの文字化け対策
git:
  overrideGpg: true
```

設定はこれだけでいい。凝り始めると沼なので、最小限にしておくのが精神衛生上よろしい。🧘

## 🎬 まとめ

LazyGitは「Gitを使いやすくするツール」ではない。**Gitのことを考えなくて済むツール**だ。🧠✨

スペースでステージ、`c`でコミット、`p`でプッシュ。脳のリソースをGitの操作に使わず、コードに集中できる。💻

`z`のアンドゥは保険だ。🛡️ 何かやらかしても一瞬で戻れる。この安心感は、深夜の`git push -f`にも似た蛮勇を与えてくれる（やるなよ）。😂

まだ`git add`を手打ちしてるなら、今日インストールしてくれ。👇

```bash
brew install lazygit && lazygit
```

明日から「なんで今までCLIで打ってたんだ」って思うはずだ。🤦

俺がそうだった。🔥
