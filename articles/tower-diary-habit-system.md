---
title: "tower - Claude Codeと作る日記・習慣管理システム"
emoji: "🗼"
type: "tech"
topics: ["ClaudeCode", "習慣", "Markdown", "Git", "Babashka"]
published: false
---

# tower - Claude Codeと作る日記・習慣管理システム

前回の記事「[ジャーナリングツールの変遷](https://zenn.dev/infohiroki/articles/journaling-tool-evolution)」で、GitHub + Claude Codeの組み合わせに落ち着いた話をしました。

今回は、この考え方をさらに発展させた**「tower」**というシステムを紹介します。

**GitHubリポジトリ**: [tower-template](https://github.com/infohiroki/tower-template)

---

## towerとは

毎朝「おはよう」と言うだけで、AIが日記の準備と習慣トラッキングをサポートするシステムです。

**特徴：**
- Markdownベース（特別なアプリ不要）
- Gitで管理（変更履歴が残る、複数マシンで同期）
- Claude Code連携（前日の振り返り、ストリーク計算、提案）

---

## なぜtowerを作ったか

### 既存の日記アプリへの不満

- **複雑すぎる** - 機能が多すぎて、開くのが億劫
- **続かない** - 専用アプリは結局使わなくなる
- **データがロックされる** - サービス終了したらどうなる？

### 「Markdown + Git」という解答

プログラマーにとって最も馴染みのあるツール：

- **Markdown** - どこでも編集できる、フォーマットがシンプル
- **Git** - 変更履歴が残る、複数マシンで同期できる
- **Claude Code** - AIが面倒な作業を自動化

この組み合わせなら、**10年後も同じファイルを読める**。

---

## towerの仕組み

### ディレクトリ構造

```
tower/
├── CLAUDE.md              # AIへの指示書（朝のルーティン定義）
├── diary/
│   ├── template.md        # 日記テンプレート
│   └── 2026/02/03.md      # 日記ファイル
└── scripts/
    └── sync-repos.clj     # ワークスペース同期（おまけ）
```

### 朝のルーティン

Claude Codeで「おはよう」と言うと：

1. **日記ファイル作成** - テンプレートから今日の日記を生成
2. **ストリーク計算** - 前日の習慣達成状況から連続日数を計算
3. **振り返り** - 前日の「気づき」「心がけ」を読み上げ
4. **提案** - 今日に活かせるポイントを提示

これだけで、日記を書く準備が整います。

---

## 日記テンプレートの設計

### 習慣トラッキング

```markdown
## 習慣
- [ ] 運動する 🔥5
- [ ] 読書する 🔥12
- [ ] 瞑想する 🔥3
```

- **チェックボックス** - 達成したら `[x]` に変更
- **ストリーク** - `🔥N` で連続達成日数を表示

AIが前日の日記を読んで、自動的にストリーク数を更新します。

### 感謝・気づき・心がけ

```markdown
## 感謝
今日感謝できることは？
- 朝早く起きれたこと
- 美味しいコーヒーが飲めたこと

## 気づき
- 朝の30分が一日の生産性を決める

## 心がけ
明日意識したいこと：
- 寝る前のスマホを控える
```

これらのセクションが「振り返り」の材料になります。

### 自己評価

```markdown
## 自己評価
| 評価 | 意味 |
|------|------|
| ⭐⭐⭐⭐⭐ | 最高の1日 |
| ⭐⭐⭐⭐ | 良い1日 |
| ⭐⭐⭐ | 普通の1日 |
| ⭐⭐ | イマイチ |
| ⭐ | 難しい1日 |

**今日の評価**: ⭐⭐⭐⭐
```

1日の終わりに振り返って評価。
AIが未記入を検知したら促してくれます。

---

## 習慣の哲学

### Never miss twice

**1日サボるのはOK。でも2日連続はNG。**

完璧主義は習慣の敵です。
1日サボっても自分を責めない。
ただし、2日目は何があってもやる。

これだけで、習慣は続きます。

### キーストーンハビット

**1つの習慣が他の良い習慣を引き起こす。**

例えば：
- 朝の運動 → 食事も気をつける → よく眠れる
- 読書 → 新しいアイデア → 仕事の質向上

自分にとってのキーストーンハビットを見つけることが大事。

### ストリークの力

```
🔥7
```

この数字が見えると、途切れさせたくない。
ゲーミフィケーションの力を借りて、習慣を維持します。

---

## Claude Codeとの連携

### CLAUDE.mdで指示を定義

```markdown
## 朝のルーティン（おはよう）

ユーザーが「おはよう」と言ったら：
1. 日記ファイルを作成
2. ストリークを計算
3. 前日の振り返りを表示
```

CLAUDE.mdファイルにルールを書いておくと、Claude Codeがそれに従って動作します。

### MCP連携（オプション）

- **google-calendar** - 今日の予定を日記に自動挿入
- **github** - 昨日のコミット履歴を自動ログ

MCPサーバーと連携すると、さらに自動化が進みます。

---

## おまけ：sync-repos.clj

複数のGitリポジトリを一括管理するBabashkaスクリプト。

```bash
# 全リポジトリをpull
bb scripts/sync-repos.clj

# ワークスペースも開く
bb scripts/sync-repos.clj --open
```

VS Codeのマルチルートワークスペースと組み合わせると、複数マシン間で開発環境を統一できます。

---

## セットアップ方法

### 1. リポジトリをフォーク

[tower-template](https://github.com/infohiroki/tower-template) を「Use this template」でフォーク。

### 2. クローン

```bash
git clone https://github.com/YOUR_USERNAME/tower.git ~/tower
```

### 3. カスタマイズ

`diary/template.md` を自分好みに編集：
- 習慣リストを変更
- 追加したい項目を追加

### 4. Claude Codeで開く

```bash
cd ~/tower
claude
```

### 5. 「おはよう」と言う

これだけで、日記の準備が整います。

---

## まとめ

towerは「続けられる日記」のためのシステムです。

**ポイント：**
- Markdown + Git = データが自分のもの
- Claude Codeが面倒な作業を自動化
- Never miss twiceで完璧主義を捨てる

日記や習慣管理で挫折した経験がある人は、ぜひ試してみてください。

**GitHubリポジトリ**: [tower-template](https://github.com/infohiroki/tower-template)

---

## 関連リンク

- [前回の記事: ジャーナリングツールの変遷](https://zenn.dev/infohiroki/articles/journaling-tool-evolution)
- [tower-template (GitHub)](https://github.com/infohiroki/tower-template)
