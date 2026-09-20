---
name: gitlab-sub-issue-linker
description: Manages GitLab issue links (relates_to, blocks, or is_blocked_by) with glab. Use when relating existing issues; parent/child hierarchy is out of scope.
metadata:
  version: "1.0"
---

# GitLab Issue リンク操作 (Issue Linker)

このスキルは GitLab の Issue links API を使用して、issue 間のリンクを作成・一覧・削除する手順を定める。

## 前提知識: child/parent 階層は API 非対応

GitLab REST API の Issue links API がサポートする `link_type` は以下の 3 種のみである。

| link_type | 意味 |
| :--- | :--- |
| `relates_to` | 対等な関連（双方向）。Free プランで利用可（GitLab 13.4 以降） |
| `blocks` | 対象 issue の作業をブロックする |
| `is_blocked_by` | 対象 issue の作業がブロックされている |

**真の child/parent 階層（Work Items の子課題）は REST API で作成できない。**
これは UI の Work Items 機能のみで対応しており（Free プランで利用可）、API 非対応の既知問題として GitLab issue 551181 で報告・未解決である。
したがって、子（child）/ parent に相当する関係が必要な場合は、以下の代替運用を取る。

- **階層・依存関係の表現**: `blocks` / `is_blocked_by` を使用する（親 = 先に完了すべき issue が `blocks`、子 = 後続の issue が `is_blocked_by`）
- **対等な関連の表現**: `relates_to` を使用する
- 真の子課題ツリーが必要な場合は UI で作成し、本スキルの API 操作は適用しない

## 対象リポジトリの自動検出

`glab api` の `:fullpath` プレースホルダはカレントディレクトリの git リモートから自動置換される。リポジトリ名・ユーザー名をハードコードしてはならない。

```powershell
git remote get-url origin
# 例: https://gitlab.com/<GROUP>/<REPO>.git
```

## 1. リンクの一覧取得

```powershell
glab api "projects/:fullpath/issues/<ISSUE_IID>/links"
```

レスポンスの各要素に `id`（issue link ID）、`source_issue`、`target_issue`、`link_type` が含まれる。

## 2. リンクの作成

```powershell
glab api -X POST "projects/:fullpath/issues/<ISSUE_IID>/links" -F target_issue_iid=<TARGET_IID> -F link_type=blocks
```

`glab api` の `-f` / `-F` パラメータ値はプレースホルダー展開されない（URL パスの `:fullpath` のみ展開される）。そのため `target_project_id=:id` のような指定は文字列 `:id` のまま送信されて失敗する。
同一プロジェクト内のリンクでは `target_project_id` は省略可能である（省略時は現在のプロジェクトが使われる）。別プロジェクト間のリンクでは、`projects/:fullpath` のプロジェクト ID を事前取得して実 ID を渡すこと。

```powershell
# 別プロジェクト間の場合: 実プロジェクト ID を取得して渡す
$targetProjectId = (glab api "projects/:fullpath" --jq ".id" | Out-String).Trim()
glab api -X POST "projects/:fullpath/issues/<ISSUE_IID>/links" -F target_project_id=$targetProjectId -F target_issue_iid=<TARGET_IID> -F link_type=blocks
```

| パラメータ | 説明 |
| :--- | :--- |
| `<ISSUE_IID>` | リンク元（親側）の issue の IID（内部番号） |
| `target_issue_iid` | リンク先（子側）の issue の IID |
| `link_type` | `relates_to` / `blocks` / `is_blocked_by`（省略時は `relates_to`） |

**例**: issue #5 が issue #7 をブロックする関係を作る場合

```powershell
glab api -X POST "projects/:fullpath/issues/5/links" -F target_issue_iid=7 -F link_type=blocks
```

## 3. リンクの削除

```powershell
glab api -X DELETE "projects/:fullpath/issues/<ISSUE_IID>/links/<ISSUE_LINK_ID>"
```

`<ISSUE_LINK_ID>` は一覧取得（手順 1）で得た `id` を使用する。

## 4. Issue 作成時の同時リンク

`glab issue create` は `--linked-issues` と `--link-type` フラグを持ち、作成と同時にリンクを張れる。`--link-type` のデフォルトは `relates_to` である。
**注意**: 新規作成される Issue がリンク元（source）になる。子課題から親への関係は `blocks` ではなく `is_blocked_by` を指定する（`blocks` にすると「子が親をブロックする」という逆関係になる）。

```powershell
glab issue create -t "feat: 子課題" --linked-issues 5 --link-type is_blocked_by --yes
```

## 認証とハードコード禁止

- 認証は `glab` の設定（`GITLAB_TOKEN` または `glab auth login` の保存済み認証）に依存する。トークンをコマンドに埋め込まないこと
- リポジトリ名・ユーザー名・プロジェクトパスをハードコードせず、`:fullpath` プレースホルダで自動検出すること
