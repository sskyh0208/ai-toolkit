---
marp: true
theme: THEME_NAME
paginate: true
header: 'サンプルヘッダー'
footer: 'サンプルフッター / 2026'
---

<!-- _class: lead -->
<!-- _paginate: false -->

# テーマ確認用サンプルデッキ

サブタイトル — このテーマの見え方を全要素で確認する

---

## 見出しと本文・リスト

本文の段落。**強調** と *斜体*、`inline code`、[リンク](https://marp.app/) を含む。

- 箇条書きレベル1
  - ネストしたレベル2
  - `code` 入りの項目
- **強調**入りの項目

1. 番号付きリスト
2. 二つ目の項目

---

## コードブロック

```python
from dataclasses import dataclass

@dataclass
class Slide:
    title: str
    page: int = 1

    def render(self) -> str:
        return f"# {self.title} (p.{self.page})"
```

コードの前後の余白、フォント、背景のコントラストを確認する。

---

## テーブルと引用

| 項目 | 値 | 備考 |
|---|---|---|
| 背景色 | `--color-bg` | ベース |
| 文字色 | `--color-fg` | 本文 |
| アクセント | `--color-accent` | 見出し・強調 |

> 引用ブロックの見え方。左ボーダーや文字色の変化を確認する。

---

![bg right:40%](https://picsum.photos/seed/theme-check/800/1200)

## 画像分割レイアウト

`![bg right:40%]` との組み合わせ。

- テキスト側の余白は適切か
- 画像側と色がケンカしないか

---

<!-- _class: invert -->

## 反転バリアント(invert)

`<!-- _class: invert -->` を付けたスライド。
ダーク/ライトが反転しても **強調** や `code` が読めることを確認する。

---

## まとめ(最終ページ)

- ページ番号がすべてのページで正しい位置に出ているか
- ヘッダー/フッターと本文が重なっていないか
- 文字あふれがないか

**確認が終わったらこのデッキで PNG を書き出して目視すること。**
