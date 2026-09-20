---
name: gitlab-create-mr
description: Creates draft GitLab Merge Requests with glab. Use when a branch is ready for review.
metadata:
  version: "1.0"
---

# GitLab Merge Request (Draft MR) 作成ガイドライン

1. 情報収集と分析
  あなたが以下の情報を自動的に収集・分析する。
  以下のコマンドリストは、**修正や最適化を行わず、文字列として完全に一致する状態で**実行すること。別コマンドへの置換は禁止する。
  セキュリティエラーが発生するので `&&` などでコマンドを接続することは禁止する。

    - 現在のブランチ名: !{git rev-parse --abbrev-ref HEAD}
    - mainからの差分コミット: !{git log main..HEAD --oneline --pretty=format:"%h - %s"}
    - リモートURL: !{git remote get-url origin}
    - MRテンプレート: !{cat "$(git rev-parse --show-toplevel)/.gitlab/merge_request_templates/*.md"} （存在しない場合は取得不要）
    - チケット番号: !{git rev-parse --abbrev-ref HEAD | grep -oE '^[0-9]+'}
    - 対象リポジトリ（自動検出）: !{git remote get-url origin} からリポジトリを特定する。ハードコード禁止
    - Actual Code Changes (Diff) ノイズを除外し、本質的な変更差分のみを取得する: !{git diff main..HEAD -- . ':(exclude)package-lock.json' ':(exclude)yarn.lock' ':(exclude)*.lock'}

2. Issueの確認
  チケット番号が存在する場合、Issueの内容を確認するため、`glab issue view <チケット番号>` を実行して背景情報を抽出する。
  対象リポジトリはカレントディレクトリの git リモートから自動検出されるため、`-R` は原則不要である。

3. 推論と構築 (Reasoning)
  取得した情報を以下の優先順位で統合し、MRの内容を構築せよ。

    1. **Diff情報**: 実際のコード変更が最優先。「何をしたか」の事実とする。
    2. **Issue情報**: 「なぜそれをしたか」の背景情報として利用する。
    3. **Commit情報**: 作業の流れを補足する情報として利用する。

4. MR内容の生成
  分析結果に基づき、以下の項目を生成する。

    * **MRタイトル**: 変更内容の要点を的確に表現したタイトルを生成する。
    * **MR本文**: 読み込んだテンプレートに基づき、変更の概要、背景、詳細などを具体的に記述する。テンプレートがない場合は適当に作る。
    * **Issue連携**: 関連するIssueを自動的に閉じるため、本文内に必ず `Closes #チケット番号` の形式で記述を含めること。
      - チケット番号の頭には `#` を付与する。
      - GitLab はこのクローズパターン（Close / Fixes / Resolves 等）を MR マージ時に検出し、対象 Issue を自動クローズする。

5. コマンド生成と実行
  GitLab CLI (`glab`) を使用して Draft MR を作成する。

    * `--draft`: Draft MRとして作成する（`--wip` でも可）。
    * `--title`: 生成したMRタイトルを設定する。
    * `--description-file`: 生成したMR本文を設定する（ファイル経由を推奨）。
    * `--target-branch`: ベースブランチは常に`main`とする。省略時はプロジェクトのデフォルトブランチが使われる。
    * `--source-branch`: 現在のブランチ（デフォルト）を使用する。
    * `--assignee`: "@me" を設定する。PowerShell では `@` が splatting 演算子として解釈されるため、`-a '@me'` のように引用符で囲むこと。
    * `--yes`: 確認プロンプトを省略する。

    a. **GitLab CLI (glab) を使用する**:
      - `run_shell_command` を使用し、上記の仕様で `glab mr create` を実行する。
      - **注意**: 引数のパース精度を優先するため、`cmd /c` を介さず直接実行すること。
    b. **いずれも使用できない場合**:
      - 実行すべき `glab mr create` コマンドとその引数をテキストとして出力する。

6. 結果報告
   MRが正常に作成されたことをユーザーに報告する。

## PowerShell ヒアドキュメントの注意点

MR本文に Markdown のインラインコード（バッククォート `` ` ``）を含める場合は、**シングルクォートのヒアドキュメント（`@' ... '@`）を使用すること。**

```powershell
$body = @'
## 概要
- `GraphDocument` の更新系ユースケースを追加した

Closes #26
'@
$body | Out-File -Encoding UTF8 temporary.local\mr_body.md
glab mr create --draft -b main -a '@me' --yes --title "feat: 更新系ユースケースを追加" --description-file temporary.local\mr_body.md
```

## GitLab 固有の仕様差

- GitHub の Pull Request は GitLab では **Merge Request (MR)** と呼ぶ。番号参照は `!番号` 形式
- Draft は `--draft`（旧 `--wip`）で作成する。タイトルに "Draft:" は自動付与されないため、発言や通知から Draft 状態を判別する
- `Closes #番号` は MR がプロジェクトのデフォルトブランチ（通常 main）へ**マージされた時点**で Issue をクローズする。Draft 作成時点ではクローズされない
- MRテンプレートは `.gitlab/merge_request_templates/` に配置する（GitHub の `.github/pull_request_template.md` に相当）
- 対象リポジトリはカレントディレクトリの git リモートから自動検出される。別リポジトリのみ `-R OWNER/REPO` で指定する
- 認証は `glab` の設定（`GITLAB_TOKEN` または `glab auth login` の保存済み認証）に依存する。トークンをコマンドに埋め込まないこと
