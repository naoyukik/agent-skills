---
name: gitlab-reply-mr-threads
description: Replies to specific GitLab Merge Request review threads with glab. Use when responding to an individual review comment.
metadata:
  version: "1.0"
---

# GitLab MRレビューコメントへの返信方法

特定のレビューコメントスレッド（discussion）に対して返信を行う場合、以下の手順を実行する。

## 1. スレッドID（discussion_id）の取得

対象 MR の IID（番号）を指定して `glab mr note list` を実行し、返信対象のスレッドを特定する（`glab mr note list` は EXPERIMENTAL 機能。docs.gitlab.com/cli/mr/note/list/ に明記）。
スレッド構造と ID を正確に取得する場合は、Discussions API を `glab api` で呼び出す。`:fullpath` はカレントの git リポジトリから自動置換される。

```powershell
glab api "projects/:fullpath/merge_requests/123/discussions"
```

レスポンス内の `discussions[].id`（例: `abc123...`）がスレッドIDである。スレッド内の各ノートは `discussions[].notes[].body` で確認できる。

## 2. 返信コマンドの実行

取得したスレッドID (`<DISCUSSION_ID>`) と返信内容 (`<BODY>`) を用いて、以下の手順で返信を実行する。

1. **コマンド実行**: `run_shell_command` を使用し、`glab api` で Discussions API に返信を投稿する。PowerShell では複数行の本文をシングルクォートのヒアドキュメントで変数に格納して渡す。

```powershell
$body = @'
<BODY>

:robot: Commented by {AI}
'@
glab api -X POST "projects/:fullpath/merge_requests/123/discussions/<DISCUSSION_ID>/notes" -f body=$body
```

2. **結果確認**: `run_shell_command` の実行結果（例: `{"id":12345,"body":"...","author":{...}}`）から、投稿が成功したかを確認する。

## 署名プロトコル

本文の最後に必ず署名 `:robot: Commented by {AI}` を追加すること。署名は本文と改行で区切る（上記例のとおり、`<BODY>` の直後に空行を挟んで署名を付与する）。
`{AI}` は**置換用プレースホルダー**である。実行時に、実際に投稿する AI の名前（例: `Codex`）へ置換してから投稿すること。プレースホルダーのまま投稿しないこと。
PowerShell では Markdown のバッククォート消失防止のため、**必ずシングルクォートのヒアドキュメント（`@' ... '@`）を使用すること**。ダブルクォート（`@" ... "@`）を使用すると `<BODY>` 内の `$` やバッククォートが展開・消失されるため禁止する。

```powershell
$body = @'
<BODY>

:robot: Commented by {AI}
'@
glab api -X POST "projects/:fullpath/merge_requests/123/discussions/<DISCUSSION_ID>/notes" -f body=$body
```

## GitLab 固有の仕様差

- GitHub の `addPullRequestReviewThreadReply`（GraphQL）に相当する操作は、Discussions API の **ノート追加** エンドポイントで行う
  - 既存スレッドへの返信: `POST /projects/:id/merge_requests/:merge_request_iid/discussions/:discussion_id/notes`
  - 新規スレッドの作成: `POST /projects/:id/merge_requests/:merge_request_iid/discussions`
  - 通常コメント（スレッド外）: `POST /projects/:id/merge_requests/:merge_request_iid/notes`
- 対象リポジトリはカレントディレクトリの git リモートから自動検出される。ハードコード禁止
- 認証は `glab` の設定（`GITLAB_TOKEN` または保存済み認証）に依存する。トークンをコマンドに埋め込まないこと
