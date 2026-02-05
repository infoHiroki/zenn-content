# Zenn コンテンツ方針

## 文体

**ブコウスキー + 町田康 + 中島らも**を混ぜた文体で書く。

### 特徴
- **生々しい・直接的**（ブコウスキー）
- **リズミカル・パンク**（町田康）
- **ユーモア・自虐・温かみ**（中島らも）

### 具体的なルール
- 一人称は「俺」
- 短い文を多用
- 自虐ネタOK
- 読者に語りかける口調（「〜だろ？」「〜してくれ」）
- 絵文字はふんだんに使用（見出し・本文・テーブル内・文末すべてに）
- 専門用語は使うが、難しく書かない
- 画像・GIFは公式リポジトリのREADMEから探して積極的に使う（OSSならMITライセンスで引用OK）

### 例
```markdown
# ❌ NG（硬い）
ジャーナリングツールの選定において、AIとの連携性は重要な評価軸となります。

# ✅ OK（この文体）
日記アプリ、何個試した？俺は数えきれない。全部ダメだった。
```

---

## SEO重複対策

### 背景
同じ記事を自社サイト（infohiroki.com）とZennの両方に掲載している。

### 方針
**両方に掲載するが、30%以上の差別化を行う**

| 項目 | Zenn | 自社サイト |
|------|------|-----------|
| **文体** | カジュアル（上記の文体） | 説明的・ビジネス寄り |
| **タイトル** | バズ狙い・キャッチー | 内容が分かる説明的タイトル |
| **CTA** | なし | あり（LINE相談・サービスページへの導線） |
| **絵文字** | 多め | 控えめ |
| **ターゲット** | 開発者・技術者 | ビジネスオーナー・導入検討者 |

### SEOへの影響
- Googleは重複コンテンツをペナルティしない（どちらかを優先表示するだけ）
- 30%以上差別化すれば、別コンテンツとして認識される可能性が高い
- Zennのドメインパワーが強いため、検索ではZenn版が優先される可能性あり

---

## 記事一覧

| トピック | Zennタイトル | slug | 公開状態 |
|---------|-------------|------|---------|
| ジャーナリング | ジャーナリングツールの変遷 | journaling-tool-evolution | ✅ 公開 |
| Tower | tower | tower-diary-habit-system | 🚧 下書き |
| Zellij | Zellij完全ガイド | zellij-complete-guide | ✅ 公開 |
| LazyGit | LazyGitを知った日、俺はファミコン少年に戻った | lazygit-cheatsheet | ✅ 公開 |
| Claude Code | ChatGPTコピペ地獄を抜けたら、そこはClaude Codeだった | claude-code-changed-everything | ✅ 公開 |
| Vibe Coding | Anthropicの男がコードを読むなと言った日 | vibe-coding-production | ✅ 公開 |

---

## 運用ルール

### 新規記事作成時
1. Zenn版をこの文体で先に書く
2. 自社サイト版は説明的な文体で書く（CTAを末尾に追加）
3. タイトルを差別化する

### 更新時
- Zenn版と自社サイト版を同時に更新する必要はない
- 内容の差別化が維持されていればOK

---

## infohiroki.com → Zenn 移植手順

既存記事をZennに移植するときの手順。

### 1. 元記事を読む
```
src/lib/content/blog/{slug}.md
```

### 2. Zenn版を書く
```
~/Dev/zenn-content/articles/{slug}.md
```
- frontmatter: `title` / `emoji` / `type: "tech"` / `topics` / `published: true`
- 文体: 上記の文体ルールに従う
- タイトル: 元記事と変える（バズ狙い・キャッチー）
- 構成: 網羅型ガイド → 体験ベース読み物に再構成
- 絵文字: ふんだんに
- 画像/GIF: 公式リポジトリから積極的に引用
- CTA: 入れない
- 30%以上差別化する

### 3. 記事一覧を更新（両方）
- `~/Dev/zenn-content/CONTENT_POLICY.md` の記事一覧テーブル
- `~/Dev/infohiroki-svelte/CONTENT_POLICY.md` の記事一覧テーブル

### 4. コミット & プッシュ（両方のリポジトリ）
```bash
# zenn-content
cd ~/Dev/zenn-content
git add articles/{slug}.md CONTENT_POLICY.md
git commit -m "📝 {記事名}を公開"
git push

# infohiroki-svelte
cd ~/Dev/infohiroki-svelte
git add CONTENT_POLICY.md
git commit -m "📝 CONTENT_POLICY: {記事名}のZenn対応を記録"
git push
```

---

## 参考

- 自社サイト側の方針: infohiroki-svelte/CONTENT_POLICY.md
