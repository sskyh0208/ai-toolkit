# 実在するLTスライド用カスタムテーマの解剖

su8ru（すばる）氏の公開リポジトリ [su8ru/slides](https://github.com/su8ru/slides) を題材に、実運用されている Marp カスタムテーマ + カスタムエンジンの構成を分解する。同氏の Zenn 記事「[LT スライド高速作成技術](https://zenn.dev/huitgroup/articles/marp-cli-lt-slide)」の実装そのものである。

以下のコードはすべて 2026-07-24 時点の `main` ブランチから raw ファイルとして取得した実物の引用（全文または抜粋）。推測で補ったコードはない。

## 目次

1. [リポジトリ構成](#1-リポジトリ構成)
2. [設定ファイル `.marprc`](#2-設定ファイル-marprc)
3. [package.json とビルドコマンド](#3-packagejson-とビルドコマンド)
4. [テーマ CSS の解剖（su8ru.css）](#4-テーマ-css-の解剖su8rucss)
   - 4.1 テーマ宣言と `@import 'default'`
   - 4.2 OKLCH による色パレット設計
   - 4.3 Marp Core 既定テーマの CSS 変数を上書きする
   - 4.4 見出し（グラデーション文字）
   - 4.5 リスト・コード・ページ番号
   - 4.6 ユーティリティクラス（2カラム / 3カラム / flex）
   - 4.7 小技: QRコード配置・脚注・キャプション
5. [OKLCH 色空間を使うメリット](#5-oklch-色空間を使うメリット)
6. [カスタムエンジン（engine.mjs + markdown-it プラグイン）](#6-カスタムエンジンenginemjs--markdown-it-プラグイン)
   - 6.1 実物の engine.mjs
   - 6.2 独自プラグイン talk-metadata の仕組み
   - 6.3 最小構成例と `--engine` の使い方
7. [スライド Markdown 側での使われ方](#7-スライド-markdown-側での使われ方)
8. [取得できなかった情報・注意点](#8-取得できなかった情報注意点)

---

## 1. リポジトリ構成

GitHub API（`git/trees/main?recursive=1`）で取得したファイル一覧の要約:

```
slides/
├── .marprc                    # marp-cli の設定ファイル
├── engine.mjs                 # カスタムエンジン(markdown-it プラグイン注入)
├── package.json               # ビルド/プレビュー/デプロイの scripts
├── plopfile.mjs               # plop による新規スライドの雛形生成
├── wrangler.jsonc             # Cloudflare Workers(静的アセット配信)設定
├── lib/
│   ├── talk-metadata.mjs      # 自作 markdown-it プラグイン
│   └── talk-metadata.test.mjs # そのテスト(vitest)
├── src/
│   ├── index.md               # トップページ(スライド一覧)
│   ├── themes/
│   │   ├── su8ru.css          # メインテーマ
│   │   └── su8ru-white.css    # 旧/別バージョンのテーマ
│   └── <YYMMDD-slug>/         # 発表1本 = 1ディレクトリ
│       ├── index.md
│       └── images/…
└── templates/slide/index.md.hbs  # plop 用テンプレート
```

設計上のポイント:

- **「1発表 = 1ディレクトリ」**。`src/240830-seb03/index.md` のように日付+スラッグで切り、画像は各ディレクトリの `images/` に同居させる。ビルド後の URL がそのまま `slides.example.com/240830-seb03/` になる。
- **テーマは `src/themes/` にまとめて置き、`.marprc` の `themeSet` で一括登録**。スライド側は front-matter に `theme: su8ru` と書くだけでよい。
- **雛形生成（`plop slide`）とデプロイ（wrangler）まで含めて「LTを高速に量産する」パイプライン**になっている。`wrangler.jsonc` は `dist/` を静的アセットとして配信するだけの最小設定:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "slides",
  "compatibility_date": "2026-04-20",
  "assets": { "directory": "./dist" }
}
```

## 2. 設定ファイル `.marprc`

全文（5行しかない）:

```yaml
themeSet: ./src/themes
inputDir: ./src
output: ./dist
lang: ja
html: true
```

- `themeSet`: ディレクトリを指定すると中の CSS がすべてテーマとして登録される。テーマを増やしてもコマンドは変わらない。
- `inputDir` / `output`: `src/**/*.md` → `dist/**/*.html` の変換を宣言。おかげで `package.json` 側のコマンドが `marp --engine ./engine.mjs` だけで済む。
- `html: true`: Markdown 内の生 HTML（`<div class="flex">` や `<style scoped>`）を許可する。ユーティリティクラス運用の前提。

## 3. package.json とビルドコマンド

`scripts` と devDependencies（実物）:

```json
{
  "scripts": {
    "dev": "marp --engine ./engine.mjs -s src",
    "build:clean": "rimraf dist",
    "build:html": "marp --engine ./engine.mjs",
    "build:ogimage": "marp --engine ./engine.mjs --image jpeg --allow-local-files",
    "build:assets": "cpx \"src/**/images/**/*\" dist",
    "build": "pnpm run build:clean && pnpm run build:html && pnpm run build:ogimage && pnpm run build:assets",
    "cf:dev": "wrangler dev",
    "cf:deploy": "pnpm run build && wrangler deploy",
    "new:slide": "plop slide",
    "test": "vitest run"
  },
  "devDependencies": {
    "@marp-team/marp-cli": "^4.3.1",
    "@shikijs/markdown-it": "^4.0.2",
    "cpx2": "^8.0.2",
    "js-yaml": "^4.2.0",
    "plop": "^4.0.5",
    "rimraf": "^6.1.3",
    "vitest": "^4.1.8",
    "wrangler": "^4.83.0"
  }
}
```

読みどころ:

- **`dev`**: `marp -s src` はプレビューサーバーモード。ここでも `--engine ./engine.mjs` を付けているので、開発中もコードハイライトや独自プラグインが本番と同じ挙動になる。
- **`build:html` にオプションがほぼない**のは `.marprc` に入出力設定を寄せているため。CLI 引数は「エンジン差し替え」のような役割の違いだけを担う。
- **`build:ogimage`**: 同じ Markdown を `--image jpeg` で1枚目だけ画像化し、`index.jpg` を生成する。これを OGP 画像として使う（後述の talk-metadata プラグインが `image:` ディレクティブに `…/index.jpg` を差し込む）。`--allow-local-files` はローカル画像を含むスライドの画像化に必須。
- **`build:assets`**: marp-cli は Markdown しか変換しないので、`images/` は `cpx` で `dist/` に別途コピーする。

## 4. テーマ CSS の解剖（su8ru.css）

ファイルは `src/themes/su8ru.css`、全147行。以下、意図が読み取れる単位で全文を分割して引用する。

### 4.1 テーマ宣言と `@import 'default'`

```css
/* @theme su8ru */

@import 'default';
@import url('https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,100..900&family=JetBrains+Mono:wght@100..800&family=Zen+Kaku+Gothic+New:wght@500;700&display=swap');
```

- 先頭コメントの `/* @theme su8ru */` が Marpit のテーマ登録名。front-matter の `theme: su8ru` と対応する。
- **ゼロから書かず `@import 'default'` で Marp Core の default テーマを継承**し、差分だけ書く。code ブロックの余白、テーブル、自動スケーリングなどの面倒な部分をタダで手に入れる、実務的に最重要のテクニック。
- フォントは Google Fonts の可変フォント（欧文 Inter / コード JetBrains Mono / 和文 Zen Kaku Gothic New）。オフライン発表なら埋め込みやローカル配信への差し替えを検討する点には注意。

### 4.2 OKLCH による色パレット設計

```css
:root {
  --su8ru-main: oklch(0.6 0.08 245);
  --su8ru-accent: oklch(0.75 0.12 245);
  --su8ru-accent2: oklch(0.75 0.12 185);
  --su8ru-white: oklch(0.95 0.01 245);
  --su8ru-gray: oklch(0.55 0.04 245);
  --su8ru-black: oklch(0.39 0.03 245);
}
```

パレット全6色がすべて `oklch(明度 彩度 色相)` で、設計ルールが数値にそのまま現れている:

- **色相をほぼ 245（青）に固定**し、彩度と明度だけを振ってモノトーン系パレットを作る。「白」「黒」「グレー」まで色相245に寄せているので、純白・純黒でなくほんのり青みがかった統一感が出る。
- アクセント2色（`--su8ru-accent` / `--su8ru-accent2`）は **明度 0.75・彩度 0.12 を完全に揃え、色相だけ 245→185 に変えている**。後述の h1 グラデーションで並べたとき、明るさが揃っているためムラなく見える。
- 命名は `main / accent / white / gray / black` という役割ベース。色替えはこの6行を書き換えるだけで済む。

### 4.3 Marp Core 既定テーマの CSS 変数を上書きする

```css
section {
  font-family: Inter, 'Zen Kaku Gothic New', sans-serif;
  --fontStack-monospace: 'JetBrains Mono', monospace;
  font-weight: 500;

  background: var(--su8ru-white);
  color: var(--su8ru-black);

  border-radius: 1rem;

  --fgColor-accent: var(--su8ru-main);
  --fgColor-default: var(--su8ru-black);
  --fgColor-muted: var(--su8ru-gray);
  --header-footer-color: var(--su8ru-gray);
  --paginate-color: var(--su8ru-gray);
}
```

- Marpit ではスライド1枚 = `section` 要素。ここが「ページ全体」のスタイル起点になる。
- **`--fgColor-accent` / `--fgColor-default` / `--fgColor-muted` / `--fontStack-monospace` は Marp Core v4 の default テーマ（GitHub Primer 由来）が参照する変数**。default を継承しているからこそ、リンク色やコード用フォントを個別セレクタで殴らずに変数上書き1行で差し替えられる。`--header-footer-color` / `--paginate-color` も同様に default テーマ側のヘッダー・フッター・ページ番号色の変数。
- **自作パレット（`--su8ru-*`）→ 既製テーマの変数（`--fgColor-*`）へ流し込む2層構造**が、この CSS の変数設計の肝。パレット層とテーマ適用層が分離しているため、色変更が波及しやすい。
- `border-radius: 1rem` はスライド自体の角丸。HTML 表示時にカードっぽく見せる遊び。

### 4.4 見出し（グラデーション文字）

```css
h1 {
  background-image: linear-gradient(90deg,
      var(--su8ru-accent) 0%,
      var(--su8ru-accent2) 80%);
  background-clip: text;
  color: transparent;
  margin-top: 1rem;
  margin-bottom: 0;
  line-height: 1.6;
  font-size: 1.6rem;
}

img+h1 {
  margin-top: 0;
}

h2 {
  color: var(--su8ru-main);
  margin-top: 1rem;
  margin-bottom: 0;
  font-size: 1.2rem;
}
```

- `background-clip: text` + `color: transparent` で **h1 をグラデーション文字**にする定番テクニック。グラデーションの両端が 4.2 の「明度・彩度を揃えた2色」なので破綻しない。
- `img+h1 { margin-top: 0 }` のような**隣接セレクタでの余白調整**が随所にある。Markdown から生成される DOM は構造が予測できるので、クラスを増やさず「画像の直後の h1」を狙い撃ちできる。
- Zenn 記事で本人が述べている通り、本文は h2/h3 を主役に大きめの文字で運用する設計。

### 4.5 リスト・コード・ページ番号

```css
p,
ul,
ol {
  margin: 1rem 0;
}

pre,
blockquote {
  margin: 1rem 0;
}

li+li,
li>ul {
  margin-top: 0.4rem;
}

li::marker {
  color: var(--su8ru-gray);
}

section::after {
  content: attr(data-marpit-pagination) ' / ' attr(data-marpit-pagination-total);
}

strong {
  font-weight: 700;
}

pre {
  font-size: 0.7rem;
  line-height: 1.5;
  font-weight: 500;
}

img {
  background: none !important;
}
```

- `li::marker` でリストの行頭記号だけグレーに落とし、本文テキストを立たせる。地味だが視認性への効きが大きい。
- **ページ番号は `section::after` の `content` を上書き**して「現在 / 総数」形式（`3 / 12`）にする。`data-marpit-pagination` / `data-marpit-pagination-total` は Marpit が `paginate: true` のとき各 section に付与する属性。
- `pre { font-size: 0.7rem }` でコードブロックを本文より確実に小さくし、1画面に収まる行数を稼ぐ。
- `img { background: none !important }` は、default テーマ（GitHub スタイル）が画像に敷く背景色を殺して透過 PNG をきれいに見せるための上書き。

### 4.6 ユーティリティクラス（2カラム / 3カラム / flex）

```css
.col2 {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
}

.col3 {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
}

.col2>* {
  margin: 0;
}

.flex {
  display: flex;
  gap: 1rem;
}
```

- **テーマ側にレイアウト用クラスを用意しておき、Markdown 側では `<div class="col2">…</div>` で囲むだけ**にするパターン。`.marprc` の `html: true` が前提。
- `minmax(0, 1fr)` がミソ。素の `1fr` だと中身（長いコードや画像）の最小コンテンツ幅で列幅が崩れるが、`minmax(0, 1fr)` なら必ず等分になる。
- `.col2>* { margin: 0 }` で、グリッド内に入った p や ul の既定マージンをリセットし、`gap` に間隔管理を一本化する。
- 実際のスライド（`src/260603-workers/index.md`）での使用例:

```markdown
<div class="flex">

![h:500](images/search.png)
*新 シラバス検索画面（開発中）*

![h:500](images/timetable.png)
*時間割画面*

</div>
```

HTML タグの前後に空行を入れることで、内側を Markdown として解釈させている。

### 4.7 小技: QRコード配置・脚注・キャプション

```css
.qrcode {
  display: block;
  position: absolute;
  right: 32px;
  bottom: 70px;
}

blockquote>blockquote>blockquote {
  font-size: 50%;
  font-weight: 400;
  padding: 0;
  padding-top: 0.25rem;
  margin: 0;
  border: 0;
  border-top: 0.01rem solid var(--su8ru-gray);
  position: absolute;
  width: 800px;
  bottom: 80px;
  left: 80px;

  &>ol>li {
    margin-top: 0;
  }
}

s {
  opacity: 0.7;
}

br+em {
  font-size: 0.8rem;
  color: var(--su8ru-gray);
}
```

- `.qrcode`: スライド右下に QR コード画像を絶対配置するためのクラス。`section` は `position` の基準になるので `absolute` がスライド内で完結する。
- **`blockquote>blockquote>blockquote`（3重引用）を「脚注・出典欄」に転用**している。Markdown 側で `>>> 出典: …` と書くだけで、ページ下部に罫線付きの小さな注釈が出る。専用構文のない Marp で脚注を実現する発想の転換で、CSS ネスト（`&>ol>li`）も使っている。
- `br+em`: 「`<br />` 直後の斜体」= 画像キャプション、という自分ルールをセレクタ化したもの（4.6 の使用例にある `*時間割画面*` がこれ）。クラスを書かずに済ませるための「Markdown の書き方とセレクタの対応付け」の好例。

### 補足: もう1つのテーマ su8ru-white.css

`src/themes/su8ru-white.css` は旧バージョンとみられる簡素版。構造は同じ（`@import 'default'` + OKLCH パレット + 変数上書き）で、2カラムのクラス名が `.columns`、パレットは色相 240 の4色:

```css
:root {
  --su8ru-main: oklch(0.65 0.05 240);
  --su8ru-dark: oklch(0.5 0.05 240);
  --su8ru-gray: oklch(0.6 0.03 240);
  --su8ru-black: oklch(0.25 0.02 240);
}
```

同一手法で作られたテーマを2枚並べると、「色相固定・明度と彩度で階調を作る」パターンが再現性のある設計手順だと分かる。

## 5. OKLCH 色空間を使うメリット

su8ru.css の実例から読み取れる利点:

1. **知覚的に均一な明度（L）**: `oklch(L C H)` の L は人間の見た目の明るさにほぼ比例する。HSL の「lightness」は色相によって見た目の明るさがバラつく（同じ L 50% でも黄色は明るく青は暗い）が、OKLCH なら **L を揃える = 見た目の明るさが揃う**。su8ru.css のアクセント2色が `0.75 0.12` を共有し色相だけ違うのは、まさにこれを利用したペア作り。
2. **パレットの拡張が算術でできる**: 「もう1段暗いバリエーションが欲しい」→ L を 0.1 下げるだけ、「彩度を抑えたい」→ C を下げるだけ、と数値操作が意図と1対1で対応する。16進カラーコードの並びからは不可能な操作性。
3. **グラデーションのムラ防止**: 明度の揃った2色を端点にすれば、途中で暗く沈むグラデーションになりにくい（4.4 の h1）。
4. **調整ツール**: 著者は https://oklch.com/ で色を調整したと Zenn 記事で述べている。L/C/H をスライダーで動かしながら sRGB 範囲内かも確認できる。
5. ブラウザ対応は 2023 年以降の主要ブラウザで安定。Marp のプレビュー/HTML 出力も PDF 変換（Chromium 経由）も問題なく通る。

## 6. カスタムエンジン（engine.mjs + markdown-it プラグイン）

### 6.1 実物の engine.mjs

リポジトリルートの `engine.mjs`、全文11行:

```js
import Shiki from '@shikijs/markdown-it';
import { talkMetadataPlugin } from './lib/talk-metadata.mjs';

const mdItShiki = await Shiki({
  themes: {
    dark: 'nord',
    light: 'nord',
  },
});

export default ({ marp }) => marp.use(talkMetadataPlugin).use(mdItShiki);
```

ポイント:

- marp-cli の `--engine` に渡すモジュールは、**`{ marp }`（構築済みの Marp Core インスタンス）を受け取り、拡張後のインスタンスを返す関数を default export** すればよい。`marp.use()` は markdown-it の `.use()` がそのまま生えているもので、markdown-it プラグインを何でも注入できる。
- トップレベル `await` で Shiki（シンタックスハイライタ）を初期化している。**Marp 標準の highlight.js では物足りないコードハイライトを Shiki の `nord` テーマに差し替える**のが導入動機（Zenn 記事いわく「CSS だけではどうにもならないこともあります。例えばコードブロックのカラー」）。dark/light 両方に `nord` を指定しているのは、テーマ側の配色が固定なのでモード分岐を実質無効化するため。

### 6.2 独自プラグイン talk-metadata の仕組み

`lib/talk-metadata.mjs`（164行、vitest のテスト付き）。核心部分のみ引用:

```js
export const talkMetadataPlugin = (md) => {
  md.core.ruler.before('marpit_directives_front_matter', 'talk_metadata', (state) => {
    state.src = transformTalkMetadata(state.src);
  });
};
```

- markdown-it の **core ruler に、Marpit が front-matter を解釈するルール（`marpit_directives_front_matter`）より前に割り込ませ、`state.src`（Markdown 原文）を書き換える**プリプロセッサ型プラグイン。トークン操作をせず文字列変換に徹しているのでテストもしやすい。
- 何をするか: front-matter に

  ```yaml
  talk:
    slug: "260603-workers"
    date: "2026-06-03"
    event: "Cloudflare Workers Tech Talks in Hokkaido #2"
  ```

  と書いておくと、
  1. `slug/date/event` から公開 URL（`https://slides.su8ru.dev/<slug>`）、短縮 URL、OGP 画像 URL（`<canonical>/index.jpg`）、`date | event` 形式のラベルを組み立て、
  2. `description:` `footer:` `image:` `url:` の Marp ディレクティブを HTML コメントとして本文冒頭に自動挿入し（`directiveComment()`）、
  3. 本文中の `{{ talk.label }}` や `{{ urls.short }}` といった **Mustache 風プレースホルダを実値に置換**する（`replacePlaceholders()`）。

- つまり「イベント名と日付を1回書けば、フッター・OGP・自己紹介ページの URL 表記まで全部埋まる」仕組み。3章の `build:ogimage`（`index.jpg` 生成）とここの `image` URL が対になっている。

### 6.3 最小構成例と `--engine` の使い方

実物から余分を削った最小構成（Shiki だけ入れる場合）:

```js
// engine.mjs
import Shiki from '@shikijs/markdown-it';

const shiki = await Shiki({
  themes: { light: 'nord', dark: 'nord' },
});

export default ({ marp }) => marp.use(shiki);
```

自作プラグインを足す場合の骨格（talk-metadata と同じ「本文前処理」パターン）:

```js
// engine.mjs
const myPlugin = (md) => {
  md.core.ruler.before('marpit_directives_front_matter', 'my_rule', (state) => {
    state.src = state.src.replaceAll('{{year}}', String(new Date().getFullYear()));
  });
};

export default ({ marp }) => marp.use(myPlugin);
```

CLI からの使い方（su8ru/slides の実コマンド）:

```bash
# プレビュー(ウォッチサーバー)
marp --engine ./engine.mjs -s src

# HTML ビルド(入出力は .marprc の inputDir/output に従う)
marp --engine ./engine.mjs

# 1枚目を JPEG 化(OGP 画像用)
marp --engine ./engine.mjs --image jpeg --allow-local-files
```

`--engine` は `.marprc` に `engine: ./engine.mjs` と書いて省略することもできる（このリポジトリは CLI 引数側で指定する流儀）。必要な依存（`@shikijs/markdown-it` など）は自分の `package.json` に入れる。

## 7. スライド Markdown 側での使われ方

新規スライドの雛形 `templates/slide/index.md.hbs`（plop が `pnpm new:slide` で展開する）の冒頭:

```markdown
---
marp: true
paginate: true
theme: su8ru

title: {{{json title}}}
author: "すばる / su8ru"
talk:
  slug: {{{json slug}}}
  date: {{{json date}}}
  event: {{{json event}}}
---

# {{ title }}

<style scoped>
  .profile-icon {
    width: 90px;
    float: left;
    margin-right: 16px;
    mix-blend-mode: multiply;
  }
</style>

<img src="https://images.su8ru.dev/ichika.png" class="profile-icon" width="90px" height="90px" />

### すばる / su8ru

<br />

{{ talk.label }}

{{ urls.short }}
```

（`{{{json …}}}` は plop/Handlebars の展開、`{{ talk.label }}` `{{ urls.short }}` は 6.2 のプラグインが実行時に置換するプレースホルダ。二段構えになっている点に注意。）

役割分担がきれいに層になっている:

| 層 | 担当 |
|---|---|
| テーマ CSS（su8ru.css） | 全スライド共通の見た目・ユーティリティクラス |
| engine.mjs + プラグイン | 全スライド共通の変換ロジック（ハイライト、メタデータ展開） |
| `<style scoped>` | そのスライド1枚だけの微調整（プロフィールアイコンの絶対配置など） |
| plop テンプレート | 毎回同じ自己紹介ページなどの定型文 |

実発表ファイル（`src/260603-workers/index.md`）でも、front-matter は `theme: su8ru` + `talk:` ブロックのみ、レイアウト調整はテーマのクラス（`.flex`）と `<style scoped>` で行っており、テンプレート通りの運用が確認できた。なお `transition: fade 0.3s` ディレクティブも使っている（Marp CLI の HTML 出力でのページ遷移アニメーション）。

## 8. 取得できなかった情報・注意点

- **`plopfile.mjs` の中身は未取得**（雛形生成の詳細ロジックは本文の推測を避け、テンプレートの存在確認まで）。`lib/talk-metadata.test.mjs`、`src/index.md`（トップページ）も未読。
- `--fgColor-*` / `--header-footer-color` / `--paginate-color` が Marp Core default テーマの変数である、という説明は Marp Core v4 の一般知識に基づく解釈で、このリポジトリ内に定義箇所があるわけではない（`@import 'default'` 先の marp-core 側にある）。
- `su8ru-white.css` が「旧バージョン」というのは命名とコミット状況からの推定。リポジトリ内に明記はない。
- Zenn 記事の本文全文は WebFetch の要約経由で確認したため、記事中コード片の逐語引用はしていない（本ドキュメントのコードはすべて GitHub raw から取得したもの）。
- 取得日は 2026-07-24。リポジトリは活発に更新されているため、行番号や内容は変わりうる。
