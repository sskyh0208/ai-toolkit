# Marpit Markdown リファレンス

Marpit 公式ドキュメント(https://marpit.marp.app/)に基づく、スライド執筆用リファレンス。

## 目次

1. [スライドの区切り](#スライドの区切り)
2. [ディレクティブの書き方](#ディレクティブの書き方)
   - [HTMLコメント記法](#htmlコメント記法)
   - [Front-matter記法](#front-matter記法)
3. [ディレクティブの種別](#ディレクティブの種別)
   - [グローバルディレクティブ](#グローバルディレクティブ)
   - [ローカルディレクティブ](#ローカルディレクティブ)
   - [スポットディレクティブ(`_`プレフィックス)](#スポットディレクティブ_プレフィックス)
4. [ディレクティブ一覧表](#ディレクティブ一覧表)
5. [各ディレクティブの詳細と例](#各ディレクティブの詳細と例)
   - [theme](#theme)
   - [style](#style)
   - [headingDivider](#headingdivider)
   - [paginate](#paginate)
   - [header / footer](#header--footer)
   - [class](#class)
   - [背景・文字色(backgroundColor ほか)](#背景文字色backgroundcolor-ほか)
6. [画像構文](#画像構文)
   - [機能対応表](#機能対応表)
   - [画像のリサイズ](#画像のリサイズ)
   - [画像フィルタ](#画像フィルタ)
   - [背景画像 `![bg]`](#背景画像-bg)
   - [背景サイズキーワード](#背景サイズキーワード)
   - [高度な背景(複数背景・split背景)](#高度な背景複数背景split背景)
7. [フラグメントリスト](#フラグメントリスト)

---

## スライドの区切り

Marpit は水平罫線(例: `---`)でスライドのページを分割する。

```markdown
# Slide 1

foo

---

# Slide 2

bar
```

- CommonMark の仕様上、ダッシュの罫線 `---` の前には空行が必要な場合がある。
- 空行を入れたくない場合は、アンダースコア罫線 `___`、アスタリスク罫線 `***`、スペース入り罫線 `- - -` が使える。

---

## ディレクティブの書き方

ディレクティブは YAML としてパースされる。値に YAML の特殊文字を含む場合は、正しく認識させるためにクォートで囲むこと(Marpit コンストラクタの `looseYAML` オプションで緩いパースを有効化することも可能)。

### HTMLコメント記法

```markdown
<!--
theme: default
paginate: true
-->
```

- HTMLコメントはプレゼンターノートにも使われる。ディレクティブとしてパースされた場合、`Marpit.render()` の `comments` 結果には収集されない。

### Front-matter記法

YAML front-matter もサポートされる。**Markdown の先頭に置かなければならず**、ダッシュ罫線で囲む。

```markdown
---
theme: default
paginate: true
---
```

- スライドをページ分割するための罫線と混同しないこと。実際のスライド内容は front-matter の終了罫線の後から始まる。

---

## ディレクティブの種別

### グローバルディレクティブ

**スライドデッキ全体の設定値**(テーマなど)。同じグローバルディレクティブを複数回書いた場合、Marpit は**最後の値のみ**を認識する。

- 注意: グローバルディレクティブの `$` プレフィックスは v1.4.0 で削除された。

### ローカルディレクティブ

**ページごとの設定値**。**定義したページと、それ以降のページ**に適用される。

```markdown
<!-- backgroundColor: aqua -->

This page has aqua background.

---

The second page also has same color.
```

### スポットディレクティブ(`_`プレフィックス)

ローカルディレクティブを**現在のページのみ**に適用したい場合、ディレクティブ名の先頭に `_` を付ける。

```markdown
<!-- _backgroundColor: aqua -->

Add underscore prefix `_` to the name of local directives.

---

The second page would not apply setting of directives.
```

- カスタムローカルディレクティブでも同様に `_` プレフィックスが機能する。

---

## ディレクティブ一覧表

### グローバルディレクティブ一覧

| 名前 | 種別 | 意味 | 例 |
| :--- | :--- | :--- | :--- |
| `theme` | グローバル | スライドデッキのテーマを指定(`themeSet` に登録済みのテーマ名) | `theme: default` |
| `style` | グローバル | テーマ調整用のCSSを指定 | `style: \| ...`(下記参照) |
| `headingDivider` | グローバル | 見出しの直前で自動的にページ分割する(1〜6のレベル、または配列) | `headingDivider: 2` |
| `lang` | グローバル | 各スライドの `lang` 属性の値を設定 | `lang: ja` |

### ローカルディレクティブ一覧

すべて `_` プレフィックスでスポットディレクティブとして使用可能。

| 名前 | 種別 | 意味 | 例 |
| :--- | :--- | :--- | :--- |
| `paginate` | ローカル | `true` でスライドにページ番号を表示(`true` / `false` / `hold` / `skip`) | `paginate: true` |
| `header` | ローカル | スライドのヘッダー内容を指定 | `header: 'Header content'` |
| `footer` | ローカル | スライドのフッター内容を指定 | `footer: 'Footer content'` |
| `class` | ローカル | スライドの `<section>` 要素のHTMLクラスを指定 | `class: lead` |
| `backgroundColor` | ローカル | スライドの `background-color` スタイルを設定 | `backgroundColor: aqua` |
| `backgroundImage` | ローカル | スライドの `background-image` スタイルを設定 | `backgroundImage: url(bg.jpg)` |
| `backgroundPosition` | ローカル | スライドの `background-position` スタイルを設定(デフォルト: `center`) | `backgroundPosition: top` |
| `backgroundRepeat` | ローカル | スライドの `background-repeat` スタイルを設定(デフォルト: `no-repeat`) | `backgroundRepeat: repeat` |
| `backgroundSize` | ローカル | スライドの `background-size` スタイルを設定(デフォルト: `cover`) | `backgroundSize: contain` |
| `color` | ローカル | スライドの `color`(文字色)スタイルを設定 | `color: white` |

---

## 各ディレクティブの詳細と例

### theme

Marpit インスタンスの `themeSet` に追加されたテーマ名を指定する。

```markdown
<!-- theme: registered-theme-name -->
```

### style

通常は `<style>` 要素でテーマを調整できるが、他のMarkdownエディタで開いたときに表示が崩れる可能性がある。代わりに `style` グローバルディレクティブが使える。

```markdown
---
theme: base-theme
style: |
  section {
    background-color: #ccc;
  }
---
```

### headingDivider

見出しの直前で自動的にスライドを分割する。Pandoc の `--slide-level` や Deckset 2 の "Slide Dividers" に類似。

- 1〜6 の見出しレベル、またはその配列を指定する。
- **数値**の場合: そのレベル以上の大きさの見出しで有効(例: `headingDivider: 2` なら `#` と `##` で分割され、`###` では分割されない。下記の例を参照)。
- **配列**の場合: **指定したレベルのみ**で有効。
- Marpit コンストラクタで headingDivider のデフォルトレベルを設定することもできる。

以下の2つのMarkdownは同じ出力になる。

通常の記法:

```markdown
# 1st page

The content of 1st page

---

## 2nd page

### The content of 2nd page

Hello, world!

---

# 3rd page

😃
```

headingDivider を使った記法:

```markdown
<!-- headingDivider: 2 -->

# 1st page

The content of 1st page

## 2nd page

### The content of 2nd page

Hello, world!

# 3rd page

😃
```

### paginate

```markdown
<!-- paginate: true -->

You would be able to see a page number of slide in the lower right.
```

各スライドでは「ページ番号の表示」と「ページ番号のインクリメント」の2つが起きており、`paginate` の値で両方を制御できる。

| `paginate` の値 | ページ番号の表示 | インクリメント |
| :--- | :--- | :--- |
| `true` | 表示 | する |
| `false` | 非表示 | する |
| `hold` | 表示 | しない |
| `skip` | 非表示 | しない |

タイトルスライドをページ番号から除外する一般的な方法は、2ページ目に `paginate` を定義すること。

```markdown
# Title slide

This page will not have pagination by lack of the `paginate` directive.

---

<!-- paginate: true -->

Pagination will render from this slide onwards (starting at 2).
```

またはスポットディレクティブを使う。

```markdown
---
paginate: true
_paginate: false # or use `_paginate: skip`
---
```

ページ番号から除外しつつ表示も隠すには `skip`:

```markdown
<!-- _paginate: skip -->

# Slide to exclude

This page will not update the page number and also not show the pagination
```

ページ番号から除外するが表示は維持するには `hold`:

```markdown
---
paginate: true
---

# Slide 1

> Page 1 of 1

---

<!-- _paginate: hold -->

# Slide 2

> Page 1 of 1
```

### header / footer

複数スライドにわたり同じ内容(デッキタイトルなど)を表示するのに使う。

```markdown
---
header: 'Header content'
footer: 'Footer content'
---

# Page 1

---

## Page 2
```

レンダリング結果:

```html
<section>
  <header>Header content</header>
  <h1>Page 1</h1>
  <footer>Footer content</footer>
</section>
<section>
  <header>Header content</header>
  <h2>Page 2</h2>
  <footer>Footer content</footer>
</section>
```

- 内容は対応する要素でラップされ、各スライドの適切な位置に挿入される。スライドコンテンツの一部として見なされる。
- PowerPoint のようにスライドの余白部分に配置したい場合は、**対応したテーマを使う必要がある**。

Markdown記法での装飾やインライン画像の挿入も可能。無効なYAMLとしてパースされないよう(ダブル)クォートで囲むこと。

```markdown
---
header: '**bold** _italic_'
footer: '![image](https://example.com/image.jpg)'
---
```

- 注意: Markdownのパース順の都合により、`header` / `footer` ディレクティブ内では `![bg]()` 構文は使えない。

### class

スライドページの `<section>` 要素の class 属性を変更する。テーマに次のようなルールがある場合:

```css
section.lead h1 {
  text-align: center;
}
```

`class` スポットディレクティブを `lead` に設定すれば、中央寄せの見出しが使える。

```markdown
<!-- _class: lead -->

# THE LEADING HEADER
```

### 背景・文字色(backgroundColor ほか)

色やグラデーションを背景に使いたい場合、`backgroundColor` や `backgroundImage` ローカルディレクティブでスタイルを設定できる。

```markdown
<!-- backgroundImage: "linear-gradient(to bottom, #67b8e3, #0288d1)" -->

Gradient background

---

<!--
_backgroundColor: black
_color: white
-->

Black background + White text
```

カスタマイズをサポートする宣言:

- `backgroundColor`
- `backgroundImage`
- `backgroundPosition`(デフォルト: `center`)
- `backgroundRepeat`(デフォルト: `no-repeat`)
- `backgroundSize`(デフォルト: `cover`)
- `color`

単一ページへの背景画像・色の設定には、拡張画像構文(`![bg]`)も使える。

---

## 画像構文

Marpit は Markdown の画像構文 `![](image.jpg)` を拡張している。拡張機能は画像の**代替テキストに対応するキーワードを含める**ことで有効になる。残りの代替テキストは、インライン画像では alt テキスト、背景画像では図のキャプションとしてレンダリングされる。

### 機能対応表

| 機能 | インライン画像 | スライド背景 | 高度な背景 |
| :--- | :---: | :---: | :---: |
| キーワードによるリサイズ | `auto` のみ | ✅ | ✅ |
| パーセントによるリサイズ | ❌ | ✅ | ✅ |
| 長さ指定によるリサイズ | ✅ | ✅ | ✅ |
| 画像フィルタ | ✅ | ❌ | ✅ |
| 複数背景 | - | ❌ | ✅ |
| split背景 | - | ❌ | ✅ |

### 画像のリサイズ

`width` と `height` キーワードオプションでリサイズできる。

```markdown
![width:200px](image.jpg) <!-- Setting width to 200px -->
![height:30cm](image.jpg) <!-- Setting height to 300px -->
![width:200px height:30cm](image.jpg) <!-- Setting both lengths -->
```

短縮形 `w` と `h` もサポートされ、通常はこちらが便利。

```markdown
![w:32 h:32](image.jpg) <!-- Setting size to 32x32 px -->
```

- インライン画像では、`auto` キーワードと CSS で定義された長さ単位**のみ**使用可能。
- レンダリング結果を不変にするため、ビューポート関連の単位(`vw`, `vh`, `vmin`, `vmax` など)は使えない。

### 画像フィルタ

画像の代替テキストに `<filter-name>(:<param>(,<param>...))` を含めることで、CSSフィルタを適用できる。フィルタは**インライン画像と高度な背景**で使える(通常のスライド背景では不可)。

| Markdown | 引数付きの例 |
| :--- | :--- |
| `![blur]()` | `![blur:10px]()` |
| `![brightness]()` | `![brightness:1.5]()` |
| `![contrast]()` | `![contrast:200%]()` |
| `![drop-shadow]()` | `![drop-shadow:0,5px,10px,rgba(0,0,0,.4)]()` |
| `![grayscale]()` | `![grayscale:1]()` |
| `![hue-rotate]()` | `![hue-rotate:180deg]()` |
| `![invert]()` | `![invert:100%]()` |
| `![opacity]()` | `![opacity:.5]()` |
| `![saturate]()` | `![saturate:2.0]()` |
| `![sepia]()` | `![sepia:1.0]()` |

引数を省略した場合、上表の引数がデフォルト値として使われる。複数フィルタの同時適用も可能。

```markdown
![brightness:.8 sepia:50%](https://example.com/image.jpg)
```

### 背景画像 `![bg]`

代替テキストに `bg` キーワードを含めるだけで、スライドの背景を指定できる。

```markdown
![bg](https://example.com/background.jpg)
```

- 1つのスライドに2つ以上の背景画像を定義した場合、Marpit は**最後に定義した画像のみ**を表示する。複数画像を表示したい場合は、インラインSVGスライドを有効にして「高度な背景」を使う。

### 背景サイズキーワード

キーワードで背景画像をリサイズできる。値は基本的に CSS の `background-size` に従う。

```markdown
![bg contain](https://example.com/background.jpg)
```

| キーワード | 説明 | 例 |
| ---: | :--- | :--- |
| `cover` | スライド全体を埋めるように拡大縮小 **(デフォルト)** | `![bg cover](image.jpg)` |
| `contain` | スライドに収まるように拡大縮小 | `![bg contain](image.jpg)` |
| `fit` | `contain` のエイリアス(Deckset互換) | `![bg fit](image.jpg)` |
| `auto` | 拡大縮小せず原寸を使用 | `![bg auto](image.jpg)` |
| _`x%`_ | パーセント値で倍率を指定 | `![bg 150%](image.jpg)` |

`width`(`w`)/ `height`(`h`)オプションキーワードによる長さ指定も引き続き使える。

### 高度な背景(複数背景・split背景)

- 注意: **実験的なインラインSVGスライドが有効な場合のみ動作する。**
- 高度な背景では、複数背景・split背景・背景への画像フィルタがサポートされる。

#### 複数背景

```markdown
![bg](https://fakeimg.pl/800x600/0288d1/fff/?text=A)
![bg](https://fakeimg.pl/800x600/02669d/fff/?text=B)
![bg](https://fakeimg.pl/800x600/67b8e3/fff/?text=C)
```

これらの画像は**横一列**に並ぶ。

#### 方向キーワード(vertical)

`vertical` 方向キーワードで、並び方向を水平から垂直に変更できる。

```markdown
![bg vertical](https://fakeimg.pl/800x600/0288d1/fff/?text=A)
![bg](https://fakeimg.pl/800x600/02669d/fff/?text=B)
![bg](https://fakeimg.pl/800x600/67b8e3/fff/?text=C)
```

#### split背景

`bg` キーワードに `left` または `right` キーワードを付けると、指定した側に背景用のスペースを作る。スペースはスライドサイズの半分で、スライドコンテンツのスペースも縮む。

```markdown
![bg left](https://picsum.photos/720?image=29)

# Split backgrounds

The space of a slide content will shrink to the right side.
```

split背景と複数背景の併用も可能(指定した側に複数背景が並ぶ)。

```markdown
![bg right](https://picsum.photos/720?image=3)
![bg](https://picsum.photos/720?image=20)

# Split + Multiple BGs

The space of a slide content will shrink to the left side.
```

- 同じスライドで `left` と `right` が混在した場合、Marpit は**最後に定義したキーワード**を使う。

#### split サイズ指定

`left:33%` のようにパーセントで split サイズを指定できる。

```markdown
![bg left:33%](https://picsum.photos/720?image=27)

# Split backgrounds with specified size
```

---

## フラグメントリスト

v0.9.0 以降、Marpit は特定のマーカーを持つリストを、内容を1つずつ表示するための**フラグメントリスト**としてパースする。

### 箇条書きリスト

CommonMark は箇条書きマーカーとして `-`、`+`、`*` を許容する。Marpit は **`*` をマーカーに使った場合**にフラグメントリストとしてパースする。

```markdown
# Bullet list

- One
- Two
- Three

---

# Fragmented list

* One
* Two
* Three
```

### 番号付きリスト

CommonMark の番号付きリストマーカーは数字の後に `.` または `)` を持つ。Marpit は **`)` を使った場合**にフラグメントリストとしてパースする。

```markdown
# Ordered list

1. One
2. Two
3. Three

---

# Fragmented list

1) One
2) Two
3) Three
```

### レンダリング

フラグメントリストのレンダリング結果のHTML構造は通常のリストと同じ。リスト項目に `data-marpit-fragment` データ属性が追加されるだけで、認識された順に 1 から番号が振られる。

さらに、フラグメントリストを持つスライドの `<section>` 要素には `data-marpit-fragments` データ属性が追加され、そのスライドのフラグメントリスト項目数を示す。

```html
<section id="1">
  <h1>Bullet list</h1>
  <ul>
    <li>One</li>
    <li>Two</li>
    <li>Three</li>
  </ul>
</section>
<section id="2" data-marpit-fragments="3">
  <h1>Fragmented list</h1>
  <ul>
    <li data-marpit-fragment="1">One</li>
    <li data-marpit-fragment="2">Two</li>
    <li data-marpit-fragment="3">Three</li>
  </ul>
</section>
```

- 注意: フラグメントリストはDOM構造や見た目を変えない。レンダリングされたリストを実際にフラグメントとして扱うかどうかは、統合するアプリの挙動に依存する。
