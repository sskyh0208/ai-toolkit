---
name: zensical-authoring
description: Zensical（Material for MkDocs 後継）のドキュメントページを Markdown で執筆する。「admonition／注意書きを入れたい」「コードブロックにアノテーション・行番号を付けたい」「タブでコード例を切り替えたい」「Mermaid 図・数式・カードグリッドを入れたい」「frontmatter でタイトル・タグ・非表示設定をしたい」など、docs/ 配下の Markdown を書く・直す場面で必ずこの skill を使う。記法ごとの詳細は references/syntax-reference.md を参照。
metadata:
  trigger: Zensical ページ執筆、admonition、コードブロック、コンテンツタブ、Mermaid、frontmatter
  language: ja
---

# Zensical ページ執筆

Zensical の `docs/` 配下の Markdown を書く。記法の全カタログは [references/syntax-reference.md](references/syntax-reference.md) にある。使う記法のセクションを確認してから書く。

## 大原則

1. **インデントは4スペース**。Python Markdown 系のため、admonition・コンテンツタブ・リスト内段落など、ブロックの中身は必ずスペース4つでインデントする。2スペースでは動かない。これがこの環境で最も踏みやすい罠。
2. **ページ間リンクは `.md` への相対リンク**で書く（`[設定](../setup/basics.md)`）。生成後の HTML パスや絶対 URL を書かない。Zensical が正しい URL に変換し、`use_directory_urls` の設定変化にも追従する。
3. **各ページの先頭に h1（`# タイトル`）を1つ書く**。`nav` 定義があっても h1 がないとファイル名が見出しになる。
4. **記法には対応する拡張が必要**。`zensical new` の既定設定ならこの skill で扱う記法はほぼ全部使える（テーブル含む）。外部ファイル埋め込み（`pymdownx.snippets`）と画像キャプション（`pymdownx.blocks.caption`）は既定に含まれないことを実ビルドで確認済みなので、使うなら設定に追加する（`zensical-config` 参照）。
5. **書いたら `zensical build --strict` で検証**。リンク切れがソース位置付きで検出される（`zensical-project` 参照）。

## 頻出記法チートシート（Zensical 0.0.47 でレンダリング確認済み）

```markdown
---
title: ページタイトル
description: 検索エンジン用の説明
icon: lucide/rocket
tags:
  - Setup
---

# ページタイトル

!!! note "カスタムタイトル"

    本文は4スペースインデント。型: note/tip/info/warning/danger/example/quote など12種。

??? tip "折りたたみ（???+ で初期展開）"

    折りたたみには pymdownx.details が必要（既定で有効）。

=== "Python"

    ``` python
    print("hello")
    ```

=== "JavaScript"

    ``` js
    console.log("hello")
    ```
```

コードブロックのオプションと図:

````markdown
``` python title="example.py" linenums="1" hl_lines="2"
def main():
    return 42  # (1)!
```

1. コードアノテーション。`# (1)!` をコメント位置に置き、直後の番号付きリストが本文になる。

``` mermaid
graph LR
  A[Start] --> B{Check};
  B -->|Yes| C[Done];
```
````

インライン: `==ハイライト==`、`^^挿入^^`、`~~削除~~`、`H~2~O`、`A^T^A`、`++ctrl+alt+del++`、絵文字/アイコン `:smile:` `:fontawesome-brands-github:`。

## ファイル配置

- `docs/index.md` がトップページ。`README.md` も index.html になるが、**`index.md` との併存は未定義動作**なのでどちらかに統一する。
- ディレクトリ構造がそのままナビゲーションになる（`nav` 未定義時）。セクションの概要ページは `<section>/index.md` に置く（`navigation.indexes` 有効時にセクション直付けされる）。
- 用語集など全ページ共通の断片は `docs/` の**外**（例 `includes/`）に置く。`docs/` 内に置くと未参照ファイルとして警告されうる。

## 参照

- 記法カタログ（admonition 全型、タブ、コード、図、画像、グリッド、テーブル、脚注、数式、アイコン、ボタン、ツールチップ、frontmatter 全項目）: [references/syntax-reference.md](references/syntax-reference.md)
- 記法に必要な拡張の設定: `zensical-config` skill
- ビルド・プレビュー: `zensical-project` skill
