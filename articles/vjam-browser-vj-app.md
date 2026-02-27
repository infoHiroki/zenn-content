---
title: "スマホでVJがしたいだけなのに何故こんなことになっているのか"
emoji: "🎆"
type: "tech"
topics: ["javascript", "WebAudioAPI", "p5js", "PWA", "個人開発"]
published: true
---

## 発端

パーティーで映像を出したかった。それだけの話である。

音楽が鳴っている。人が踊っている。そこに映像があったらもっといいのにと思った。思ったのだが、VJソフトというものを調べてみると、Resolume Arenaというのが€399する。VDMXというのが$199する。しかもノートPCが要る。ノートPCを持っていくのか。パーティーに。あの重い板を。

馬鹿馬鹿しいと思った。

俺が欲しいのはスマホでURLを開いたら映像が出る、というただそれだけのものであって、何百ドルも払って重い機材を担いでいくような大層なものではない。ないのだが、そういうものが存在しない。ないなら作るしかないではないかと、そういう経緯で作り始めたのがVJamである。

https://vjam.vercel.app

## 何をするものか

スマホのマイクで音楽を拾う。BPMを自動検出する。180種類以上のビジュアルがビートに反応して勝手に動く。以上。

HDMIの変換アダプタでプロジェクターに繋げば、それがVJである。中古のプロジェクターとスマホ1台。それだけでいい。

**デモ動画**: https://www.youtube.com/watch?v=_nrSgubFBL8

## 技術の話

さて、ここからが本題というか、Zennに書く以上は技術的な話をしなければ何をしに来たのかわからないので書く。

### 技術スタック

```
- フロントエンド: Vanilla JS（フレームワークなし、ビルドステップなし）
- 描画: p5.js（Canvas 2D + WEBGL mode）
- 音声分析: Web Audio API（AnalyserNode + FFT）
- PWA: Service Worker（オフライン対応）
- ホスティング: Vercel（静的ファイルのみ）
- 決済: Stripe Payment Link + HMAC ライセンスキー
```

フレームワークなし。サーバーなし。ビルドツールなし。DBなし。静的ファイルをVercelに置いているだけ。サーバーサイドのコードは決済検証用のVercel Function（2ファイル）だけである。

こう書くと随分と潔いようだが、実態としてはフレームワークを使うほどの複雑なUIがなかっただけであって、別に思想があるわけではない。

## ビート検出の話

これが一番面白かった。というか正直に言うと、VJアプリを作りたかったのか、Web Audio APIで遊びたかったのか、もはやよくわからない。半々ぐらいだったと思う。

### マイクから音を取る

```javascript
const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
const ctx = new AudioContext();
const source = ctx.createMediaStreamSource(stream);
const analyser = ctx.createAnalyser();
analyser.fftSize = 2048;
source.connect(analyser);
```

これでマイクの音がFFTにかかる。周波数スペクトルが取れる。ここまでは教科書通りで何も難しくない。

問題はここからで、「ビートを検出する」というのが思ったより厄介なのである。

### ビート検出が厄介な理由

最初はRMSスパイク（音量の急変）だけで検出していた。静かな環境で音楽を流すぶんにはこれで十分動く。しかしクラブのような、常に大音量が鳴り続けている環境だとどうなるか。RMSの平均値がどんどん上がっていって、スパイクとの差が縮まって、ビートが検出できなくなる。

つまり、うるさい場所で使えないVJソフトということになり、それはVJソフトとして根本的に駄目なのではないかという話になる。

そこでスペクトラルフラックスを入れた。

```javascript
// 周波数構成の変化量を検出
let flux = 0;
for (let i = 0; i < spectrum.length; i++) {
    const diff = spectrum[i] - this._prevSpectrum[i];
    if (diff > 0) flux += diff;
}
```

これは何をやっているかというと、音量ではなく音の「中身」が変わったかどうかを見ている。キックドラムが入ると低音域がガッと変わる。スネアが入ると中高音域が変わる。音量が一定でも周波数構成は変化するので、これで検出できる。

結局、3つの経路で判定することにした。

1. **RMSスパイク** — 音量の急変（閾値1.3倍）
2. **スペクトラルフラックス** — 周波数構成の変化量
3. **バスオンセット** — 低音域エネルギーの変化（閾値1.2倍）

どれか1つでも引っかかればビートとする。冗長だが確実である。

### 周波数帯域を分けてプリセットに渡す

音声データをbass/mid/high/trebleの4帯域に分けて、各プリセットに渡す。プリセット側は好きな帯域を好きなように使う。

```javascript
draw(p, audioData) {
    const bass = audioData.bass;       // 低音 → 大きな動き
    const mid = audioData.mid;         // 中音 → 色変化
    const treble = audioData.treble;   // 高音 → 細かいディテール
    const beat = audioData.beat;       // ビート → フラッシュ/爆発
}
```

この設計にしたことで、プリセットを量産しやすくなった。音声データの加工はaudio-analyzerが一手に引き受けて、プリセットは描画だけに集中できる。

## 180種のプリセットをどう管理しているか

全てのプリセットは`BasePreset`を継承している。

