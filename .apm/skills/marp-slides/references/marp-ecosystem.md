# Marp エコシステム リファレンス

Marp でスライドを書き、HTML / PDF / PPTX / 画像へ変換するための実用リファレンス。
公式ドキュメント(marp-core / marp-cli / marp-vscode の README、Marpit ドキュメント)に基づく。

## 目次

1. [Marp エコシステムの構成](#1-marp-エコシステムの構成)
2. [組み込みテーマ(default / gaia / uncover)](#2-組み込みテーマdefault--gaia--uncover)
3. [Marp Core が Marpit に追加する機能](#3-marp-core-が-marpit-に追加する機能)
   - marp: true / math(KaTeX・MathJax)/ 絵文字 / 自動スケーリング / size
4. [よく使う Marpit ディレクティブ](#4-よく使う-marpit-ディレクティブ)
5. [Marp CLI の使い方](#5-marp-cli-の使い方)
   - インストール / 変換コマンド / --theme / --watch / --server / --allow-local-files / 設定ファイル
6. [VS Code 拡張(Marp for VS Code)](#6-vs-code-拡張marp-for-vs-code)
7. [スピーカーノートの書き方](#7-スピーカーノートの書き方)
8. [スライド全体のサンプル](#8-スライド全体のサンプル)

---

## 1. Marp エコシステムの構成

| コンポーネント | 役割 |
|---|---|
| **Marpit** | Markdown からスライドを作る最小限のフレームワーク。ディレクティブや背景画像構文などの基盤 |
| **Marp Core** | Marpit に組み込みテーマ・数式・絵文字・自動スケーリングなどを追加した変換エンジン |
| **Marp CLI** | Marp Core を使って Markdown を HTML / PDF / PPTX / 画像に変換するコマンドラインツール |
| **Marp for VS Code** | VS Code 上でのプレビュー・エクスポート・IntelliSense を提供する拡張機能 |

スライドは通常の Markdown ファイルで、`---`(水平線)がスライドの区切りになる。

---

## 2. 組み込みテーマ(default / gaia / uncover)

テーマはフロントマターまたは HTML コメントのグローバルディレクティブで指定する。

```markdown
---
marp: true
theme: gaia
---
```

または

```markdown
<!-- theme: gaia -->
```

### default

- Marp のデフォルトテーマ。GitHub の Markdown スタイルをベースに、スライド向けに最適化されている
- スライドの内容は常に垂直方向に中央揃えされる
- GitHub markdown CSS の CSS 変数を継承しており、多くの変数は上流(GitHub 側)で定義されている

**サポートクラス:**

| クラス | 効果 |
|---|---|
| `invert` | 反転(ダーク)カラースキームに変更 |

```markdown
<!-- class: invert -->
```

### gaia

- 旧 yhatt/marp のクラシックデザインがベース。azusa-colors の Keynote テンプレートに着想を得ている

**サポートクラス:**

| クラス | 効果 |
|---|---|
| `lead` | スライド内容を中央寄せにする(タイトルスライド向け) |
| `invert` | 反転カラースキーム |
| `gaia` | 追加のカラースキームを適用 |

複数クラスは YAML の配列、またはスペース区切り文字列で併用できる。

```markdown
---
theme: gaia
class:
  - lead
  - invert
---
```

```markdown
<!-- class: lead gaia -->
```

カラースキームは CSS 変数で上書きできる。

```html
<style>
  :root {
    --color-background: #fff;
    --color-foreground: #333;
    --color-highlight: #f96;
    --color-dimmed: #888;
  }
</style>
```

### uncover

- 「simple, minimal, modern」の 3 つのデザインコンセプトを持つ。reveal.js に着想を得ている

**サポートクラス:**

| クラス | 効果 |
|---|---|
| `invert` | 反転カラースキーム |

- 色は background / foreground / highlight / header などの CSS 変数でカスタマイズ可能

---

## 3. Marp Core が Marpit に追加する機能

### marp: true(有効化フラグ)

フロントマターに `marp: true` を書くと、VS Code 拡張などのツールが「この Markdown は Marp スライドである」と認識する。VS Code ではこれが Marp 機能の有効化条件になる。

```markdown
---
marp: true
theme: default
---

# 最初のスライド
```

### math ディレクティブ(数式: KaTeX / MathJax)

Pandoc の Markdown スタイルに従い、インライン数式は `$...$`、ブロック数式は `$$...$$` で書く。

- 使用ライブラリは `math` グローバルディレクティブで宣言できる(`katex` または `mathjax`)
- Marp Core はレンダリング品質の観点から MathJax を推奨(デフォルト)
- ディレクティブの宣言は JS コンストラクタの `math` オプションより優先されるが、コンストラクタ側で無効化されている場合はディレクティブで再有効化できない

```markdown
---
marp: true
math: katex
---

# 数式の例

インライン: $ax^2+bx+c$

$$
\begin{align}
x &= 1+1 \tag{1} \\
  &= 2
\end{align}
$$
```

### 絵文字

- `:smile:` のような絵文字ショートコードと Unicode 絵文字は、twemoji の SVG ベクター画像に変換される

```markdown
Marp は楽しい :+1: 🎉
```

### 自動スケーリング(Auto scaling)

組み込みテーマ(default / gaia / uncover)はすべて、フル機能の自動スケーリングに対応している。

**見出しのフィット(`<!-- fit -->`):** 見出しに `<!-- fit -->` コメントを付けると、スライド幅いっぱいに自動で拡大縮小される。

```markdown
# <!-- fit --> Fitting header
```

**自動縮小ブロック:** コードブロックと KaTeX の数式ブロックは、スライドからはみ出さないよう自動的に縮小される。

### size ディレクティブ(4:3 / 16:9)

組み込みテーマは 2 つのプリセットサイズをサポートする。

| 値 | 解像度 |
|---|---|
| `16:9`(デフォルト) | 1280x720 |
| `4:3` | 960x720 |

```markdown
---
theme: gaia
size: 4:3
---
```

---

## 4. よく使う Marpit ディレクティブ

Marp Core は Marpit ベースなので、Marpit のディレクティブがそのまま使える。

### paginate(ページ番号)

```markdown
<!-- paginate: true -->
```

| 値 | ページ番号表示 | カウントの増加 |
|---|---|---|
| `true` | 表示 | あり |
| `false` | 非表示 | あり |
| `hold` | 表示 | なし |
| `skip` | 非表示 | なし |

### header / footer

複数スライドにわたって同じ内容(タイトルなど)を表示する。内容にはインライン Markdown やインライン画像が使える(ただし `![bg]()` 構文は使えない)。

```markdown
---
header: 'Header content'
footer: 'Footer content'
---
```

### 背景

```markdown
![bg](image.jpg)
```

`backgroundImage` / `backgroundPosition`(デフォルト center)/ `backgroundRepeat`(デフォルト no-repeat)/ `backgroundSize`(デフォルト cover)ディレクティブでも指定できる。

```markdown
<!-- backgroundImage: "linear-gradient(to bottom, #67b8e3, #0288d1)" -->
```

### スポットディレクティブ(そのページだけに適用)

ディレクティブ名に `_` プレフィックスを付けると、現在のページだけに適用される(`_class`、`_paginate`、`_backgroundColor`、`_color` など)。

```markdown
<!-- _class: lead -->
# THE LEADING HEADER
```

タイトルスライドだけページ番号を消す定番パターン:

```markdown
---
marp: true
paginate: true
---

<!-- _paginate: false -->

# タイトルスライド

---

# 2 枚目(ここからページ番号が出る)
```

---

## 5. Marp CLI の使い方

### インストール / 実行

Node.js v18 以上が必要。インストールせず npx で実行するのが手軽。

```bash
# npx(インストール不要)
npx @marp-team/marp-cli@latest slide-deck.md

# npm(プロジェクトローカル)
npm install --save-dev @marp-team/marp-cli

# npm(グローバル)
npm install -g @marp-team/marp-cli

# Homebrew(macOS / Linux)
brew install marp-cli

# Docker
docker pull marpteam/marp-cli
```

このほか Scoop(Windows)、リリースページのスタンドアロンバイナリ(Node.js 不要)がある。

### 変換コマンド例

**注意: PDF / PPTX / 画像への変換には Chrome、Edge、Firefox いずれかのブラウザが必要。**

```bash
# HTML(デフォルト)
marp slide-deck.md
marp slide-deck.md -o output.html

# PDF
marp --pdf slide-deck.md
marp slide-deck.md -o converted.pdf     # 出力拡張子からも形式を判定

# PowerPoint (PPTX)
marp --pptx slide-deck.md
marp slide-deck.md -o converted.pptx

# 全スライドを PNG / JPEG 画像に(複数ファイル)
marp --images png slide-deck.md
marp --images jpeg slide-deck.md

# タイトルスライド(1 枚目)のみ画像化
marp --image png slide-deck.md
marp slide-deck.md -o output.png

# 高解像度画像(スケール指定)
marp slide-deck.md -o title-slide@2x.png --image-scale 2

# スピーカーノートをテキストで出力
marp --notes slide-deck.md
marp slide-deck.md -o output.txt
```

PDF 向けの追加オプション:

```bash
# スピーカーノートを PDF の注釈として埋め込む
marp --pdf --pdf-notes slide-deck.md

# アウトライン(ブックマーク)付き PDF
marp --pdf --pdf-outlines slide-deck.md
marp --pdf-outlines.pages=false slide-deck.md      # ページのアウトラインを無効化
marp --pdf-outlines.headings=false slide-deck.md   # 見出しのアウトラインを無効化
```

PPTX の編集可能版(実験的機能):

```bash
marp --pptx --pptx-editable slide-deck.md
```

- LibreOffice Impress が必要
- 複雑なスタイルではエラーになる可能性あり
- スピーカーノートは非対応

### --theme(カスタム CSS / テーマ指定)

```bash
# 組み込みテーマを指定
marp --theme gaia slide-deck.md

# カスタム CSS ファイルを指定
marp --theme custom-theme.css slide-deck.md

# 複数テーマを登録(Markdown 側の theme ディレクティブで選択)
marp --theme-set theme-a.css theme-b.css theme-c.css -- deck-a.md deck-b.md
marp --theme-set ./themes -- deck.md
```

### --watch / --server / --preview

```bash
# watch: ファイルを監視して自動変換。ブラウザで開いていれば自動リフレッシュ
marp -w slide-deck.md

# server: ディレクトリを http://localhost:8080/ で配信
marp -s ./slides
PORT=5000 marp -s ./slides      # ポート変更

# preview: プレビューウィンドウを開く(watch モード自動有効)
marp -p slide-deck.md
```

サーバーモードではクエリ文字列で変換形式を指定できる。

```
http://localhost:8080/deck-a.md?pdf
```

### --allow-local-files の注意

セキュリティ上の理由から、ブラウザ経由の変換(PDF / PPTX / 画像)ではローカルファイル(ローカル画像など)へのアクセスがデフォルトで禁止されている。ローカル画像を含むスライドを変換するには次のフラグが必要。

```bash
marp --pdf --allow-local-files slide-deck.md
```

**警告: 潜在的なセキュリティリスクがあるため、信頼できる Markdown にのみ使用すること。**

### 設定ファイル

`marp.config.js`、`.marprc`(JSON / YAML)、`package.json` の `marp` セクションに対応。`--config-file path/to/config.js` で明示指定もできる。

```json
// package.json
{
  "marp": {
    "inputDir": "./slides",
    "output": "./public",
    "themeSet": "./themes"
  }
}
```

```yaml
# .marprc.yml
allowLocalFiles: true
pdf: true
```

```javascript
// marp.config.mjs(markdown-it プラグインの追加)
import markdownItContainer from 'markdown-it-container'
export default {
  engine: ({ marp }) => marp.use(markdownItContainer, 'custom'),
}
```

主なオプション: `author` / `description` / `keywords` / `title` / `bespoke.osc` / `bespoke.progress` / `browser` / `jpegQuality` / `parallel`(デフォルト 5)/ `template`(デフォルト bespoke)/ `html` など。

---

## 6. VS Code 拡張(Marp for VS Code)

### 有効化

フロントマターに `marp: true` を書くと Marp 機能が有効になる。ツールバーアイコンの「Toggle Marp feature for current Markdown」でも切り替え可能。

```markdown
---
marp: true
---

# Your slide deck
```

### プレビュー

VS Code 組み込みの Markdown プレビューがそのままスライドプレビューになる。エディタのカーソル位置に対応するスライドがハイライトされる(`markdown.preview.markEditorSelection` で無効化可能)。

### エクスポート

ツールバーの Marp アイコンから「Export slide deck...」を選ぶか、コマンドパレット(F1 / Ctrl+Shift+P)から実行する。

- 対応形式: HTML、PDF、PPTX、PNG / JPEG(最初のスライドのみ)、TXT(ノートのみ)
- PDF / PPTX / 画像のエクスポートには Chrome、Chromium、Edge、Firefox のいずれかが必要(`markdown.marp.browser` / `markdown.marp.browserPath` で制御)

### カスタムテーマ

`.vscode/settings.json` で CSS を登録する。CSS 側には `/* @theme テーマ名 */` コメントで名前を付け、Markdown のフロントマターで `theme: テーマ名` として使う。ローカル CSS は編集時に自動リロードされる。

```javascript
{
  "markdown.marp.themes": [
    "https://example.com/custom-theme.css",
    "./themes/your-theme.css"
  ]
}
```

### その他

- ディレクティブの自動補完・構文ハイライト・ホバーヘルプ・診断(不正なディレクティブや非推奨構文の検出、クイックフィックス)
- スライド単位のアウトラインビューと折りたたみ(`markdown.marp.outlineExtension`)
- Workspace Trust 対応: 信頼されていないワークスペースではエクスポートなどが制限され、HTML 要素は常に無視される(`markdown.marp.html` で制御)
- vscode.dev / github.dev でも動作するが、エクスポートは不可

---

## 7. スピーカーノートの書き方

Markdown 中の **HTML コメントがスピーカーノート(プレゼンターノート)** になる。ただし、ディレクティブ(`<!-- _class: lead -->` など)として解釈されるコメントはノートにはならない。ノートはスライドページごとに収集される。

```markdown
---
marp: true
---

# スライド 1

<!-- これはスライド 1 のスピーカーノートです -->
<!-- 複数のコメントを書くこともできます -->

---

# スライド 2

<!--
複数行の
ノートも書けます
-->
```

ノートの出力先:

- **HTML(bespoke テンプレート)**: プレゼンタービューで表示される
- **PDF**: `marp --pdf --pdf-notes` で PDF の注釈として埋め込み
- **テキスト**: `marp --notes` または `-o output.txt` でノートのみ出力
- **VS Code**: エクスポート形式 TXT でノートのみ出力

---

## 8. スライド全体のサンプル

````markdown
---
marp: true
theme: gaia
size: 16:9
paginate: true
header: 'プロジェクト名'
footer: '© 2026 Example Inc.'
math: mathjax
---

<!-- _class: lead -->
<!-- _paginate: false -->

# <!-- fit --> 発表タイトル

発表者名 / 2026-07-24

<!-- タイトルスライドのノート: 挨拶と自己紹介 -->

---

# アジェンダ

- 背景 :bulb:
- 提案手法
- 評価結果

<!-- 全体の流れを 30 秒で説明する -->

---

<!-- _class: invert -->

# 数式の例

インライン: $E = mc^2$

$$
\frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2
$$

---

# コード例(自動縮小される)

```js
console.log('コードブロックはスライドに収まるよう自動縮小される')
```
````

変換コマンド:

```bash
# ローカル画像を含む場合は --allow-local-files を付けて PDF 化
npx @marp-team/marp-cli@latest --pdf --allow-local-files --pdf-notes deck.md

# 編集しながらプレビュー
npx @marp-team/marp-cli@latest -p deck.md
```
