# zensical.toml 設定カタログ

Zensical 公式ドキュメント（https://zensical.org/docs/setup/）の設定を項目別にまとめたもの。すべて `zensical.toml` 形式。`mkdocs.yml` を使う場合は同名キーの YAML 表現に読み替える。

## 基本設定（[project]）

| キー | 説明 |
|---|---|
| `site_name` | サイト名。**唯一の必須設定** |
| `site_url` | 正規 URL。instant navigation / instant previews / カスタムエラーページの前提 |
| `site_description` | HTML head の description。ページ frontmatter の description が優先 |
| `site_author` | HTML head の author |
| `copyright` | フッターの著作権表記。HTML 可 |
| `docs_dir` | ソースディレクトリ（既定 `docs`）。**`.` は不可** |
| `site_dir` | 出力先（既定 `site`） |
| `use_directory_urls` | `true`（既定）: `/usage/` 形式。`false`: `/usage.html` 形式。offline ビルドでは自動で `false` |
| `dev_addr` | serve のバインド先（既定 `localhost:8000`） |
| `watch` | serve 中に追加監視するパスの配列。変更でフルリビルド |
| `repo_url` / `repo_name` | ヘッダーのリポジトリリンク |
| `edit_uri` | 「このページを編集」の URI（例 `edit/main/docs/`） |
| `extra_css` / `extra_javascript` | docs_dir 相対の追加アセット配列 |
| `nav` | 明示ナビゲーション（後述） |
| `validation` | リンク検証（後述） |

MkDocs 互換で未対応: `remote_branch`, `remote_name`, `exclude_docs`, `draft_docs`, `not_in_nav`, `hooks`。

## テーマ（[project.theme]）

```toml
[project.theme]
variant = "classic"        # "modern"（既定）| "classic"（Material for MkDocs と同じ見た目）
language = "ja"            # サイト言語
direction = "ltr"          # ltr / rtl
logo = "images/logo.png"   # docs_dir 相対
favicon = "images/favicon.png"
custom_dir = "overrides"   # テンプレートオーバーライド用

[project.theme.font]
text = "Inter"
code = "JetBrains Mono"
# font = false  # Google Fonts の自動読み込みを無効化（データプライバシー対応）

[project.theme.icon]
logo = "lucide/smile"      # 画像の代わりにバンドルアイコンを使う場合
repo = "fontawesome/brands/git-alt"
edit = "material/pencil"
view = "material/eye"
```

## 色（palette）

```toml
# 単一スキーム
[project.theme.palette]
scheme = "default"         # "default"（ライト）| "slate"（ダーク）
primary = "indigo"         # プライマリカラー
accent = "indigo"          # アクセントカラー

# ライト/ダーク切り替えトグル（配列にする）
[[project.theme.palette]]
scheme = "default"
toggle.icon = "lucide/sun"
toggle.name = "Switch to dark mode"

[[project.theme.palette]]
scheme = "slate"
toggle.icon = "lucide/moon"
toggle.name = "Switch to light mode"
```

OS 設定に追従させるには各 palette に `media = "(prefers-color-scheme: light)"` / `"(prefers-color-scheme: dark)"` を付ける。3 状態（自動/ライト/ダーク）は `media = "(prefers-color-scheme)"` の palette を先頭に追加する。

カスタム色は `primary = "custom"` にして extra_css で CSS 変数を上書きする。

## features 一覧（[project.theme] features = [...]）

ナビゲーション:

| フラグ | 効果 |
|---|---|
| `navigation.instant` | 内部リンクを XHR 化（SPA 風。site_url 必須） |
| `navigation.instant.prefetch` | ホバー時に先読み |
| `navigation.instant.progress` | 読み込みプログレスバー |
| `navigation.tracking` | スクロールで URL のアンカーを更新 |
| `navigation.tabs` | 上位セクションをヘッダー下のタブに |
| `navigation.tabs.sticky` | タブを常に表示 |
| `navigation.sections` | 上位セクションをサイドバーのグループに |
| `navigation.expand` | サイドバーを全展開 |
| `navigation.path` | パンくずリスト |
| `navigation.prune` | 非表示ナビを HTML から除去（サイズ削減） |
| `navigation.indexes` | セクションに index ページを直接割り当て |
| `navigation.footer` | フッターに前へ/次へ |
| `navigation.top` | トップに戻るボタン |
| `toc.follow` | 目次がスクロールに追従 |
| `toc.integrate` | 目次を左サイドバーに統合 |

コンテンツ:

| フラグ | 効果 |
|---|---|
| `content.code.copy` | コードコピーボタン |
| `content.code.select` | 行選択ボタン |
| `content.code.annotate` | コードアノテーション |
| `content.action.edit` / `content.action.view` | ページ編集/ソース表示ボタン（repo_url 必須） |
| `content.tabs.link` | 同ラベルのコンテンツタブを連動 |
| `content.tooltips` | title 属性を改良ツールチップで表示 |
| `content.footnote.tooltips` | 脚注をツールチップ表示 |

その他: `header.autohide`（ヘッダー自動非表示）、`announce.dismiss`（お知らせバーを閉じられる）、`search.highlight`（検索語ハイライト）。

## ナビゲーション（nav）

省略すると docs_dir のディレクトリ構造から自動生成。明示する場合:

```toml
[project]
nav = [
  {"Home" = "index.md"},
  {"About" = [
     "about/index.md",          # navigation.indexes 有効時はセクションの index に
     {"Vision" = "about/vision.md"},
  ]},
  {"GitHub Repo" = "https://github.com/user/repo"},  # 外部リンク可
]
```

## 検索・タグ

