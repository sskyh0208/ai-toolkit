---
name: marp-custom-theme
description: >-
  Marp / Marpit のカスタムテーマ(テーマCSS)をユーザーの好みに合わせて設計・作成・改良するためのスキル。
  ユーザーが「スライドの見た目を変えたい」「自分好みのテーマにしたい」「配色・フォントを変えたい」「テーマCSSを作りたい」
  「デフォルトテーマに飽きた」「◯◯っぽいデザインのスライドにしたい」など、Marpスライドのデザイン・テーマ・見た目に
  関する要望を出したときは、CSSに言及がなくても必ずこのスキルを使うこと。既存カスタムテーマの微調整にも使う。
---

# Marp カスタムテーマ作成

このスキルはスライドの**見た目(テーマCSS)**専用。スライド本文の書き方・変換は marp-slides スキルの担当。

Marpのテーマは「`/* @theme 名前 */` メタコメントを持つ1枚のCSS」で、`section` がスライド1枚に対応する。この構造さえ守れば普通のCSSの知識がそのまま使える。

## ワークフロー

### 1. 好みを把握する

「自分好み」はユーザーごとに違う。手掛かり(ブランドカラー、参考サイト・スライド、雰囲気を表す言葉)が与えられていればそこから設計し、足りなければ**作り始める前に聞く**:

- 用途(LT / 社内報告 / 登壇 / 学会)と雰囲気(ポップ / ミニマル / かっちり)
- ライト基調かダーク基調か
- ベース色・アクセント色(なければ好きな色や参考物)
- フォントの好み(ゴシック / 明朝 / 丸ゴ / 等幅多用)

無難な平均的テーマで済ませない。聞いた答えがそのままデザイントークンになる。

### 2. デザイントークンを決める

色・フォント・余白を `:root` の CSS 変数に集約する。色は **OKLCH** で設計すると複数色の明度・彩度を揃えやすく、パレットが破綻しにくい(MarpのレンダリングはChromiumなので `oklch()` はそのまま使える):

```css
:root {
  --color-bg: oklch(98% 0.005 250);
  --color-fg: oklch(25% 0.02 250);
  --color-accent: oklch(60% 0.18 250);   /* 明度を固定して色相だけ振ると調和する */
  --font-body: 'Noto Sans JP', sans-serif;
}
```

### 3. ベースを選ぶ

- **色替え・部分調整** → `@import 'default';`(または gaia / uncover)で組み込みテーマを継承し、差分だけ書く。ページ番号の `content` などが引き継がれるので楽
- **雰囲気から作る大改造** → ゼロから書く。継承すると打ち消しだらけになる

### 4. 全要素をスタイルする(チェックリスト)

スライドで実際に使われる要素を漏らすと「その要素を使った瞬間に崩れるテーマ」になる:

- [ ] `section` — サイズ(既定1280x720)・padding・背景・文字色・行間
- [ ] `h1`〜`h3`(アクセントの入れ方をここで決める)
- [ ] リスト(`li::marker` の色)・段落・`a`・`strong`
- [ ] `code`(インライン)と `pre`(ブロック)— 背景コントラスト必須
- [ ] `table`(th の配色・ボーダー)・`blockquote`
- [ ] `section::after` のページ番号 — **`content` に `attr(data-marpit-pagination)` を含めること**(含めないと宣言ごと無視される仕様)
- [ ] `header` / `footer`(position: absolute で四隅に)
- [ ] クラスバリアント — 最低限 `section.lead`(タイトル用センタリング)と `section.invert`(反転)。`<!-- _class: -->` で切り替える

### 5. サンプルデッキで目視イテレーション

このスキルの `assets/sample-deck.md` が全要素入りのテスト台。`THEME_NAME` を自作テーマ名に置換してPNG書き出しし、**全ページを実際に見る**:

```bash
npx @marp-team/marp-cli@latest sample-deck.md --theme my-theme.css --images png -o preview/deck.png
```

見るポイント: 文字あふれ / 本文と背景のコントラスト(目安4.5:1) / ページ番号・ヘッダー・フッターの位置と重なり / invert時も `code` や `strong` が読めるか。崩れていたら直して再レンダリング。**見た目の成果物を目視せずに納品しない。**

### 6. 「今後ずっと自動で当たる」形で納品する

CSSファイルを渡すだけでは、次にスライドを作るとき(人間でもAIでも)テーマの当て方をまた段取りすることになる。スライドを書くディレクトリに次の3点をセットで配置し、**front-matter に `theme: 名前` と書くだけで全経路(CLI変換・VS Codeプレビュー・AI生成)に適用される状態**にして納品する:

1. `themes/<名前>.css` — テーマ本体(テーマはこのディレクトリに集約)
2. `.marprc.yml` — `themeSet: themes` の1行。以後 `npx @marp-team/marp-cli slide.md --pdf` だけでテーマ名が解決される(`--theme` フラグ不要)
3. `.vscode/settings.json` — `"markdown.marp.themes": ["./themes/<名前>.css"]`(既存設定があれば配列に追記)。VS Codeプレビューにも適用される

そのうえで使い方を短く伝える: front-matter は `theme: <@theme名>`(CSS内の `/* @theme */` と一致)、単発で使うなら `--theme my-theme.css` でも可。

## CSSで解決できないとき

コードハイライトの配色変更や独自Markdown記法は、テーマCSSではなくカスタムエンジン(markdown-itプラグイン + `--engine engine.mjs`)の領域。references/real-world-example.md に最小構成がある。

## リファレンス(必要になったら読む)

- **references/theme-css.md** — Marpitテーマの正確な仕様(@theme、サイズ、ページ番号、@import-theme、style scoped)。テーマを書く前に必読
- **references/theme-workflow.md** — VS Code / CLI へのテーマ登録方法、コミュニティテーマから抽出したスタイルパターン集、Google Fontsの読み込みと注意
- **references/real-world-example.md** — 実在するLTテーマの解剖(OKLCH設計、2カラムなどのユーティリティクラス、カスタムエンジン)
