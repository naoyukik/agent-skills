---
name: operating-git
description: Be sure to refer to this when using Git commands. Detailed instructions on executing commands and troubleshooting Git workflows.
metadata:
  version: "1.1"
---

# Operating Git – Procedure & Examples

本スキルは、Git 操作の具体的な手順と、ミスを防ぐための確認フローを提供する。

## 1. 推奨されるステージング手順

意図しないファイルの混入を防ぐため、以下の手順を習慣化すること。

```bash
# 1. 変更内容の確認
git status
git diff

# 2. ファイルを個別にステージング
git add path/to/file1 path/to/file2

# 3. ステージングされた内容の最終監査
git diff --staged
```

## 2. コミットの作成

### フォーマット

- **1行目: タイトル**: `<type>: ユーザー指定の言語で説明（50文字以内）`
- **空行**
- **説明文(optional)**: 自明な説明はせずに、なぜその変更が必要なのか、もしくは何を達成するための実装なのかを完結に記述すること。箇条書きで記載すること
- **空行**
- **参照**: `ref: IssueNumber` を記述すること。IssueNumberはGitブランチの `^[0-9]+-` にマッチする数字のこと

e.g. ブランチ名: 188-implement-unit-tests

```text
test: WordParser のユニットテストを実装

- 単語パースロジックの正当性を自動検証するために実装した 

ref: 188
```

## 3. トラブルシューティング

### 誤って `git add .` してしまった場合
直ちに以下のコマンドでステージングを解除せよ。
```bash
git reset
```

### コミット後にミスに気づいた場合
プッシュ前であれば、修正後に `amend` を検討せよ。
```bash
git add <forgotten_file>
git commit --amend --no-edit
```
