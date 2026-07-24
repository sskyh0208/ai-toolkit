---
name: marp-slides
description: >-
  Marp / Marpit を使って Markdown からプレゼンテーションスライドを作成・編集・変換するためのスキル。
  ユーザーが「スライド」「プレゼン」「発表資料」「LT資料」の作成を依頼したとき、.md でスライドを書きたいとき、
  Marp・Marpit・marp-cli に言及したとき、既存文書のスライド化や PDF/PPTX への変換を求めたときは、
  明示的に「Marp」と言われなくても必ずこのスキルを使うこと。テーマCSSのカスタマイズにも対応する。
---

# Marp スライド作成

Marp は Markdown からスライドデッキを生成するエコシステム。`---` で区切った Markdown を書き、Marp CLI や VS Code 拡張で HTML / PDF / PPTX に変換する。

## 基本ワークフロー

0. **ユーザーのカスタムテーマを探す。** スライドを作る前に、プロジェクト内の `themes/*.css`(`/* @theme 名前 */` を持つCSS)、`.marprc`/`.marprc.yml` の `themeSet:`、`.vscode/settings.json` の `markdown.marp.themes` を確認する。見つかったら組み込みテーマではなく**そのテーマ名を front-matter の `theme:` に使う**。ユーザーがテーマを育てている環境で default に戻すのは体験を壊す。変換時は `.marprc` があればフラグ不要、なければ `--theme-set themes/` を付ける
1. front-matter を書く(`marp: true` は Marp CLI / VS Code で必須)
2. `---`(水平線)でスライドを区切りながら本文を書く
3. ディレクティブでページ番号・ヘッダー・背景・テーマ固有クラスを制御する
4. 必要ならテーマ CSS や `<style>` で見た目を調整する
5. Marp CLI で変換して確認する

### 最小テンプレート

```markdown
---
marp: true
theme: default
paginate: true
---

<!-- _paginate: false -->

# タイトル

発表者名 / 日付

---

## 最初のセクション

- ポイント1
- ポイント2
```

## 書くときの原則

- **`---` の前には空行を置く。** CommonMark の仕様上、直前に空行がないと水平線(=スライド区切り)として解釈されないことがある。見出しごとに自動でページを分けたい場合は `headingDivider: 2` グローバルディレクティブも使える。
- **1スライド1メッセージ。** スライドは紙面が固定サイズ(既定 16:9)で、あふれた内容は見切れる。箇条書きは1枚あたり5行程度、コードブロックは15行程度までを目安に、多ければスライドを分割する。
- **ディレクティブには2つのスコープがある。** front-matter やコメントで書く `theme:` `paginate:` などはそれ以降の全スライドに効くグローバル/ローカル設定。`_paginate: false` のようにアンダースコア付き(スポットディレクティブ)にすると、そのスライド1枚だけに効く。タイトルページのページ番号消しはこの典型。
- **タイトル・区切りスライドは装飾を変える。** gaia テーマなら `<!-- _class: lead -->`、反転配色なら `class: invert` が使える(テーマにより異なる。references/marp-ecosystem.md 参照)。
- **画像は Marp 独自構文を活用する。** `![bg](url)` で背景全面、`![bg left:40%](url)` で画面分割、`![w:300](url)` でサイズ指定ができる。レイアウト目的の HTML を書く前に画像構文で済まないか検討する。
- **プレースホルダ画像はURLを固定する。** `https://picsum.photos/800/600` のようなランダム画像URLは、プレビュー・HTML・PDFのそれぞれで別の写真が読み込まれ、「出力によって色味や印象が違う」という混乱を生む。`https://picsum.photos/seed/<任意の文字列>/800/600` や `/id/<番号>/` で固定すれば全出力で同じ画像になる。本番の資料ではローカルまたは自社ホストの画像を使う(PDF/PPTX変換時は `--allow-local-files`)。
- **数式は `$...$` / `$$...$$`。** Marp Core が MathJax でレンダリングする(`math: katex` ディレクティブで KaTeX に変更可)。
- **スピーカーノートは HTML コメント。** ディレクティブでないコメント(`<!-- これはノート -->`)はプレゼンターノートとして扱われる。
- **確認は実変換で。** `marp` コマンドが使える環境なら変換してエラーが出ないか確かめる。

## リファレンス(必要になったら読む)

- **references/marpit-markdown.md** — ディレクティブの完全な一覧(グローバル/ローカル/スポット)、スライド区切りの正確なルール、画像・背景画像構文(サイズ・分割・フィルタ)、フラグメントリスト。ディレクティブ名や画像構文をうろ覚えのまま書かず、ここで確認する。
- **references/theme-css.md** — カスタムテーマ CSS(`/* @theme */`)、`section` セレクタ、ページ番号(`section::after`)・ヘッダー・フッターのスタイリング、`<style scoped>`。見た目のカスタマイズを頼まれたら読む。
- **references/marp-ecosystem.md** — 組み込みテーマ(default / gaia / uncover)の特徴とテーマ固有クラス、数式(math)、自動スケーリング、Marp CLI での HTML/PDF/PPTX/画像変換コマンドとオプション、VS Code 拡張、スピーカーノートの詳細。テーマ選定・変換・出力形式の話になったら読む。

## 変換コマンド早見

```bash
npx @marp-team/marp-cli@latest slide.md          # HTML
npx @marp-team/marp-cli@latest slide.md --pdf    # PDF
npx @marp-team/marp-cli@latest slide.md --pptx   # PowerPoint
npx @marp-team/marp-cli@latest slide.md --theme custom.css --pdf
```

ローカル画像を使った PDF/PPTX 変換では `--allow-local-files` が必要になる点に注意。
