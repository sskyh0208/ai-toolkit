---
name: zensical-config
description: Zensical の zensical.toml を設定する。「サイト名・テーマ・色・フォントを変えたい」「ナビゲーションタブ／セクション／目次を調整したい」「検索・タグ・リンク検証・Markdown 拡張を設定したい」「mkdocs.yml から移行したい」など、Zensical プロジェクトの設定ファイルを追加・変更する場面で必ずこの skill を使う。コマンド実行は zensical-project、本文の記法は zensical-authoring を参照。
metadata:
  trigger: zensical.toml 設定、Zensical テーマ・ナビゲーション・検索・拡張の設定、MkDocs 移行
  language: ja
---

# Zensical 設定（zensical.toml）

Zensical プロジェクトの設定はプロジェクトルートの `zensical.toml` に書く。設定項目の一覧と TOML スニペットは [references/config-reference.md](references/config-reference.md) にある。個別項目を書く前に必ず該当セクションを確認する。

## 大原則

1. **既存の `zensical.toml` を編集する**。`zensical new` が生成するテンプレートには全設定がコメント付きで入っているので、該当コメントを外して値を変える方が新規に書くより安全。
2. **すべての設定は `[project]` スコープ配下**。`[project.theme]`、`[project.markdown_extensions.*]` のようにネストする。
3. **機能トグルは `features` 配列**。ナビゲーションやコードブロックの挙動の多くは `[project.theme]` の `features = [...]` に文字列を足すだけで有効になる。
4. **変更したら `zensical build --strict` で確認する**（`zensical-project` 参照）。設定ミスはビルド時に検出される。

## 最小構成

```toml
[project]
site_name = "My site"                  # 唯一の必須設定
site_url = "https://example.com"       # 公開するなら必須級（instant navigation 等の前提）
```

## よく使うパターン

```toml
[project]
site_name = "My docs"
site_url = "https://example.com"
repo_url = "https://github.com/user/repo"   # ヘッダーにリポジトリリンク
edit_uri = "edit/main/docs/"                # content.action.edit と併用

[project.theme]
language = "ja"                             # サイト言語（60+ 言語対応）
features = [
    "navigation.sections",   # サイドバーにセクション見出し
    "navigation.top",        # トップに戻るボタン
    "content.code.copy",     # コードコピーボタン
    "search.highlight",      # 検索語ハイライト
]

# ライト/ダーク切り替え
[[project.theme.palette]]
scheme = "default"
toggle.icon = "lucide/sun"
toggle.name = "Switch to dark mode"

[[project.theme.palette]]
scheme = "slate"
toggle.icon = "lucide/moon"
toggle.name = "Switch to light mode"
```

## MkDocs（Material for MkDocs）からの移行

- Zensical は `mkdocs.yml` をネイティブに読めるため、まず無変更でビルドを試す。
- 見た目を Material for MkDocs と揃えるにはテーマを classic にする: `[project.theme]` に `variant = "classic"`。
- 未対応設定（`hooks`、`exclude_docs`、`draft_docs`、`not_in_nav`、`remote_branch`、`remote_name`）は削除するか代替を検討する。
- ビルドが通らない場合は `markdown_extensions = {}` で拡張の既定セットを無効化して MkDocs の既定に合わせる。

## Gotchas

- `docs_dir = "."` は不可。ソースは `docs/` などのサブディレクトリに置く。
- Zensical の拡張既定セットは MkDocs より多い（MkDocs は `meta`/`toc`/`tables`/`fenced_code` のみ）。挙動差が出たら上記の方法で既定を無効化する。
- `[project.markdown_extensions]` を 1 つでも明示すると既定セット全体が置き換わる。部分的に足したい場合は `zensical new` が生成する既定ブロック（references 参照）を丸ごと維持した上で追記する。
- 記法ごとに必要な拡張が異なる。記法→拡張の対応は `zensical-authoring` の references に一覧がある。

## 参照

- 設定カタログ: [references/config-reference.md](references/config-reference.md)
- 公式 setup ガイド: https://zensical.org/docs/setup/basics/
