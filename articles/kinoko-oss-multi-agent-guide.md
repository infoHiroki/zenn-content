---
title: "Claude Code 5体を菌糸ネットワークで繋いだらOSSになった"
emoji: "🍄"
type: "tech"
topics: ["ClaudeCode", "マルチエージェント", "tmux", "OSS"]
published: false
---

## 🍄 菌類、公開します

[前回](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)、将軍システム10体で$100溶かして菌類5体に転生した話を書いた。

あれからしばらく個人で使い続けた。ハードコードを潰して、設定を変数に出して、誰でも動かせる形にした。

**kinoko、OSSにしました 🍄**

GitHub: [infoHiroki/kinoko](https://github.com/infoHiroki/kinoko)

Claude Code CLI を5インスタンス、tmux の上で並列に走らせる。指揮官の霊芝がタスクを分解して、4体のワーカーが同時にぶん回す。通信は Markdown ファイルと `send-keys`。それだけ。サーバーもDBもいらない。

```
      ユーザー
        │
        ▼ (自然言語で指示)
  ┌───────────┐
  │   霊芝    │  指揮官 (Opus) 🧙‍♂️
  │  reishi   │  タスク分解・進捗管理
  └─────┬─────┘
        │ タスク配分
        ▼
┌──────────┬──────────┐
│   beni   │   cord   │  Opus × 2 🔥
├──────────┼──────────┤
│   mata   │  enoki   │  Sonnet × 2 ⚡
└──────────┴──────────┘
```

前の記事は「なぜ作ったか」。今回は「どう使うか」の話。

---

## 💸 まず金の話をする

**Claude Max プラン必須。** $100/月 か $200/月。ここは避けられない。

Claude Code 自体が Max プラン前提で、それを5インスタンス並列に立てる。デフォルトはマタンゴとエノキがSonnet、残り3体がOpus。$100プランでも普通に動く。全員Opusにしたら死ぬけどな 💀

| プラン | 体感 | 一言 |
|--------|------|------|
| **$100/月** | 普通に動く | デフォルト構成で問題ない。たまにリミット踏む |
| **$200/月** | 快適 | 何も気にしなくていい |

「高い」と思っただろ？ 俺もそう思う。でも5体が並列でコード書いてる画面を見ると、元は取れてる気がしてくる。気がするだけかもしれん 🙃

---

## 📋 用意するもの

- **tmux**（3.4以上推奨）— `tmux -V` で確認。古かったら `brew install tmux` で入れ直せ
- **Claude Code CLI** — `npm install -g @anthropic-ai/claude-code`
- **macOS か Linux** — Windows は無理。WSLならいけるかもしれんが試してない

tmux 3.4 未満でも動くが、`pane-colours` が使えないので暗い背景だと黒文字が見にくくなる。気になるなら `brew install tmux` で最新を入れろ。

---

## 🚀 3分で起動する

```bash
git clone https://github.com/infoHiroki/kinoko.git
cd kinoko
./deploy.sh
```

これだけ。npm install もビルドも設定ファイルもない。

`deploy.sh` が勝手に tmux セッションを作って、5ペインに分割して、Claude Code を5体立ち上げて、全員に「お前は誰で何をすべきか」を教え込む。約20秒で全菌接続完了 🍄

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

美しいだろ？ そうでもないか。まあいい。

他のコマンド:

```bash
./deploy.sh --clean  # 前回の残骸を全部掃除して起動
./deploy.sh --kill   # セッション停止
```

---

## 🎮 使い方：霊芝に話しかけるだけ

起動したら、画面下部の全幅ペイン——霊芝に向かって喋る。日本語で。

```
このプロジェクトのREADMEを書いて、テストも追加して
```

霊芝が勝手に分解する 🧠

- 「READMEの執筆はベニテングタケに」（創造的 → Opus）
- 「テスト追加は冬虫夏草に」（力技 → Opus）
- 「lint修正はエノキに」（軽量 → Sonnet）

ワーカーは `queue/tasks/{名前}.md` からタスクを読んで実行。完了したら `queue/reports/{名前}.md` に結果を書いて霊芝に報告。霊芝が `dashboard.md` に進捗をまとめる。

**全部リアルタイムで見える。** これが核心。

Task機能やAPI経由のサブエージェントと違って、全員の思考過程がtmux上で丸見え。間違った方向に全力疾走してたら途中で止められる。ブラックボックスは嫌いだ。俺は全部見たい。

ちなみにこの記事も kinoko で書いてる。冬虫夏草が今まさにこの文章を叩いてる。メタ 🍄

---

## ⚙️ いじれるところ

### モデル変更

`deploy.sh` 冒頭の変数を書き換える:

```bash
MODEL_BENI="opus"       # ベニテングタケ 🍄
MODEL_CORD="opus"       # 冬虫夏草 🐛
MODEL_MATA="sonnet"     # マタンゴ 👻
MODEL_ENOK="sonnet"     # エノキ 🍢
MODEL_REISHI="opus"     # 霊芝（指揮官）🧙‍♂️
```

全員 Sonnet にすればレートリミットに余裕が出る。全員 Opus にすれば精度は上がるが$200プランでもキツくなる。

| パターン | 構成 | 向いてるやつ |
|---------|------|-------------|
| デフォルト | Opus×3 + Sonnet×2 | 大体これでいい |
| 節約モード | Sonnet×5 | 軽いタスク。財布に優しい |
| 全力モード | Opus×5 | 複雑なリファクタ。財布に厳しい |
| ハイブリッド | 霊芝だけOpus + 他Sonnet | 指揮官だけ賢くしたいとき |

### 権限スキップ

```bash
SKIP_PERMISSIONS=false   # デフォルト: 毎回確認あり
SKIP_PERMISSIONS=true    # 全部ノーチェックで実行
```

`true` にするとエージェント間の通信が完全自動になる。ただし**ファイル書き込みもコマンド実行も全部確認なし**。危ないっちゃ危ない。

俺は `true` で使ってる。リアルタイムで見えてるから、ヤバそうなら止められる。でもデフォルトは `false` にした。最初は確認ありで動かして、慣れたら外すのがいい。

### スピナー動詞 🎰

kinoko には菌類テーマのスピナー動詞500個が付いてる。

```bash
cp examples/spinner-verbs.json .claude/settings.json
```

Claude Code の「Thinking...」が「胞子を拡散中...」「菌糸ネットワーク5G接続中...」「腐敗のCI/CD...」になる。完全に趣味。でもターミナルの雰囲気が激変する。

詳しくは[スピナー記事](https://zenn.dev/hirokitakamura/articles/claude-code-spinner-verbs)に書いた。

---

## 🔌 中身の話：通信はMarkdownだけ

kinoko の通信は驚くほどローテクだ。

```
霊芝 → queue/tasks/beni.md にMarkdownで指示を書く
     → tmux send-keys で「タスクを確認してください」と叩く

beni → タスクファイルを読む → 作業する
     → queue/reports/beni.md に結果を書く
     → tmux send-keys で「報告があります」と霊芝を叩く
```

**エージェント間の通信コスト、ゼロ。** APIトークンを1つも使わない。ファイルの読み書きと tmux のキー送信だけ。

なぜ Markdown か？ 将軍システムは YAML で通信してたが、kinoko では捨てた。Claude は Markdown を一番自然に読み書きできる。YAML はインデントやクォートでパースエラーが出て地味にダルい。人間も読みやすい。デバッグが楽。三方良し 🎯

### send-keys の罠、3つ踏んだ 🪤

tmux の `send-keys`、見た目はシンプルだが罠がある。全部踏んだ。

**罠1: ペインインデックスが信用できない**

tmux はペインを分割するとインデックスを位置ベースで振り直す。分割した順番と一致しない。ハマった。

```bash
# ❌ これ、分割後にズレる
tmux send-keys -t kinoko:0.2 'メッセージ'

# ✅ @agent_id で名前引き。安定
PANE=$(tmux list-panes -t kinoko:0 \
  -F '#{pane_id} #{@agent_id}' \
  | grep " matango$" | cut -d' ' -f1)
tmux send-keys -t "$PANE" 'メッセージ'
```

**罠2: Enter が権限プロンプトに吸われる**

テキストと Enter を1回で送ると、Claude Code が権限確認を出したタイミングで Enter がそっちに吸われる。意図しない許可が出る。怖い。

```bash
# ❌ Enter が吸われることがある
tmux send-keys -t "$PANE" 'メッセージ' Enter

# ✅ 2回に分ける
tmux send-keys -t "$PANE" 'メッセージ'
tmux send-keys -t "$PANE" Enter
```

**罠3: 連続送信でバッファが溢れる**

複数ペインに高速で send-keys すると、メッセージが消える。2秒間隔を空ければ安定する。

```bash
tmux send-keys -t "$PANE_BENI" 'タスクを確認してください'
tmux send-keys -t "$PANE_BENI" Enter
sleep 2  # これ大事
tmux send-keys -t "$PANE_CORD" 'タスクを確認してください'
tmux send-keys -t "$PANE_CORD" Enter
```

全部 `deploy.sh` に組み込み済み。使う側は何も考えなくていい。

---

## ⚡ 公式 Agent Teams との違い

Claude Code には公式のマルチエージェント「Agent Teams」が実験的にある:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

環境変数1つで動く。楽。

じゃあ kinoko いらなくね？ って思うだろ。わかる。比較する。

| | kinoko 🍄 | Agent Teams ⚡ |
|---|---|---|
| セットアップ | `git clone` + `./deploy.sh` | 環境変数1つ |
| 可視性 | **全ペイン丸見え** | リーダーペインのみ |
| 通信コスト | **ゼロ** | トークン消費あり |
| カスタマイズ | 全部いじれる | 設定の範囲内 |
| エージェント設計 | 名前・性格・適性を定義可能 | 自動割り当て |
| 安定性 | 自己責任 | 公式サポート |
| 学び | **めちゃくちゃ学べる** | すぐ使える |

正直、実用性だけなら Agent Teams のほうが楽。

でも kinoko は**仕組みが全部見える**。マルチエージェントの通信、タスク分解、エラーハンドリング——公式ツールがブラックボックスでやってることを、自分の手で配線できる。壊せる。直せる。理解できる。

「ベニテングタケが報告を書いてる」のを見てニヤニヤしたい人は kinoko を使ってくれ。

---

## 📁 ファイル構成

```
kinoko/
├── CLAUDE.md                # 全エージェントの設計書。これが脳
├── deploy.sh                # 起動スクリプト。これが心臓
├── dashboard.md             # 進捗ダッシュボード。霊芝が更新する
├── instructions/
│   ├── reishi.md            # 霊芝の行動ルール
│   └── worker.md            # ワーカー共通ルール
├── queue/
│   ├── command.md           # ユーザー → 霊芝
│   ├── tasks/{worker}.md    # 霊芝 → ワーカー
│   └── reports/{worker}.md  # ワーカー → 霊芝
└── examples/
    └── spinner-verbs.json   # スピナー動詞500個
```

改造したいなら `CLAUDE.md` を読め。通信プロトコル、禁止事項、タスクファイル形式——全部書いてある。kinoko の設計書そのもの 📖

---

## ⚠️ 既知の制限

- **macOS / Linux のみ** — Windows？ 知らん
- **tmux 3.4未満だと見にくい** — `pane-colours` が使えず暗い背景で黒文字が沈む
- **API使用量** — 5体並列。通常の数倍。覚悟しろ
- **コンテキスト消失** — 長時間稼働するとコンパクション（コンテキスト圧縮）が起きる。回復手順は `CLAUDE.md` に書いてあるから、エージェントは自力で復帰する。が、完璧じゃない

---

## 🍄 最後に

kinoko は「マルチエージェントを自分で組んでみたい人」のためのスターターキット。

公式 Agent Teams がある今、自作する実用的な理由は正直少ない。でも全部見えること、全部いじれること、全部理解できること——これは公式ツールじゃ得られない。

```bash
git clone https://github.com/infoHiroki/kinoko.git
cd kinoko
./deploy.sh
```

3分で菌糸ネットワークが動き出す。壊しても俺は知らん 🍄

---

:::message
前回の記事 → [Claude Code マルチエージェント10体で$100溶けたから菌類5体に転生した話](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)
スピナーの話 → [Claude Codeの処理待ちが「般若心経ラップ中」になった日](https://zenn.dev/hirokitakamura/articles/claude-code-spinner-verbs)
:::
