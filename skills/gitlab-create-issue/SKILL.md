---
name: gitlab-create-issue
description: Creates or updates GitLab issues with glab using Japanese title and body templates. Use when managing issues under these conventions.
metadata:
  version: "1.0"
---

# GitLab Issue作成ガイドライン

## タイトル形式

```
type: 日本語での簡潔な説明
```

接頭辞（type）は半角小文字とする。

## Type一覧

| Type | 用途 |
|------|------|
| `feat` | 新機能の追加 |
| `fix` | バグ修正 |
| `refactor` | リファクタリング（機能変更を伴わないコードの整理・改善） |
| `style` | コードの意味に影響しない修正（空白、フォーマット等） |
| `docs` | ドキュメントの更新 |
| `test` | テストの追加・修正 |
| `chore` | ビルドプロセスやツール、依存ライブラリの更新 |

## 内容テンプレート

```markdown
## 背景・目的 (Background / Purpose)

なぜこの作業が必要なのか、現状の課題は何かを記述する。
関連する設計指針やガイドラインがある場合は明記する。

## タスク (Tasks)

- [ ] 具体的な作業1
- [ ] 具体的な作業2
- [ ] 具体的な作業3

## ゴール (Goal)

どのような状態になれば、この Issue を「完了（Closed）」とみなすことができるかを定義する。
```

## 運用ルール

- **適切な粒度**: 1つのIssueで扱う範囲を大きくしすぎない。多岐にわたる場合は分割を検討
- **リファクタリングの根拠**: `refactor` Issueでは、どのコード規約やアーキテクチャ方針に基づいた変更なのかを明示
- **透明性**: 思考過程や途中の気付きはIssueのコメント欄に随時記録

## 対象リポジトリの自動検出

`glab` はカレントディレクトリの git リモートから対象リポジトリを自動検出する。リポジトリ名・ユーザー名をハードコードしてはならない。

```powershell
git remote get-url origin
# 例: https://gitlab.com/<GROUP>/<REPO>.git
```

別リポジトリを操作する場合のみ `-R` フラグで指定する（`glab issue create -R OWNER/REPO ...`）。

## GitLab CLI (glab) による Issue 作成時の注意点

PowerShell 環境から `glab issue create` や `glab issue update` を使用して Issue の本文を作成・更新する場合、ヒアドキュメントを利用することが推奨される。
その際、Markdown内のインラインコード（バッククォート `` ` ``）がPowerShellのエスケープ文字として解釈され消失してしまうのを防ぐため、**必ずシングルクォートのヒアドキュメント（`@' ... '@`）を使用すること。**

**正しい例:**
```powershell
$body = @'
## ゴール (Goal)
- `vte::Perform` の `osc_dispatch` を実装すること
'@
$body | Out-File -Encoding UTF8 temporary.local\issue_body.md
glab issue create --title "feat: xxx" --description-file temporary.local\issue_body.md
```

`--description-file` は `-` を指定することで標準入力からも読み取れる（`cat description.md | glab issue create -t "title" --description-file -`）。
オプションの要否は `glab issue create --help` で確認すること。

## GitLab 固有の仕様差

- Issue の識別子はプロジェクト内の IID（内部番号）であり、`#番号` 形式で参照する
- `glab issue create --linked-issues <IID>` と `--link-type`（`relates_to` / `blocks` / `is_blocked_by`）で、作成と同時に issue リンクを張れる。child/parent 階層リンクは API 非対応（詳細は `gitlab-sub-issue-linker` を参照）
- 認証は `glab` の設定（`GITLAB_TOKEN` または `glab auth login` の保存済み認証）に依存する。トークンを本文やコマンドに埋め込まないこと
