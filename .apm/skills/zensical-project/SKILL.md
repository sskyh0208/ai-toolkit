---
name: zensical-project
description: Zensical（Material for MkDocs 後継の静的サイトジェネレータ）でドキュメントサイトを新規作成・プレビュー・ビルド・公開する。「Zensical でドキュメントサイトを作って」「zensical new / serve / build を実行したい」「ドキュメントを GitHub Pages に公開したい」のほか、zensical.toml があるプロジェクトでサイトの起動・ビルド・リンク検証をする場面で必ずこの skill を使う。設定の書き方は zensical-config、Markdown 記法は zensical-authoring を参照。
metadata:
  trigger: Zensical サイト作成、zensical new/serve/build、ドキュメント公開、リンク検証
  language: ja
---

# Zensical プロジェクト運用

Zensical プロジェクトの作成・プレビュー・ビルド・公開を行う。コマンドはすべて Zensical 0.0.47 で動作確認済み。

## インストール

Python パッケージとして PyPI から入れる。venv 推奨。

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install zensical
```

uv を使う場合は `uv init && uv add --dev zensical` の後、常に `uv run zensical` で実行する。**uv の symlink モードは非対応**（公式の既知の制限）。

## プロジェクト作成

```sh
zensical new <ディレクトリ>   # 省略時はカレントディレクトリ
```

生成物:

```text
.
├─ .github/workflows/docs.yml   # GitHub Pages 公開用ワークフロー（push で build → deploy）
├─ docs/
│  ├─ index.md
│  └─ markdown.md               # 記法サンプル。不要なら削除してよい
└─ zensical.toml                # 全設定にコメントと docs URL 付きのテンプレート
```

- 既存ファイルは上書きしない。`zensical.toml` が既にあるとエラーで終了する。
- 生成される `zensical.toml` は推奨拡張がすべて有効な状態。設定の変更は `zensical-config` skill を参照。

## プレビュー（執筆ループ）

```sh
zensical serve                          # localhost:8000
zensical serve --dev-addr localhost:8123  # ポート変更。-o でブラウザを開く
```

ソース変更で自動リビルド・自動リロードされる。エージェントが表示確認する場合はバックグラウンドで起動して curl で叩く:

```sh
zensical serve --dev-addr localhost:8123 &
sleep 4
curl -s -o /dev/null -w "%{http_code}" http://localhost:8123/<page>/   # 200 を確認
```

serve はプレビュー専用。本番配信には使わない。

## ビルドと検証

```sh
zensical build            # site/ に静的サイトを生成
zensical build --strict   # 警告（リンク切れ等）があれば exit 1
zensical build --clean    # キャッシュ削除
```

`--strict` はリンク切れをソース位置付きで報告する（例: `syntax-test.md:72:20 page does not exist`）。**ページを書いたら必ず `zensical build --strict` を通すこと**。生成された CI ワークフローも `zensical build --clean` を実行するため、ローカルで通れば CI も通る。

## 公開

- **GitHub Pages**: `zensical new` が生成する `.github/workflows/docs.yml` をそのまま使う。リポジトリの Settings → Pages で Source を「GitHub Actions」にするだけでよい。対象ブランチは `master`/`main`。
- その他のホスティング: `site/` ディレクトリは自己完結の静的ファイルなので、任意の Web サーバ・CDN に置ける。
- ローカルファイルとして配布する場合（zip 等）は offline プラグインを使う（`zensical-config` 参照）。

## Gotchas（実行して確認した挙動・公式の既知の制限）

- `site_name` は必須設定。欠けるとビルドできない。
- `docs_dir = "."` にはできない（公式の一時的制限）。ソースは必ずサブディレクトリに置く。
- `docs_dir` 内に `README.md` と `index.md` を両方置くと挙動が未定義。どちらか一方にする。
- `nav` を明示定義したページに `# 見出し` がないと、h1 がファイル名になる（MkDocs と挙動が異なる既知の差分）。ページ先頭に必ず h1 を書く。
- MkDocs 互換: `zensical.toml` の代わりに既存の `mkdocs.yml` をそのまま読める。ただし `hooks` / `exclude_docs` / `draft_docs` / `not_in_nav` / `remote_branch` / `remote_name` は未対応。

## 参照

- 設定（zensical.toml）: `zensical-config` skill
- Markdown 記法: `zensical-authoring` skill
- 公式ドキュメント: https://zensical.org/docs/
