# Marpit テーマCSS リファレンス

Marpit公式ドキュメント(theme-css / inline-svg / usage / directives / introduction)に基づく、
カスタムテーマ作成と Markdown 経由のスタイル調整のためのリファレンス。

## 目次

1. [テーマCSSの基本構造](#1-テーマcssの基本構造)
   - [HTML構造: スライド = `<section>`](#html構造-スライド--section)
   - [`@theme` メタデータ(必須)](#theme-メタデータ必須)
   - [基本的なテーマの書き方](#基本的なテーマの書き方)
   - [`:root` 疑似クラスセレクタ](#root-疑似クラスセレクタ)
2. [スライドサイズ](#2-スライドサイズ)
3. [ディレクティブで注入される要素のスタイリング](#3-ディレクティブで注入される要素のスタイリング)
   - [ページ番号: `section::after`](#ページ番号-sectionafter)
   - [ヘッダー・フッター: `header` / `footer`](#ヘッダーフッター-header--footer)
4. [テーマの継承(カスタマイズテーマ)](#4-テーマの継承カスタマイズテーマ)
   - [`@import` ルール](#import-ルール)
   - [`@import-theme` ルール](#import-theme-ルール)
5. [Markdown内でのスタイル調整](#5-markdown内でのスタイル調整)
   - [`<style>` 要素](#style-要素)
   - [`style` グローバルディレクティブ](#style-グローバルディレクティブ)
   - [`<style scoped>`(スコープ付きスタイル)](#style-scopedスコープ付きスタイル)
6. [インラインSVGモード(実験的機能)](#6-インラインsvgモード実験的機能)
7. [Marpit APIでのテーマの登録と利用](#7-marpit-apiでのテーマの登録と利用)

---

## 1. テーマCSSの基本構造

### HTML構造: スライド = `<section>`

Marpitの基本的な考え方は、**各スライドページが `<section>` 要素に対応する**こと
(reveal.jsのマークアップと同じ)。テーマ作者が知るべきことは、
「`<section>` が各スライドページのビューポートとして使われる」という点だけ。

```html
<section><h1>First page</h1></section>
<section><h1>Second page</h1></section>
```

変換時、MarpitはCSSセレクタをコンテナ要素のセレクタで自動的にラップしてスコープするが、
テーマ作者はこの処理を意識する必要はない。

Marpitには**定義済みクラスやmixinは一切ない**。純粋なCSSでHTML要素をスタイリングするだけでよい。
これが他のスライドフレームワークとの違いの核心。

### `@theme` メタデータ(必須)

**`@theme` メタデータはMarpitで常に必須。** CSSコメントで定義する。

```css
/* @theme name */
```

注意: Sassの圧縮出力を使う場合は、コメントが削除されないように
`/*! comment */` 構文を使うこと。

### 基本的なテーマの書き方

```css
/* @theme marpit-theme */

section {
  width: 1280px;
  height: 960px;
  font-size: 40px;
  padding: 40px;
}

h1 {
  font-size: 60px;
  color: #09c;
}

h2 {
  font-size: 50px;
}
```

背景色・文字色・フォントなども、通常のCSSプロパティを `section` などの要素に
そのまま宣言すればよい(introductionの例):

```css
/* @theme example */

section {
  background-color: #369;
  color: #fff;
  font-size: 30px;
  padding: 40px;
}

h1,
h2 {
  text-align: center;
  margin: 0;
}

h1 {
  color: #8cf;
}
```

### `:root` 疑似クラスセレクタ

Marpitの文脈では、`:root` 疑似クラスは `<html>` ではなく
**各スライドページの `<section>` 要素**を指す。

```css
/* @theme marpit-theme */

:root {
  width: 1280px;
  height: 960px;
  font-size: 40px;
  padding: 1rem;
}

h1 {
  font-size: 1.5rem;
  color: #09c;
}

h2 {
  font-size: 1.25rem;
}
```

- Marpitテーマ内の **`rem` 単位は、親の `<section>` 要素からの相対値に自動変換**される。
  そのためスライドを埋め込むページの `<html>` の `font-size` の影響を心配する必要はない。
- `:root` は `section` と同じように使えるが、**`:root` の方がCSS詳細度が高い**。
  両方が混在する場合、`:root` セレクタ内の宣言が `section` より優先される。

---

## 2. スライドサイズ

ルートの `section` セレクタ(または `:root`)の `width` / `height` 宣言が、
テーマごとの定義済みスライドサイズになる。このサイズは section 要素のサイズとしてだけでなく、
**印刷時のPDFサイズ**としても使われる。

デフォルトは **1280 x 720 ピクセル**。クラシックな4:3にする場合:

```css
/* Change to the classic 4:3 slide */
section {
  width: 960px;
  height: 720px;
}
```

制約:

- **絶対単位による静的な長さ**で定義しなければならない。
  サポート単位は `cm`, `in`, `mm`, `pc`, `pt`, `px`, `Q`。
- スライドサイズは**テーマごとに1つ**だけ決定される。
  インラインスタイル、カスタムクラス、CSSカスタムプロパティでは変更できない。
- ただし、split backgrounds(分割背景)使用時はコンテンツの幅が縮むことがある。

---

## 3. ディレクティブで注入される要素のスタイリング

### ページ番号: `section::after`

`paginate` ローカルディレクティブ(`paginate: true`)がスライドのページ番号表示を制御する。
テーマ作者は `section::after`(または `:root::after`)疑似要素でスタイリングできる。

```css
/* Styling page number */
section::after {
  font-weight: bold;
  text-shadow: 1px 1px 0 #fff;
}
```

(デフォルトスタイルはscaffoldテーマの `section::after` を参照:
https://github.com/marp-team/marpit/blob/main/src/theme/scaffold.js)

#### content のカスタマイズ

デフォルトの content は `attr(data-marpit-pagination)`(現在のページ番号)。
テーマCSSで文字列や属性を追加できる。

```css
/* Add "Page" prefix and total page number */
section::after {
  content: 'Page ' attr(data-marpit-pagination) ' / ' attr(data-marpit-pagination-total);
}
```

- `attr(data-marpit-pagination-total)` はレンダリングされたスライドの総ページ数。
  上の例は `Page 1 / 3` のように表示される。
- **重要**: `content` 宣言には必ず `attr(data-marpit-pagination)` を含めること。
  ユーザーは `paginate: true` でページ番号が表示されることを期待しているため、
  この属性への参照が含まれていない場合、**Marpitは `content` 宣言全体を無視する**。

### ヘッダー・フッター: `header` / `footer`

`header` / `footer` 要素は、`header` / `footer` ローカルディレクティブによって
レンダリングされる可能性がある。**Marpitはこれらの要素にデフォルトスタイルを持たない。**

スライドの余白部分に配置したい場合は `position: absolute` を使うのが良い解決策。

```css
section {
  padding: 50px;
}

header,
footer {
  position: absolute;
  left: 50px;
  right: 50px;
  height: 20px;
}

header {
  top: 30px;
}

footer {
  bottom: 30px;
}
```

---

## 4. テーマの継承(カスタマイズテーマ)

別のテーマをベースにしたカスタマイズテーマを作成できる。

### `@import` ルール

CSSの `@import` ルールで別のテーマをインポートできる。

```css
/* @theme base */

section {
  background-color: #fff;
  color: #333;
}
```

```css
/* @theme customized */

@import 'base';

section {
  background-color: #f80;
  color: #fff;
}
```

インポートされる側のテーマは、事前に `Marpit.themeSet.add(css)` で
テーマセットに追加されていなければならない。

### `@import-theme` ルール

Sassなどの CSSプリプロセッサ使用時は、`@import` がコンパイル時にパス解決されて
定義が失われることがある。その場合は代わりに `@import-theme` ルールを使う。

```scss
$bg-color: #f80;
$text-color: #fff;

@import-theme 'base';

section {
  background: $bg-color;
  color: $text-color;
}
```

- テーマ用の `@import` と `@import-theme` は、CSSのルート内のどこにでも書ける。
- インポートされた内容は、ルールごとの順序でCSSの先頭に挿入される
  (**`@import` が `@import-theme` より先に処理される**)。

---

## 5. Markdown内でのスタイル調整

テーマを丸ごと作り直さずに、現在のテーマを部分的に調整したいときに使う。

### `<style>` 要素

MarpitはMarkdown内に書かれた `<style>` HTML要素を特別扱いする。
指定したインラインスタイルは**テーマと同じコンテキストで解析され、
変換後のCSSにテーマと一緒にバンドルされる**。

```markdown
---
theme: base
---

<style>
section {
  background: yellow;
}
</style>

# Tweak style through Markdown

You would see a yellow slide.
```

`<style>` 要素はレンダリング後のHTMLには現れず、出力されるCSSにマージされる。

### `style` グローバルディレクティブ

`<style>` 要素と同じ目的で `style` グローバルディレクティブも使える。
`<style>` 要素は他のMarkdownエディタで開いたときにドキュメントの表示を
壊す可能性があるため、その回避策として有用。

```markdown
---
theme: base-theme
style: |
  section {
    background-color: #ccc;
  }
---
```

(ディレクティブはYAMLとして解析される。front-matter はMarkdownの先頭に置き、
ダッシュの罫線で挟む。HTMLコメント `<!-- -->` 形式でも書ける。
グローバルディレクティブはスライドデッキ全体の設定値であり、
同じものを複数回書いた場合は最後の値だけが認識される。)

### `style scoped`(スコープ付きスタイル)

`<style scoped>` によるスコープ付きインラインスタイルもサポートされる。
`style` 要素に `scoped` 属性があると、そのスタイルは**現在のスライドページにのみ**適用される。

```markdown
<!-- Global style -->
<style>
h1 {
  color: red;
}
</style>

# Red text

---

<!-- Scoped style -->
<style scoped>
h1 {
  color: blue;
}
</style>

# Blue text (only in the current slide page)

---

# Red text
```

スライドページごとにスタイルを微調整したいときに便利。

---

## 6. インラインSVGモード(実験的機能)

WebKitブラウザでのレンダリング問題のため**実験的機能**とされている。

### 有効化と出力構造

Marpitコンストラクタオプションで `inlineSVG: true` を設定すると、
各 `<section>` 要素がインラインSVG(`<svg>` 内の `<foreignObject>`)でラップされる。

```javascript
const marpit = new Marpit({
  inlineSVG: true,
})
```

出力されるHTML:

```html
<svg data-marpit-svg viewBox="0 0 1280 960">
  <foreignObject width="1280" height="960">
    <section><h1>Page 1</h1></section>
  </foreignObject>
</svg>
<svg data-marpit-svg viewBox="0 0 1280 960">
  <foreignObject width="1280" height="960">
    <section><h1>Page 2</h1></section>
  </foreignObject>
</svg>
```

`inlineSVG` オプションにはオプションオブジェクトも渡せる
(詳細は Marpit API ドキュメント の `InlineSVGOptions` を参照)。

### この方式を採る理由(効果)

1. **ピクセルパーフェクトなスケーリング**:
   スライドページのスケーリングロジックをSVGに委譲できる。表示サイズを定義するだけでよい。

   ```css
   /* Fit slide page to viewport */
   svg[data-marpit-svg] {
     display: block;
     width: 100vw;
     height: 100vh;
   }
   ```

   注意: WebKitは `<foreignObject>` 内のHTML要素をスケールできない(Bug 23113)。
   `@marp-team/marpit-svg-polyfill` で緩和できる:

   ```html
   <script src="https://cdn.jsdelivr.net/npm/@marp-team/marpit-svg-polyfill/lib/polyfill.browser.js"></script>
   ```

2. **JavaScript不要**:
   Marpitのscaffoldスタイルは `section` 要素に `scroll-snap-align` を定義している。
   スクロールコンテナに `scroll-snap-type` を定義すればビューポートに整列・フィットする
   (CSS Scroll Snap)。最小構成のWebプレゼンテーションにJavaScriptが不要になる。

3. **分離されたレイヤー**:
   advanced backgrounds(高度な背景)はコンテンツから分離された `<foreignObject>` 内で動作する。
   ページごとの元のMarkdown DOM構造が保たれるため、背景要素の挿入によって
   `:first-child` 疑似クラスや隣接結合子(`+`)のようなCSSセレクタが壊れることを防げる。

### `::backdrop` CSSセレクタ

インラインSVGモード有効時、テーマCSSやインラインスタイルの `::backdrop` セレクタは
SVGコンテナにリダイレクトされる。次のルールは `<svg data-marpit-svg>` 要素にマッチする:

```css
::backdrop {
  background-color: #448;
}
```

Marpit統合アプリの中には、SVGコンテナの背景をスライドのbackdropとして扱うものがある。
SVGコンテナに `background` スタイルを設定することで、スライドの
レターボックス/ピラーボックスの色や画像を変更できる。

- 注意: `::backdrop` 疑似要素は適用可能なスタイルを制限しないため、
  スライドやアプリへの予期しない影響を避けるべく、
  **backdropの色変更のみに使うことが強く推奨**されている。
- このリダイレクトを無効化するには `backdropSelector` オプションを `false` にする:

  ```javascript
  const marpit = new Marpit({
    inlineSVG: { backdropSelector: false },
  })
  ```

---

## 7. Marpit APIでのテーマの登録と利用

テーマCSSは `themeSet.add()` でテーマセットに追加する。`@theme` メタコメントは必須。

```javascript
import { Marpit } from '@marp-team/marpit'

const marpit = new Marpit()

// テーマの追加
const theme = marpit.themeSet.add(`
/* @theme my-first-theme */
section {
  background-color: #123;
  color: #fff;
}
`)

// デフォルトテーマの設定
marpit.themeSet.default = marpit.themeSet.add('...')
```

`render()` はMarkdownをHTML/CSSに変換する:

```javascript
const { html, css, comments } = marpit.render('# Hello, Marpit!')
```

- `html`: スライドのHTMLマークアップ。デフォルト出力は
  `<div class="marpit"><section id="1">...</section></div>` の形
  (コンテナ要素は `Element` クラスでカスタマイズ可能)
- `css`: 適用されるスタイルシート(テーマ + Markdown内 `<style>` がマージされたもの)
- `comments`: HTMLコメントから収集された注釈(プレゼンターノート)。
  ディレクティブとして解析されたコメントは含まれない

コンストラクタには markdown-it のオプションなども渡せる:

```javascript
const marpit = new Marpit({
  markdown: {
    html: true,
    breaks: true,
  },
})
```

補足: Marpit自体はフレームワークであり、**テーマは一切同梱されない**。
公式テーマや実用的な拡張機能(自動スケーリング等)が必要な場合は
`@marp-team/marp-core` を使う。
