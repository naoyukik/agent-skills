---
name: gitlab-check-mr-review-comments
description: Analyzes and triages GitLab Merge Request review comments with glab. Use when reviewer feedback requires follow-up work.
metadata:
  version: "1.0"
---

# steps
- name: Review MR Comments
  description: GitLab のマージリクエストレビューコメントを確認し、必要な修正や改善を特定します。
- name: Document Changes
  description: 変更内容をドキュメント化し、将来の参考のために記録します。

# レビューコメントの取得方法
情報の正確な取得とコンテキスト（トークン）効率を最大化するため、以下の手順で GitLab CLI (`glab`) を使用してください。

1. **コマンド実行**: 対象 MR の IID（番号）またはブランチ名を指定して `glab mr note list` を実行します（`glab mr note list` は EXPERIMENTAL 機能。docs.gitlab.com/cli/mr/note/list/ に明記）。対象リポジトリはカレントディレクトリの git リモートから自動検出されます。

```powershell
glab mr note list 123
# またはブランチ名で指定
glab mr note list feat/my-branch
```

2. **詳細なスレッド構造が必要な場合**: 差分位置とスレッド ID を含む詳細は Discussions API で取得します。`glab api` の `:fullpath` プレースホルダはカレントの git リポジトリから自動置換されます。

```powershell
glab api "projects/:fullpath/merge_requests/123/discussions"
```

3. **内容確認**: `run_shell_command` の実行結果から、レビュー内容を直接確認してください。

# レビューコメントの報告
レビューコメントを取得したら、ユーザーに以下のフォーマットで報告してください。
> **【MR Review Comments】**
> - **Total Comments**: `X` 件のレビューコメントが見つかりました
> - **Comments Summary**:
> - 1. `コメント内容1` - 指摘内容の概要
> - 2. `コメント内容2` - 指摘内容の概要

# 妥当性やスコープの吟味
レビューコメントの内容を吟味し、実装すべき修正点や改善点を特定してください。必要に応じて、ユーザーに確認を取ることも検討してください。
また、指摘を「今回のMRの変更が影響を及ぼし、今回のMRで必須なもの」と「将来的な改善（別タスク）に回すべきもの」に明確に切り分けます。切り分けは後続の `分類と対応方針` を参照。
吟味した結果、トータル的に実装する必要がないと判断すれば、その旨をユーザーに報告してください。

## 分類と対応方針
レビューコメントを以下のカテゴリに分類し、対応方針を決定せよ。
1.  Critical / Safety: セキュリティ、クラッシュ、安全性、コンパイル警告に関わるもの。
    -> Action*: 今回のMRで必ず修正する。
2.  Bug Fix: MRの機能が正しく動作しない原因となるもの。
    -> Action*: 今回のMRで修正する。
3.  Refactoring / Design: 設計の美しさ、カプセル化、将来の拡張性に関するもの。
    -> Action*: 今回は修正せず、"Future Ticket" として起票する。
4.  Optimization: パフォーマンス改善（現状でボトルネックでない場合）。
    -> Action*: 今回は修正せず、"Future Ticket" として起票する。
5.  Nitpick: 些細な指摘（スコープ外のエッジケースなど）。
    -> Action*: 無視、または丁重に断る。

# タスク管理

タスクを忘れずに実行できるように、Conductorのtrackにタスクを追加してください。
現在進行中のtrackがある場合はそこに追加。アーカイブされている場合等で現在進行中のtrackが存在しない場合、新規にtrackを作成してください。

# GitLab 固有の仕様差

- GitHub の Pull Request レビューは GitLab では **Merge Request のノート（コメント）** として扱われる
- `glab mr note list` は MR のノートを discussion 単位で返す（1 ノートの通常コメントも discussion として返ることがある）。レビューコメントの件数カウントは `discussions[].notes[]` のノート単位で行うこと。差分位置などの詳細は `glab api "projects/:fullpath/merge_requests/<iid>/discussions"` で取得する
- ノートの識別子は note ID（`discussions[].notes[].id`）であり、スレッド単位は discussion ID（`discussions[].id`）。返信には discussion ID が必要（`gitlab-reply-mr-threads` を参照）
- 対象リポジトリはカレントディレクトリの git リモートから自動検出される。ハードコード禁止
- 認証は `glab` の設定（`GITLAB_TOKEN` または保存済み認証）に依存する。トークンをコマンドに埋め込まないこと

# 終了
タスク管理まで完了したら、実装は行わずにこのスキルを終了してください
