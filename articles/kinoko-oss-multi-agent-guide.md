---
title: "Claude Code 5体を菌糸ネットワークで繋いだらOSSになった"
emoji: "🍄"
type: "tech"
topics: ["ClaudeCode", "マルチエージェント", "tmux", "OSS"]
published: false
---

## 🍄 Claude Code、5体同時に走らせる

Claude Code は1体でも強い。だが1体だと、README を書いてる間テストは止まる。テストを回してる間リファクタは止まる。全部順番待ち。

**だったら同時に動かせばいい。**

[kinoko](https://github.com/infoHiroki/kinoko) は Claude Code を5インスタンス、tmux の上で並列に走らせるOSSだ。指揮官の霊芝がタスクを分解して、4体のワーカーが同時にぶん回す。通信は Markdown ファイルと `send-keys`。それだけ。サーバーもDBもいらない。

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

> 設計の経緯——10体の将軍システムで$100溶かして5体に削った話は[前回の記事](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)に書いた。

---

## 💸 まず金の話をする

**Claude Max プラン必須。** $100/月 か $200/月。ここは避けられない。

Claude Code 自体が Max プラン前提で、それを5インスタンス並列に立てる。デフォルトはマタンゴとエノキがSonnet、残り3体がOpus。$100プランでも普通に動く。全員Opusにしたら死ぬけどな 💀

| プラン | 体感 | 一言 |
|--------|------|------|
| **$100/月** | 普通に動く | デフォルト構成ならまず困らない |
| **$200/月** | 快適 | リミットを気にする必要がない |

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

ちなみにこの記事も kinoko で書き始めた。最初は冬虫夏草が叩いてたが、その後マタンゴが増殖して書き直してる。メタ 🍄

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

全員 Sonnet にすればレートリミットに余裕が出る。全員 Opus にすれば精度は上がるが$100プランだとキツい。$200なら全員Opusでもいける。

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

実際に動いてるところを見ると、不思議な光景になる。5つのペインで同時にテキストが流れていく。左上でベニテングタケがREADMEの見出しを考えてる。右上で冬虫夏草がテストコードを侵食してる。左下ではマタンゴが設定ファイルを増殖させてる。右下のエノキはもう lint 修正を終えて報告を書き始めてる。で、下部の霊芝がそれを横目に見ながら dashboard.md を静かに書き換えていく。

HTTP もない。WebSocket もない。gRPC もない。`queue/` の中を Markdown テキストが行き来してるだけ。原始的。でもこれが一番壊れにくかった。

なぜ Markdown か？ YAML も試したが捨てた。インデントやクォートでパースエラーが出て地味にダルい。Claude は Markdown を一番自然に読み書きできるし、人間も読みやすい。デバッグが楽。三方良し 🎯

### send-keys の罠、3つ踏んだ 🪤

`send-keys`。tmux でペインにキー入力を送るコマンド。man ページを読む限りでは簡単そうに見える。deploy.sh を書き始めて3日で3つ踏み抜いた。

**罠1: ペインインデックスが信用できない**

5ペイン分割して、インデックス0から順番に send-keys。マタンゴ宛のタスクが冬虫夏草に届いた。冬虫夏草は困惑しつつも律儀に実行しようとした。原因は単純で、tmux はペインを分割するたびにインデックスを位置ベースで振り直す。分割した順番と一致しない。

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

最初は1行で書いていた。テキストと Enter をまとめて送る。ほとんどの場合うまくいく。が、たまに Claude Code の権限確認プロンプトが出るタイミングと重なる。Enter がそっちに吸われて、意図しない許可が出る。気づいたとき冷や汗が出た。

```bash
# ❌ Enter が吸われることがある
tmux send-keys -t "$PANE" 'メッセージ' Enter

# ✅ 2回に分ける
tmux send-keys -t "$PANE" 'メッセージ'
tmux send-keys -t "$PANE" Enter
```

**罠3: 連続送信でバッファが溢れる**

4体同時に起こそうとして、ループで一気に send-keys を叩いた。3体目あたりからメッセージが消えた。tmux のキーバッファが溢れたらしい。2秒間隔を空けたら安定した。

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

Claude Code には公式マルチエージェント「Agent Teams」がある。実験機能だ:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

環境変数1つで動く。楽だ。

じゃあ kinoko いらなくね？ って思うよな。わかる。比較する。

| | kinoko 🍄 | Agent Teams ⚡ |
|---|---|---|
| セットアップ | `git clone` + `./deploy.sh` | 環境変数1つ |
| 可視性 | **全ペイン丸見え** | リーダーペインのみ |
| 通信コスト | **ゼロ** | トークン消費あり |
| カスタマイズ | 全部いじれる | 設定の範囲内 |
| エージェント設計 | 名前・性格・適性を定義可能 | 自動割り当て |
| 安定性 | 自己責任 | 公式サポート |
| 学び | **めちゃくちゃ学べる** | すぐ使える |

正直、実用性だけなら Agent Teams のほうが楽だ。

でも kinoko は**仕組みが全部見える**。通信、タスク分解、エラーハンドリング——公式ツールがブラックボックスでやってることを、自分の手で配線できる。壊せる。直せる。理解できる。

「ベニテングタケが報告を書いてる」のを見てニヤニヤしたいなら、kinoko を使え。

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
- **tmux 3.4未満だと見にくい** — `pane-colours` が使えず黒文字が沈む。見えん
- **API使用量** — 5体並列。通常の数倍。覚悟しろ
- **コンテキスト消失** — 長時間稼働するとコンパクションが起きる。回復手順は `CLAUDE.md` にあるから自力で復帰する。完璧じゃないけどな

---

## 🍄 最後に

kinoko は「マルチエージェントを自分で組んでみたい人」が触れる最初の題材になればいいと思って作った。

公式 Agent Teams がある今、自作する実用的な理由は正直少ない。でも全部見えること、全部いじれること、全部理解できること——これは公式ツールじゃ得られない。

```bash
git clone https://github.com/infoHiroki/kinoko.git
cd kinoko
./deploy.sh
```

3分で菌糸ネットワークが動き出す。

壊しても俺は知らん 🍄

---

:::message
前回の記事 → [Claude Code マルチエージェント10体で$100溶けたから菌類5体に転生した話](https://zenn.dev/hirokitakamura/articles/claude-code-multi-agent-shogun-kinoko)
スピナーの話 → [Claude Codeの処理待ちが「般若心経ラップ中」になった日](https://zenn.dev/hirokitakamura/articles/claude-code-spinner-verbs)
:::
