---
title: "Claude Codeマルチエージェント「kinoko」をOSS化した——セットアップからカスタマイズまで"
emoji: "🍄"
type: "tech"
topics: ["ClaudeCode", "マルチエージェント", "tmux", "OSS"]
published: false
---

## 🍄 kinoko、OSSになりました

[前回の記事](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)で「菌類5体に転生した」話を書いた。あれから個人的に使い続けて、ハードコードを潰して、誰でも使える形にした。

GitHub: [infoHiroki/kinoko](https://github.com/infoHiroki/kinoko)

**kinoko** は Claude Code CLI の複数インスタンスを tmux 上で並列稼働させるマルチエージェントシステム。指揮官（霊芝）がタスクを分解して、4体のワーカーが並列で実行する。

```
      ユーザー
        │
        ▼ (自然言語で指示)
  ┌───────────┐
  │   霊芝    │  指揮官 (Opus)
  │  reishi   │  タスク分解・進捗管理
  └─────┬─────┘
        │ タスク配分
        ▼
┌──────────┬──────────┐
│   beni   │   cord   │  Opus × 2
├──────────┼──────────┤
│   mata   │  enoki   │  Sonnet × 2
└──────────┴──────────┘
```

前の記事は「なぜ作ったか」の話。今回は「どう使うか」の話をする。

## 前提条件

必要なもの:

- **tmux 3.4以上** — `pane-colours` 機能を使うため。`tmux -V` で確認
- **Claude Code CLI** — `npm install -g @anthropic-ai/claude-code`
- **Anthropic APIキー** — 環境変数 `ANTHROPIC_API_KEY` に設定
- **macOS or Linux** — Windows は未対応

tmux のバージョンが古い人は `brew install tmux`（macOS）で最新版が入る。

## セットアップ（3分）

```bash
git clone https://github.com/infoHiroki/kinoko.git
cd kinoko
./deploy.sh
```

これだけ。

`deploy.sh` が以下を自動でやる:

1. tmux セッション `kinoko` を作成
2. 5ペインに分割（ワーカー4 + 指揮官1）
3. 各ペインに `@agent_id` を設定（ペイン特定用）
4. Claude Code を5インスタンス起動
5. 全エージェントに指示書（`instructions/*.md`）を読み込ませる

起動すると菌糸ネットワーク図が表示される:

```
  ┌─────────────────────────────────────────────────┐
  │  🍄 kinoko  菌糸ネットワーク型マルチエージェント  │
  └─────────────────────────────────────────────────┘

     beni ──────┐          ┌────── cord
     [Opus]     │          │     [Opus]
                ├──────────┤
     mata ──────┤  reishi  ├────── enoki
     [Sonnet]   │  [Opus]  │   [Sonnet]
                └──────────┘

  菌糸接続... beni ✓ cord ✓ mata ✓ enoki ✓ reishi ✓

  ✅ 全菌接続完了。地下ネットワーク稼働中。
```

その他のコマンド:

```bash
./deploy.sh --clean  # キューをリセットして起動（前回の残骸を掃除）
./deploy.sh --kill   # セッションを停止
```

## 使い方

起動したら、下部の全幅ペイン（霊芝）に自然言語で指示するだけ。

```
このプロジェクトのREADMEを書いて、テストも追加して
```

霊芝が勝手に分解する:

- 「READMEの執筆はベニテングタケに」（創造的なタスク → Opus）
- 「テスト追加は冬虫夏草に」（力技 → Opus）
- 「lint修正はエノキに」（軽量 → Sonnet）

各ワーカーが `queue/tasks/{worker}.md` からタスクを読み取って実行。完了したら `queue/reports/{worker}.md` に結果を書いて霊芝に `send-keys` で報告。霊芝が `dashboard.md` に進捗をまとめる。

**全部リアルタイムで見える。** これが kinoko の核心。Task機能やAPI経由のサブエージェントと違って、全員の思考過程がtmux上で丸見え。間違った方向に走ってたら途中で止められる。

### dashboard.md

進捗はダッシュボードに集約される:

```markdown
# kinoko ダッシュボード

## 🚨 要対応
ユーザーの判断が必要な事項

## 🔄 進行中
- [beni] task_001: README執筆中
- [cordyceps] task_002: テスト追加中

## ✅ 完了
- [enoki] task_003: lint修正完了 (14:30)

## ⏸️ 待機中
なし
```

霊芝がリアルタイムで更新する。`dashboard.md` を開いておけば全体像が掴める。

## カスタマイズ

### モデル変更

`deploy.sh` 冒頭の変数を書き換える:

```bash
MODEL_BENI="opus"       # ベニテングタケ
MODEL_CORD="opus"       # 冬虫夏草
MODEL_MATA="sonnet"     # マタンゴ
MODEL_ENOK="sonnet"     # エノキ
MODEL_REISHI="opus"     # 霊芝（指揮官）
```

全員 Sonnet にすればコストは激減する。全員 Opus にすれば精度は上がるが$200プランでも厳しくなる。用途に合わせて調整してほしい。

個人的な推奨:

| パターン | 構成 | 向いてるタスク |
|---------|------|-------------|
| デフォルト | Opus×3 + Sonnet×2 | バランス型。大半のタスクに対応 |
| 節約モード | Sonnet×5 | コスト重視。軽めのタスクに |
| 全力モード | Opus×5 | 精度重視。複雑なリファクタリング等 |
| ハイブリッド | 霊芝Opus + 全ワーカーSonnet | 指揮だけ賢くしたい場合 |

### 権限スキップ

```bash
SKIP_PERMISSIONS=false   # デフォルト: 毎回確認
SKIP_PERMISSIONS=true    # 全ツール呼び出しが確認なしで実行
```

`true` にするとエージェント間の `send-keys` 通信が完全自動化される。ただし **すべてのツール呼び出し** が確認なしで実行される点に注意。ファイル書き込みもコマンド実行も全部ノーチェック。

俺は `true` で使ってる。リアルタイムで見えてるから危なくなったら止められる。でもデフォルトは `false` にした。初回は確認ありで動かして、慣れたら外すのを推奨。

### スピナー動詞

kinoko には菌類テーマのスピナー動詞サンプルが付いてる。

```bash
cp examples/spinner-verbs.json .claude/settings.json
```

Claude Code の「Thinking...」が「胞子を拡散中...」「菌糸を展開中...」になる。完全に趣味の領域だが、ターミナルの雰囲気が変わる。

詳しくは[スピナー記事](https://zenn.dev/hirokitakamura/articles/claude-code-spinner-verbs)を参照。

## 通信アーキテクチャ

kinoko の通信は驚くほどローテクだ。

```
霊芝 → queue/tasks/beni.md にMarkdownで書き込み
     → tmux send-keys で「タスクを確認してください」

beni → queue/tasks/beni.md を読む
     → 作業実行
     → queue/reports/beni.md にMarkdownで結果を書く
     → tmux send-keys で「報告があります」
```

**API通信ゼロ。** エージェント間の通信コストはゼロ。ファイル書き込みとtmuxのキー送信だけ。通信にトークンを使わない。

### なぜ Markdown か

将軍システムはYAMLで通信していた。kinoko はMarkdownに変えた。理由:

- Claude は Markdown を最も自然に読み書きできる
- YAML はインデントやクォートのパースエラーが地味に面倒
- 人間も読みやすい（デバッグが楽）

### send-keys の罠と対策

tmux の `send-keys` には罠がある。kinoko ではこれらを踏み抜いて、対策を組み込んだ。

**罠1: ペインインデックスが不安定**

tmux はペインを分割するとインデックスを位置ベース（上→下、左→右）で振り直す。分割順序とインデックスが一致しない。

対策: 各ペインに `@agent_id` カスタム属性を設定し、名前でペインを引く。

```bash
# ❌ インデックスで指定（分割後にズレる）
tmux send-keys -t kinoko:0.2 'メッセージ'

# ✅ @agent_id で引く（安定）
PANE=$(tmux list-panes -t kinoko:0 \
  -F '#{pane_id} #{@agent_id}' \
  | grep " matango$" | cut -d' ' -f1)
tmux send-keys -t "$PANE" 'メッセージ'
```

**罠2: Enter が権限プロンプトに吸われる**

テキストと Enter を1回で送ると、Claude Code の権限確認プロンプトが途中で出た場合に Enter がそっちに吸われる。

対策: 必ず2回に分けて送信する。

```bash
# ❌ 1回で送る（Enterが吸われる場合がある）
tmux send-keys -t "$PANE" 'メッセージ' Enter

# ✅ 2回に分ける
tmux send-keys -t "$PANE" 'メッセージ'
tmux send-keys -t "$PANE" Enter
```

**罠3: 高速送信でバッファ溢れ**

複数ペインに連続で send-keys すると、tmux のバッファが溢れてメッセージが失われることがある。

対策: 2秒間隔を空ける。

```bash
tmux send-keys -t "$PANE_BENI" 'タスクを確認してください'
tmux send-keys -t "$PANE_BENI" Enter
sleep 2  # これが大事
tmux send-keys -t "$PANE_CORD" 'タスクを確認してください'
tmux send-keys -t "$PANE_CORD" Enter
```

### pane-colours[0] — 黒文字問題の解決

ダークテーマのターミナルで ANSI color 0（黒）を使ったテキストが見えない問題がある。tmux 3.4 以降なら `pane-colours[0]` で色を再定義できる:

```bash
tmux set-option -p -t "$PANE" 'pane-colours[0]' 'colour240'
```

ANSI の「黒」がグレー（#585858）に変わって読めるようになる。`deploy.sh` に組み込み済み。地味だけど実用性は高い。

## 公式 Agent Teams との違い

Claude Code には公式のマルチエージェント機能「Agent Teams」が実験的に追加されている:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

kinoko と Agent Teams、どっちを使うべきか。

| | kinoko 🍄 | Agent Teams ⚡ |
|---|---|---|
| **セットアップ** | `git clone` + `./deploy.sh` | 環境変数1つ |
| **可視性** | 全ペイン丸見え | リーダーペインのみ |
| **通信コスト** | ゼロ（ファイル+send-keys） | トークン消費あり |
| **カスタマイズ** | 全部いじれる（自分のコード） | 設定の範囲内 |
| **エージェント設計** | 名前・性格・適性を定義可能 | 自動割り当て |
| **安定性** | 自己責任 | 公式サポート |
| **学び** | めちゃくちゃ学べる | すぐ使える |

**Agent Teams を選ぶべき人:**
- すぐに使いたい。設定は最小限がいい
- 公式サポートの安心感が欲しい
- マルチエージェントの仕組みには興味がない

**kinoko を選ぶべき人:**
- 全エージェントの動きをリアルタイムで見たい
- エージェントの性格や適性を自分で設計したい
- tmux + ファイル通信の仕組みを理解したい
- 「ベニテングタケが報告を書いてる」のを見てニヤニヤしたい

正直、実用性だけなら Agent Teams のほうが楽。でも kinoko は**仕組みが全部見える教材**でもある。マルチエージェントを理解したいなら、一度自分で触ってみてほしい。

## ファイル構成

```
kinoko/
├── CLAUDE.md                # 全エージェント共通の設定・ルール
├── deploy.sh                # 起動スクリプト
├── dashboard.md             # 進捗ダッシュボード（霊芝が更新）
├── instructions/
│   ├── reishi.md            # 霊芝の行動ルール
│   └── worker.md            # ワーカー共通ルール
├── queue/
│   ├── command.md           # ユーザー → 霊芝
│   ├── tasks/{worker}.md    # 霊芝 → ワーカー（タスク指示）
│   └── reports/{worker}.md  # ワーカー → 霊芝（完了報告）
└── examples/
    └── spinner-verbs.json   # スピナー動詞サンプル
```

改造したいなら、まず `CLAUDE.md` を読むのが早い。全エージェントの行動ルール、通信プロトコル、禁止事項が書いてある。kinoko の設計書そのもの。

## コスト感

5体の Claude Code インスタンスが並列稼働するので、当然コストは嵩む。

目安:

- **$100プラン（Max）**: ギリギリ運用可能。Sonnet多めの構成推奨
- **$200プラン**: 余裕を持って運用可能。デフォルト構成で快適
- **APIキー直接**: 従量課金。タスク量に比例。小さいタスクなら安い

$100プランで全員 Opus にすると一瞬でレートリミットに達する。これは前回の記事で書いた通り。デフォルト構成（Opus×3 + Sonnet×2）でも、長時間ぶん回すとリミットに引っかかる。休憩を挟みながら使うか、Sonnet比率を上げるか、$200プランに上げるか。

## 既知の制限

- **macOS / Linux のみ** — Windows は未対応
- **tmux 3.4 以上が必要** — `pane-colours` 機能のため
- **API 使用量** — 5インスタンス並列なので通常の数倍
- **コンテキスト消失** — 長時間稼働するとコンパクション（コンテキスト圧縮）が起きる。`CLAUDE.md` に回復手順を書いてあるので、エージェントは自力で復帰できる。が、完璧ではない

## まとめ

kinoko は「マルチエージェントを自分で組んでみたい人」のためのスターターキット。

公式 Agent Teams がある今、自作する実用的な理由は正直少ない。でも全部見えること、全部いじれること、全部理解できること——これは公式ツールでは得られない。

```bash
git clone https://github.com/infoHiroki/kinoko.git
cd kinoko
./deploy.sh
```

3分で菌糸ネットワークが動き出す。

---

:::message
前回の記事: [Claude Code マルチエージェント10体で$100溶けたから菌類5体に転生した話](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)
スピナーの話: [Claude Codeの処理待ちが「般若心経ラップ中」になった日](https://zenn.dev/hirokitakamura/articles/claude-code-spinner-verbs)
:::