```toml
# 検索ハイライト
[project.theme]
features = ["search.highlight"]

# タグ（アイコン識別子の割り当て）
[project.extra.tags]
Compatibility = "compat"
```

ページ側の除外・タグ付けは frontmatter で行う（`zensical-authoring` 参照）。

## リンク検証（validation）

```toml
[project.validation]
invalid_links = true          # リンク先ページの存在チェック（既定で警告）
invalid_link_anchors = true   # アンカーの存在チェック
# validation = false          # 全チェック無効化
```

`zensical build --strict` で警告を exit 1 に昇格。

## フッター・ヘッダー

```toml
[[project.extra.social]]                 # フッターのソーシャルリンク
icon = "fontawesome/brands/github"
link = "https://github.com/user/repo"
name = "GitHub"                          # 任意（ツールチップ）

[project.extra]
generator = false                        # "Made with Zensical" 表記を消す
homepage = "https://example.com"         # ロゴのリンク先を site_url 以外に
```

## 多言語

```toml
[project.extra]
alternate = [
    { name = "English", link = "/en/", lang = "en" },
    { name = "日本語", link = "/ja/", lang = "ja" },
]
```

## オフライン配布

```toml
[project.plugins.offline]
```

`use_directory_urls` が自動で false になり、`file://` で閲覧可能なサイトを生成。search は iframe-worker の polyfill で動作。

## Markdown 拡張（[project.markdown_extensions]）

`zensical new` が生成する既定セット（推奨。これを基点に追記する）:

```toml
[project.markdown_extensions.abbr]
[project.markdown_extensions.admonition]
[project.markdown_extensions.attr_list]
[project.markdown_extensions.def_list]
[project.markdown_extensions.footnotes]
[project.markdown_extensions.md_in_html]
[project.markdown_extensions.toc]
permalink = true
[project.markdown_extensions.pymdownx.arithmatex]
generic = true
[project.markdown_extensions.pymdownx.betterem]
[project.markdown_extensions.pymdownx.caret]
[project.markdown_extensions.pymdownx.details]
[project.markdown_extensions.pymdownx.emoji]
emoji_generator = "zensical.extensions.emoji.to_svg"
emoji_index = "zensical.extensions.emoji.twemoji"
[project.markdown_extensions.pymdownx.highlight]
anchor_linenums = true
line_spans = "__span"
pygments_lang_class = true
[project.markdown_extensions.pymdownx.inlinehilite]
[project.markdown_extensions.pymdownx.keys]
[project.markdown_extensions.pymdownx.magiclink]
[project.markdown_extensions.pymdownx.mark]
[project.markdown_extensions.pymdownx.smartsymbols]
[project.markdown_extensions.pymdownx.superfences]
custom_fences = [
  { name = "mermaid", class = "mermaid", format = "pymdownx.superfences.fence_code_format" }
]
[project.markdown_extensions.pymdownx.tabbed]
alternate_style = true
combine_header_slug = true
[project.markdown_extensions.pymdownx.tasklist]
custom_checkbox = true
[project.markdown_extensions.pymdownx.tilde]
```

既定に含まれないがよく足すもの（既定で動かないことを実ビルドで確認済み）:

```toml
[project.markdown_extensions.pymdownx.snippets]            # 外部ファイル埋め込み（--8<--）・用語集
auto_append = ["includes/abbreviations.md"]                # 全ページに用語集を追記
[project.markdown_extensions.pymdownx.blocks.caption]      # 画像キャプション /// caption ///
```

テーブル（`tables`）は生成される設定に記載がなくても既定で動作する（確認済み）。明示したい場合は `[project.markdown_extensions.tables]` を足す。

拡張を全部無効化して MkDocs の既定に合わせる: `markdown_extensions = {}`。

## Zensical 独自拡張

```toml
# GLightbox: 画像クリックでライトボックス表示
[project.markdown_extensions.zensical.extensions.glightbox]
# auto = false        # true（既定）: 全画像対象。false: .on-glb クラスの画像のみ
# auto_themed = true  # #only-light/#only-dark をギャラリー分離

# Macros: Jinja テンプレート・変数展開
[project.markdown_extensions.zensical.extensions.macros]
# module_name = "macros"                    # カスタムマクロの Python モジュール
# include_yaml = ["data/variables.yml"]     # YAML を変数として読み込み
# include_dir = "includes"                  # {% include %} の検索パス

# mkdocstrings 互換（API ドキュメント生成。要 pip install mkdocstrings-python）
[project.markdown_extensions.zensical.extensions.mkdocstrings]

# Instant previews: 内部リンクのホバープレビュー
[project.markdown_extensions.zensical.extensions.preview]
configurations = [
    { targets.include = ["setup/*"] }
]
```

## アナリティクス・プライバシー

```toml
[project.extra.analytics]
provider = "google"
property = "G-XXXXXXXXXX"

# クッキー同意バナー
[project.extra.consent]
title = "Cookie consent"
description = "..."
```

## カスタマイズ

```toml
[project]
extra_css = ["stylesheets/extra.css"]        # docs/stylesheets/extra.css を追加
extra_javascript = ["javascripts/extra.js"]

[project.theme]
custom_dir = "overrides"                     # テンプレート上書き（main.html 等を置く）
```

数式（MathJax/KaTeX）や sortable table など extra_javascript を使う具体例は https://zensical.org/docs/authoring/math/ と https://zensical.org/docs/authoring/data-tables/ を参照。

## バージョニング（mike）

`pip install mike` の上で:

```toml
[project.extra.version]
provider = "mike"
# alias = true
```

公開は `mike deploy --push --update-aliases 1.0 latest`、既定版は `mike set-default --push latest`。
