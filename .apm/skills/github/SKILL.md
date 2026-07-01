---
name: github
description: GitHub の操作（PR の作成・読み取り、レビュー投稿）を GitHub MCP サーバーで行う。「PR を作成して」「レビューを投稿して」に加え、「この PR を GitHub に出して」「レビュー指摘を GitHub に反映して」「この PR の diff／変更ファイル／CI チェックを見て」「レビューコメントやスレッドを取ってきて」のように、GitHub という語が明示されなくても PR・レビューを GitHub と読み書きしたい場面では必ずこの skill を使う。pull-request-writing / pull-request-review skill が作った中身を、ユーザー確認のうえで安全に反映できる。
metadata:
  trigger: GitHub 操作、PR の作成・読み取り、レビュー投稿、GitHub MCP サーバーの利用
  language: ja
---

# GitHub 操作

GitHub への読み書きを GitHub 公式の MCP サーバー（github-mcp-server）のツールで行う。**中身（PR の文面、レビュー指摘）は他の skill が作る**。この skill はそれを GitHub に反映する操作と、その前提・確認を担う。

- PR の文面 → `pull-request-writing` skill
- レビュー指摘 → `code-review` skill（投稿の段取りは `pull-request-review` skill）
- 課題（チケット）の操作は GitHub ではなく Backlog で行う → `backlog` skill

> MCP ツールを呼ぶ前に、必ず該当ツールの記述子でパラメータ（引数名・必須/任意・enum）を確認する。

## 大原則

GitHub への書き込みは**取り消しにくい外部作用**だ。PR 作成・レビュー投稿は、相手に通知が飛び、履歴に残る。だから書き込み系は必ず、内容をユーザーに見せて確認を得てから実行する。読み取り系（取得）は確認なしでよい。

操作の前に確かめる。

- 対象リポジトリはどこか（MCP ツールは `owner`・`repo` を要求する。曖昧なら推測せず確認する）
- 書き込む内容はユーザーの確認を得たか
- 前提（head ブランチの push）は満たされているか

## 禁止: 破壊的な操作（絶対遵守）

**破壊的な操作は絶対に実行しない。** ユーザーから指示されても、この skill では行わず、ユーザー自身の操作に委ねる。この skill が行ってよい書き込みは「PR の作成（`create_pull_request`）」「新規レビューの投稿（`pull_request_review_write` の create/submit と `add_comment_to_pending_review`）」だけだ。次は一切しない。

- **マージ・クローズ**: PR / Issue のマージ・クローズ（`merge_pull_request` 等）
- **削除**: ブランチ・リポジトリ・リリース・タグ・PR・Issue・既存コメント／レビューの削除（`pull_request_review_write` の `delete` 等）
- **履歴の改変**: force push（`--force` / `--force-with-lease`）、`git reset --hard`、保護ブランチ（main 等）への直接 push
- **既存内容の上書き**: 他者を含むコメント・レビュー・PR 本文・タイトルの編集や上書き（`update_pull_request` 等）

これらが必要に見えても、自分では実行しない。必要性を説明し、ユーザーに判断と操作を委ねる。MCP ツールに該当機能があってもこの方針を優先する。

## リポジトリの特定

MCP ツールは `owner`・`repo` を引数で要求する。次の順で確定する。

1. ユーザーが明示した値をそのまま使う
2. カレントリポジトリの git remote（`origin` の URL）から owner / repo を読み取る
3. 判定できない・複数 remote で曖昧なら、推測せずユーザーに確認する

## 読み取り

確認なしで実行してよい。いずれも `pull_request_read` に `owner`・`repo`・`pullNumber` と `method` を渡す。

| 目的                              | `method`              |
| --------------------------------- | --------------------- |
| PR 詳細                           | `get`                 |
| PR の diff                        | `get_diff`            |
| PR の変更ファイル                 | `get_files`           |
| PR のレビューコメント（スレッド） | `get_review_comments` |
| PR のレビュー一覧                 | `get_reviews`         |
| PR のコメント（レビュー以外）     | `get_comments`        |
| PR の CI チェック                 | `get_check_runs`      |

- 複数を続けて使うときは並列で取得する。
- 件数が多いものはページング（`perPage`・`page`、`get_review_comments` は `after`）で取る。

## 書き込み

いずれも**ユーザーの確認を得てから**実行する。実行後は結果（PR の URL / 番号）を伝える。

### PR を作成する

- 前提: リモートに head ブランチが必要。未 push なら先に `git push -u origin HEAD`（Shell）で push する（この MCP サーバーは GitHub API 経由のため、ローカルコミットの push は git で行う）。
- `create_pull_request` を使う。

引数の対応:

| 引数    | 入れる値                            |
| ------- | ----------------------------------- |
| `owner` | owner                               |
| `repo`  | repo                                |
| `title` | PR タイトル                         |
| `body`  | PR 本文（改行・記号をそのまま渡す） |
| `head`  | head ブランチ（変更を含む側）       |
| `base`  | base ブランチ（マージ先）           |
| `draft` | draft にするなら `true`             |

- 文面・base・draft 可否は `pull-request-writing` skill のドラフトに従う。

### レビューを投稿する

インラインコメントは、保留中レビュー（pending review）を作り → インラインを足し → まとめて提出する 3 段で投稿する。

1. **保留中レビューを作る**: `pull_request_review_write`（`method: "create"`、`owner`・`repo`・`pullNumber`）。
2. **インラインコメントを足す**: `add_comment_to_pending_review` を指摘ごとに呼ぶ（`owner`・`repo`・`pullNumber`・`path`・`body`・`subjectType` が必須。行は `line`、追加行へは `side: "RIGHT"`、複数行は `startLine`＋`startSide`）。
3. **提出する**: `pull_request_review_write`（`method: "submit"`、`body` に総評、`event` に判定）。

- `event` の対応: MUST が1件以上 → `REQUEST_CHANGES` / それ以外の指摘 → `COMMENT` / 指摘なし → `APPROVE`。
- `line` は diff のハンクに現れる行のみ指定できる。
- インラインなしの総評だけなら、手順1→3だけでよい（`add_comment_to_pending_review` を省く）。
- 指摘の重要度・本文は `code-review` skill が決める。投稿の段取り全体は `pull-request-review` skill が呼び出す。

## クイックチェック

書き込み前に確認する。

- [ ] 破壊的な操作（マージ・クローズ・削除・force push・既存内容の改変）を含んでいないか?
- [ ] 対象リポジトリ（owner / repo）を推測でなく確定したか?
- [ ] 書き込む内容のユーザー確認を得たか?
- [ ] PR 作成前に head ブランチを push したか?
- [ ] インライン付きレビューを「保留中レビュー作成 → コメント追加 → 提出」の順で投稿しているか?

迷ったら最優先は上の2つ。破壊的でないこと・ユーザー確認を得たことだけは、書き込み前に必ず確かめる。書き込みは取り消しにくい外部作用だ。

## やらないこと

- 破壊的な操作（マージ・クローズ・削除・force push・既存内容の改変）を行う（→「禁止: 破壊的な操作」。指示されてもユーザーに委ねる）
- ユーザーの確認なしに書き込み系（PR 作成・レビュー投稿）を実行する
- 対象リポジトリ（owner / repo）を推測で埋める
- MCP ツールを呼ぶ前に記述子でパラメータを確認しない
- 中身（文面・指摘）をこの skill で作る（それは pull-request-writing / code-review の役割）
