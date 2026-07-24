# Marp テーマ制作の実務ワークフローとスタイル見本帳

コミュニティテーマの実例と公式ドキュメント(marp-vscode / marp-cli / Marpit / marp-core)の調査に基づくリファレンス。
コード抜粋は出典に忠実に引用し、推測で補完した箇所はその旨を明記する。確認できなかった項目は「未確認」と記載。

調査日: 2026-07-24

## 目次

1. [テーマの登録と利用](#1-テーマの登録と利用)
   - 1.1 テーマCSSの必須要件: `/* @theme 名前 */`
   - 1.2 VS Code: `markdown.marp.themes` 設定
   - 1.3 marp-cli: `--theme` と `--theme-set` の違い
   - 1.4 front-matter の `theme:` との対応関係
2. [プレビューとイテレーション](#2-プレビューとイテレーション)
   - 2.1 watch / server / preview モード
   - 2.2 PNG 書き出しによる目視確認
3. [コミュニティテーマから抽出したスタイルパターン集](#3-コミュニティテーマから抽出したスタイルパターン集)
   - 3.1 学会 beam 系(LaTeX Beamer 風)
   - 3.2 グラデーション系
   - 3.3 枠線装飾系
   - 3.4 ダークテーマ系(Dracula)
   - 3.5 日本語・学術発表系(academic)
   - 3.6 組み込みテーマ gaia の実装から学ぶ
4. [フォント読み込み(@import / Google Fonts)](#4-フォント読み込みimport--google-fonts)
5. [クラスバリアント設計パターン](#5-クラスバリアント設計パターン)
6. [出典一覧](#6-出典一覧)

---

## 1. テーマの登録と利用

### 1.1 テーマCSSの必須要件: `/* @theme 名前 */`

Marpit の仕様上、テーマ CSS には **`@theme` メタコメントが必須**。この名前が front-matter の `theme:` ディレクティブで指定する識別子になる。

```css
/* @theme your-theme */

@import 'default';

section {
  background: #fc9;
}
```

- 名前は `@theme` の直後に記述する(出典: marp-vscode README)。
- minify 時にコメントが消えないよう、圧縮を通す場合は `/*! @theme name */` 構文が推奨される(出典: Marpit theme-css ドキュメント)。

### 1.2 VS Code: `markdown.marp.themes` 設定

ワークスペース設定(`.vscode/settings.json`)に CSS のパスまたは URL を配列で列挙する。

```json
{
  "markdown.marp.themes": [
    "https://example.com/foo/bar/custom-theme.css",
    "./themes/your-theme.css"
  ]
}
```

- **リモート URL**: HTTPS の完全な URL を指定。
- **ローカルパス**: ワークスペースからの相対パス(`./` 始まり)。

登録後、Markdown 側の front-matter で有効化する。

```markdown
---
marp: true
theme: your-theme
---

# タイトル
```

`theme:` の値は CSS 内の `/* @theme your-theme */` で定義した名前と一致させる。

補足(未確認): 登録したローカル CSS の編集がプレビューへ即時反映される(ファイル監視される)かどうかは、今回の調査では README から確認できなかった。実務上はプレビューが更新されない場合に Markdown 側を再保存する、で回避できる(推測)。

### 1.3 marp-cli: `--theme` と `--theme-set` の違い

**`--theme`** — 変換対象すべてに適用する単一テーマの指定。組み込みテーマ名でも CSS ファイルパスでもよい。

```bash
# 組み込みテーマ名で指定
marp --theme gaia

# カスタムテーマ CSS を直接指定
marp --theme custom-theme.css
```

**`--theme-set`** — 複数のテーマ CSS を「テーマセット」として登録し、各 Markdown が front-matter の `theme:` ディレクティブで選ぶ方式。ファイル列挙もディレクトリ指定も可能。

```bash
marp --theme-set theme-a.css theme-b.css theme-c.css -- deck-a.md deck-b.md
marp --theme-set ./themes -- deck.md
```

使い分けの目安(調査結果からの整理):

| オプション | 役割 | テーマの選ばれ方 |
|---|---|---|
| `--theme` | 強制的に1つのテーマを適用 | CLI 引数が決める |
| `--theme-set` | 選択肢を登録するだけ | 各 Markdown の `theme:` が決める |

### 1.4 front-matter の `theme:` との対応関係

3つの場所で同じ名前を一致させるのが登録の要点。

```
テーマCSS:      /* @theme my-theme */
Markdown:       theme: my-theme        (front-matter)
登録:           markdown.marp.themes / --theme-set にそのCSSを含める
```

`--theme-set` で登録した CSS の中から、`/* @theme */` メタコメントの名前で識別・選択される(出典: marp-cli README)。

---

## 2. プレビューとイテレーション

### 2.1 watch / server / preview モード(marp-cli)

```bash
# Watch モード: ファイル変更を監視、ブラウザは自動リフレッシュ
marp -w slide-deck.md

# Server モード: ディレクトリを HTTP 配信、オンデマンド変換
marp -s ./slides
# → http://localhost:8080/deck.md?pdf のようにクエリ文字列で出力形式を指定可能

# Preview ウィンドウ: 専用ウィンドウを開く(watch モード自動有効)
marp -p slide-deck.md
```

テーマ開発時は `--theme-set ./themes -w` の組み合わせでテーマ CSS を編集しながら確認するのが実務的(推測による補完。`-w` がテーマ CSS 側の変更も監視するかは未確認)。

### 2.2 PNG 書き出しによる目視確認

レンダリング結果を確定した画像で確認したいとき(CI での差分確認、エージェントによる目視イテレーションなど)は画像出力を使う。

```bash
# 全ページを連番 PNG に変換(slide-deck.001.png, slide-deck.002.png, ...)
marp --images png slide-deck.md
marp --images jpeg slide-deck.md

# タイトルスライド(1枚目)のみ画像化
marp --image png slide-deck.md
marp slide-deck.md -o output.png
```

- 出力ファイル名は `slide-deck.001.png` のようにページ番号付きになる(出典: marp-cli README)。
- ローカル画像などローカルファイルを参照するスライドを PDF/画像変換する場合は `--allow-local-files` が必要になることがある。信頼できる Markdown に限定して使うこと(セキュリティ上の注意。出典: marp-cli README / コミュニティ回答)。

実務ワークフロー例(推測による整理):

1. `themes/my-theme.css` を作成し `/* @theme my-theme */` を書く
2. `marp --theme-set ./themes -w deck.md` でブラウザ確認しながら編集
3. 区切りごとに `marp --theme-set ./themes --images png deck.md` で全ページを静止画確認
4. 最終出力: `marp --theme-set ./themes --pdf deck.md`

---

## 3. コミュニティテーマから抽出したスタイルパターン集

### 3.1 学会 beam 系(LaTeX Beamer 風) — rnd195/my-marp-themes: beam

LaTeX の beamer クラスに着想を得たテーマ。CMU (Computer Modern Unicode) フォントのローカルインストールが推奨されている。

CSS 取得先: `https://rnd195.github.io/my-marp-themes/beam.css`

```css
/* @theme beam */
@import "default";

:root {
  font-family: "CMU Sans Serif", "Segoe UI", Helvetica, sans-serif;
  --main: #1f38c5;
  --secondary: #141414;
}
```

**Beamer 風タイトルバー**: 最初の `h1` を `position: absolute` でスライド上端に貼り付け、帯状に塗る。

```css
h1:nth-of-type(1) {
  font-family: "CMU Bright", "Segoe UI Semibold";
  color: #ffffff;
  background-color: var(--main);
  border-top: 0.3em solid var(--main);
  position: absolute;
  top: 0;
  right: 0;
  width: 100%;
  height: 1.5em;
}
```

ポイント:
- `h1:nth-of-type(1)` で「各スライドの最初の見出しだけ」を帯にする
- 表紙用の `title` クラス(h1 を中央配置 + `border-radius: 25px` + 主色背景)、参考文献用の `tinytext` クラス(`font-size: 0.65em`)を併設

### 3.2 グラデーション系 — rnd195/my-marp-themes: gradient

```css
/* @theme gradient */
@import "default";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');

:root {
  font-family: Inter, Helvetica, Arial;
}
```

- 既定の背景: `linear-gradient(40deg, #e2bee2ad 0%, #a7e1e7ad 100%)`(紫→シアン、アルファ付き16進で淡くする)
- 色替えバリアントをクラスで提供:

```css
section.blue {
  background-color: #ffffff;
  background-image: linear-gradient(to bottom right, #cadaf7 0%, #87a7e4 100%);
}
```

- コードブロックも半透明で背景に馴染ませる:

```css
code {
  background-color: rgba(146, 146, 146, 0.2);
}
pre {
  background-color: #ece1ecad;
}
```

### 3.3 枠線装飾系 — rnd195/my-marp-themes: border

`border` + `outline` + 負の `outline-offset` を重ねて二重枠を作るテクニック。

```css
/* @theme border */
@import "default";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');

:root {
  --border-color: #303030;
  --text-color: #0a0a0a;
  --bg-color-alt: #dadada;
  --mark-bg: #ffef92;
}

section {
  border: 1.3em solid var(--border-color);
  outline: 1em solid #ffffff;
  outline-offset: -0.5em;
}
```

参考文献スライド用の縮小クラスは直下セレクタで対象を限定している:

```css
section.tinytext > p, section.tinytext > ul, section.tinytext > blockquote {
  font-size: 0.65em;
}
```

### 3.4 ダークテーマ系 — dracula/marp

CSS パス: `https://raw.githubusercontent.com/dracula/marp/master/dracula/dracula.css`(`/* @theme dracula */` を宣言。メタコメントの正確な書式はコメント内である点のみ確認、`/*!` か `/*` かは未確認)

パレットを `:root` の CSS 変数に集約し、全スタイルを変数参照で書くパターン。

```css
@import url("https://fonts.googleapis.com/css?family=Lato:400,900|IBM+Plex+Sans:400,700");

:root {
  --dracula-background: #282a36;
  --dracula-foreground: #f8f8f2;
  --dracula-cyan: #8be9fd;
  --dracula-green: #50fa7b;
  --dracula-pink: #ff79c6;
  /* ほか計9色 */
}

section {
  font-size: 35px;
  font-family: "IBM Plex Sans";
  padding: 70px;
  color: var(--dracula-foreground);
  background-color: var(--dracula-background);
}
```

- 見出し h1–h6 は `color: var(--dracula-pink)`、サイズは 1.8em〜0.9em の段階
- インラインコードは `color: var(--dracula-green)` + 背景 `var(--dracula-current-line)`

ポイント: 配色システムを持つテーマ(Dracula/Nord など)は「変数定義ブロック」と「構造スタイル」を分離すると移植・派生が容易になる。

### 3.5 日本語・学術発表系 — kaisugi/marp-theme-academic

CSS パス: `https://raw.githubusercontent.com/kaisugi/marp-theme-academic/main/themes/academic.css`

`default` ではなく **`gaia` を継承**し、日本語フォント(Noto Sans JP)を重ねる構成。

```css
/* @theme academic */
@import 'gaia';
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Source+Code+Pro&display=swap');
```

- section: 背景画像なし、`padding` 上 90px・左右 40px、`font-family: 'Noto Sans JP'`
- `section.lead` の見出しを `color: #800000` に統一しつつ `text-align: left`(gaia の lead は中央揃えなので、継承テーマ側で上書きしている例)
- ヘッダーを `background-color: #800000` の帯にして所属・発表情報を載せる
- `strong` に `-webkit-text-stroke: 1px #800000` で輪郭強調
- `blockquote` を `bottom: 20px` で脚注風に下部固定、`border-top: dashed` で罫線

ポイント: 日本語スライドでは「Google Fonts の日本語 Web フォント + 既存テーマ継承 + アクセント1色」の組み合わせが少ない記述量で成立する。

### 3.6 組み込みテーマ gaia の実装から学ぶ(marp-core/themes/gaia.scss)

marp-core の組み込みテーマは SCSS で書かれているが、構造は CSS テーマ設計の手本になる。

**メタコメント**(minify 耐性のある `/*!` 形式、サイズ宣言付き):

```scss
/*!
 * @theme gaia
 * @author Yuki Hattori
 * @auto-scaling true
 * @size 16:9 1280px 720px
 */
```

**`light-dark()` によるカラースキーム設計**(SCSS 変数を CSS 変数に注入):

```scss
--color-background: light-dark(#{$color-light}, #{$color-dark});
--color-foreground: light-dark(#{$color-dark}, #{$color-light});
--color-highlight: light-dark(#{$color-primary}, #{$color-secondary});
```

**`:where()` でゼロ詳細度のクラスバリアント**(利用者が上書きしやすい):

```scss
&:where(.invert) {
  color-scheme: dark;
}
&:where(.gaia) {
  --color-background: #{$color-primary};
  --color-foreground: #{$color-light};
}
&:where(.lead) {
  place-content: safe center center;
  h1, h2, h3, h4, h5, h6 { text-align: center; }
}
```

**ヘッダ・フッタは絶対配置**:

```scss
header, footer {
  position: absolute;
  height: 70px;
  padding: 10px 25px;
}
```

注: 上記は SCSS ソース(`&` は `section` に解決される)。コンパイル後は `section:where(.invert)` などになる(この解決結果は推測による補足)。また `light-dark()` を使うのは近年の marp-core 実装であり、古いバージョンの gaia とは実装が異なる可能性がある。

---

## 4. フォント読み込み(@import / Google Fonts)

### 基本形

コミュニティテーマで広く使われている形(gradient / border / academic / dracula で確認):

```css
/* @theme mytheme */
@import "default";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');

:root {
  font-family: Inter, Helvetica, Arial;
}
```

### 動作可否と注意点

- **Google Fonts の `@import` は Marp チームが「既知の用法」として許容している**。GitHub Discussion #536 での説明によれば、テーマの import や Google Fonts のような「ほぼ `@font-face` 定義のみの CSS」の import は問題になりにくいが、それ以外の用途の `@import` は問題が起きうる、という位置づけ。
- **`@import` の記述位置**: CSS の仕様どおり、他のスタイル宣言より前(ファイル先頭のメタコメント直後)に書く。Marpit は `@import` / `@import-theme` をバンドル先頭に挿入する(`@import` が先)。
- **`@import` と `@import-theme` の違い**(Marpit 仕様): `@import 'default'` は既存テーマを取り込む標準ルールだが、Sass 等のプリプロセッサに解決されて意図が失われることがある。プリプロセッサを通すビルドでは `@import-theme` を使うと安全。素の CSS でテーマを書くなら `@import` でよい。
- **PDF 出力**: 変換は Chromium 経由のため、リモートフォントの取得にはネットワーク接続が必要(推測による補足)。過去には「VS Code プレビューでは効くが PDF に適用されない」事例が報告されている(marp-vscode Issue #305、特定バージョンでの不具合報告)。フォント読み込みが疑わしいときは `--images png` で実レンダリングを確認するのが確実。
- **ローカルフォント**: beam テーマのように CMU など特殊フォントは「利用者側の OS へのインストールを推奨」する方式もある。フォントファイル同梱 + `@font-face` で確実に埋め込む公式手順は今回の調査では**未確認**(Discussion #484 にも公式見解の記載なし)。
- **オフライン環境**: Google Fonts に到達できない場合はフォールバックフォントで描画される(CSS の一般挙動としての推測)。`font-family` には必ずフォールバックを列挙しておく。

---

## 5. クラスバリアント設計パターン

### 5.1 Markdown 側の切替方法

スライド単位でクラスを付けるには spot ディレクティブを使う:

```markdown
---
marp: true
theme: mytheme
class: invert        # ← 全スライドの既定クラス
---

<!-- _class: lead -->
# このスライドだけ lead(_付きは当該スライドのみ)

---

<!-- _class: lead invert -->
# 複数クラスの併用も可能
```

(`class:` / `<!-- _class: -->` の基本仕様は Marpit ディレクティブの一般知識。今回の各テーマ README でも「front-matter で有効化」の形で確認)

### 5.2 テーマ側の定義パターン

**パターンA: 素朴な `section.クラス名`**(コミュニティテーマに多い)

```css
section.blue {
  background-color: #ffffff;
  background-image: linear-gradient(to bottom right, #cadaf7 0%, #87a7e4 100%);
}

section.tinytext > p, section.tinytext > ul, section.tinytext > blockquote {
  font-size: 0.65em;
}
```

シンプルだが詳細度が上がるため、利用者側 CSS での上書きに `!important` や同等以上のセレクタが必要になりがち。

**パターンB: `:where()` でゼロ詳細度にする**(marp-core 組み込みテーマの現行方式)

```css
section:where(.lead) {
  place-content: safe center center;
}
section:where(.invert) {
  color-scheme: dark;
}
```

`:where()` は詳細度 0 なので、利用者が Markdown 内 `<style>` や `<style scoped>` で `section { ... }` と書くだけで上書きできる。「配布するテーマ」ではこちらが親切。

**パターンC: CSS 変数を「カスタマイズポイント」として公開する**

```css
/* テーマ側: 変数だけ差し替えれば色が全部変わる設計 */
:root {
  --color-background: #fff;
  --color-foreground: #333;
  --color-highlight: #0288d1;
}
section.brand {
  --color-highlight: #e91e63;   /* バリアントは変数の再定義だけで済む */
}
```

gaia が `section:where(.gaia)` 内で `--color-background` 等を再定義しているのがまさにこの形(3.6 節参照)。Dracula のようにパレット全体を変数化しておくと、バリアント追加が「変数の再代入」だけで完結する。

**パターンD: レイアウトバリアント**(lead = 表紙、title = 扉、tinytext = 参考文献)

コミュニティテーマで頻出する「意味ベース」のクラス名:

| クラス | 用途 | 実装例 |
|---|---|---|
| `lead` | 表紙・章扉(中央寄せ) | gaia: `place-content: safe center center` |
| `invert` | 明暗反転 | gaia: `color-scheme: dark` |
| `title` | 表紙(beam) | h1 中央配置 + 角丸 + 主色背景 |
| `tinytext` | 参考文献・注記 | `font-size: 0.65em` |
| `blue` 等の色名 | 配色替え | 背景グラデーションと見出し色の差し替え |

設計指針(調査結果からの整理): 「色を変えるバリアント」は CSS 変数の再定義で、「レイアウトを変えるバリアント」は `:where()` 付きセレクタで定義すると、両者が干渉せず利用者の上書きも容易になる。

---

## 6. 出典一覧

- rnd195/my-marp-themes (beam / border / gradient / graph_paper): https://github.com/rnd195/my-marp-themes — CSS 配布: `https://rnd195.github.io/my-marp-themes/<テーマ名>.css`
- marp-core 組み込みテーマ gaia: https://raw.githubusercontent.com/marp-team/marp-core/main/themes/gaia.scss
- Marp for VS Code README (`markdown.marp.themes`): https://github.com/marp-team/marp-vscode
- Marp CLI README (`--theme` / `--theme-set` / `--images` / `-w` / `-s` / `-p`): https://github.com/marp-team/marp-cli
- Marpit テーマ CSS 仕様 (`@theme` / `@import-theme` / `section::after` / header・footer): https://github.com/marp-team/marpit/blob/main/docs/theme-css.md
- Dracula for Marp: https://github.com/dracula/marp (`dracula/dracula.css`, master ブランチ)
- kaisugi/marp-theme-academic: https://github.com/kaisugi/marp-theme-academic (`themes/academic.css`, main ブランチ)
- テーマ一覧(さらに探すとき): https://github.com/marp-team/awesome-marp
- `@import` の可否に関する Marp チームの見解: https://github.com/marp-team/marp/discussions/536
- カスタムフォントに関する Discussion(公式見解なし): https://github.com/orgs/marp-team/discussions/484
- PDF 出力でカスタムテーマが効かない事例報告: https://github.com/marp-team/marp-vscode/issues/305