```javascript
class FlowField extends BasePreset {
    setup(p) {
        this.field = [];
        p.pixelDensity(1);  // Retinaで4倍GPU負荷になるのを防止
    }

    draw(p, audioData) {
        p.background(0);  // screen blend mode互換のため黒背景必須
    }

    destroy(p) {
        // リソース解放
    }
}
```

`setup` `draw` `destroy`。ライフサイクルはこの3つだけ。シンプルであることには理由があって、180種も作っていると、複雑なインターフェースは自分の首を絞めるだけなのである。

### レイヤー合成の仕組み

ここが設計上のミソで、各プリセットが独自のcanvasを生成して、CSSの`mix-blend-mode`で合成している。

```
canvas (preset A) ─── mix-blend-mode: screen ───┐
canvas (preset B) ─── mix-blend-mode: lighten ──┤→ #container
video element    ─── mix-blend-mode: screen ────┤
text overlay     ─── mix-blend-mode: normal ────┘
```

各プリセットは`background(0)`で黒背景を描画する。`screen`ブレンドモードでは黒が透明になるので、複数のcanvasを重ねても黒い部分は消えて、明るい部分だけが合成される。

逆に言うと、`multiply`や`overlay`は使えない。黒で上書きされて下のレイヤーが見えなくなる。この制約に気づくまでに結構な時間を使った。

## スマホで60fps出すための戦い

ここからは地味だが重要な話をする。パソコンで動くのは当たり前で、スマホで60fps出すのが本当にきつかった。

### pixelDensity(1) — これだけでGPU負荷が1/4になる

```javascript
p.pixelDensity(1);
```

p5.jsのデフォルトはRetinaディスプレイだと2。ピクセル数4倍。180個のプリセット全てに`pixelDensity(1)`を入れた。VJなので多少荒くても誰も気にしない。

### loadPixels禁止

```javascript
// ❌ フル解像度ピクセル操作 — モバイルで致命的
p.loadPixels();
for (let i = 0; i < p.pixels.length; i += 4) { ... }

// ✅ 描画プリミティブで代替
p.ellipse(x, y, r);
p.rect(x, y, w, h);
```

全画面のピクセル操作はモバイルでは論外である。4つのプリセットがこれをやっていたので全部書き直した。

### スプレッド演算子を殺す

```javascript
// ❌ 毎フレームGCオブジェクト生成
particles = particles.map(p => ({...p, x: p.x + p.vx}));

// ✅ in-place mutation
for (const p of particles) { p.x += p.vx; }
```

60fpsということは16msごとにdrawが呼ばれるということで、そこでスプレッド演算子を使うと毎秒数万のオブジェクトがGCに回る。Immutableだ関数型だという話はわかるが、毎フレーム新しいオブジェクトを作っている場合ではない。

### spliceを殺す

```javascript
// ❌ O(n²)
particles = particles.filter(p => p.life > 0);

// ✅ O(n) write-indexパターン
let w = 0;
for (let i = 0; i < particles.length; i++) {
    if (particles[i].life > 0) particles[w++] = particles[i];
}
particles.length = w;
```

`filter`は毎回新しい配列を作る。`splice`はO(n)のシフトが走る。パーティクルが3000個あるとフレームバジェットを食い潰す。write-indexパターンはallocate zeroで済む。

こういう、書いていて「俺は何をやっているんだ」と思うような最適化を延々とやった。

## 決済: DB不要のHMACライセンスキー

個人開発でDBの面倒を見たくなかったので、HMACベースのライセンスキーにした。

```
支払い → Stripe Checkout → リダイレクト(?session_id=xxx)
  → Vercel Function(session_id検証 → HMACキー生成)
  → localStorage保存
```

キー形式: `VJAM-{timestamp_base36}-{hmac_16chars}`

```javascript
// 検証はHMAC再計算するだけ。外部API不要。
const expectedHmac = createHmac('sha256', SECRET)
    .update(timestamp).digest('hex').slice(0, 16);
const valid = (providedHmac === expectedHmac);
```

DBなし。Webhookなし。npm依存なし。検証時に外部APIを叩く必要もない。タイムスタンプとシークレットからHMACを再計算して一致すればOK。

これで十分かと言われると十分である。少なくとも個人開発の規模では。

## 数字

- コード: 約15,000行（テスト含む）
- プリセット: 180種（BG） + 10種（3D）= 190種
- 開発期間: 約1ヶ月（ソロ）
- ファイルサイズ: HTML/CSS/JS合計 約500KB（gzip後）
- テスト: 100件
- 価格: $27 買い切り（Free版は36プリセット）

## おわりに

「フレームワークなし、サーバーなし、DBなし」で、スマホのブラウザだけで動くVJアプリが作れた。Web Audio APIとCanvas APIの組み合わせは、リアルタイム音声ビジュアライゼーションにかなり実用的である。

半分は技術的に面白そうだったから作ったのだが、実際にプロジェクターに映してみると、それなりにちゃんとVJっぽくなるので自分でも少し驚いている。

試してみてください。

https://vjam.vercel.app

質問・フィードバックは X([@VJam_app](https://x.com/VJam_app)) まで。
