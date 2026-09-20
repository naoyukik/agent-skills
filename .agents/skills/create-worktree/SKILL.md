---
name: create-worktree
description: Creates an isolated git worktree under `.worktree/<branch>` at the project root, copies the files or directories listed in `.worktreeinclude` into the new worktree, and resolves the branch name from user input, an issue (gh/glab), or a clarifying question. Use when asked to create a worktree, add a sibling checkout, or start work in an isolated branch.
---

# create-worktree

プロジェクトルート直下の `.worktree/` に git worktree を作成する。worktree のフォルダ名はブランチ名をそのまま使用する。`.worktreeinclude` が存在する場合は、そこに記載されたファイルまたはフォルダを bash シェルで新 worktree へコピーする。作成完了後はディレクトリのパスをユーザーに報告する。

## 1. ブランチ名の決定

以下の優先順位で決定する。

1. **ユーザー指定**: ユーザーがブランチ名を渡している場合はそれを採用する。
2. **自動生成**: 指定がない場合、Issue やタスク情報からブランチ名を生成できるなら生成する。
   - リモートのホストと利用可能な CLI を確認する。GitHub なら `gh`、GitLab なら `glab` を使用する。
   - オープンな Issue を取得する。
     - GitHub: `gh issue list --state open --limit 10 --json number,title`
     - GitLab: `glab issue list --opened --per-page 10`
   - タスクの文脈に即した Issue を選び、`<番号>-<スラグ>` 形式でブランチ名を生成する。スラグはタイトルを小文字英数字とハイフンのみに変換したものとする。日本語タイトルの場合は英訳して短いスラグにする。リポジトリの既存慣習に prefix（`feat/` 等）があるなら従う。
   - Issue が絞り込めない場合や生成ルールが適用できない場合は手順3へ移る。
3. **ユーザーへの質問**: 上記どちらも不可能なら、質問ツールでブランチ名を確認する。

生成したブランチ名は必ず git のブランチ名として妥当か検証する。

```bash
git check-ref-format "refs/heads/$BRANCH" || echo "invalid branch name: $BRANCH"
```

検証に失敗した場合はユーザーに質問してブランチ名を確定する。

## 2. 開始地点（ベースブランチ）の決定

`main` ブランチが存在すればそこを開始地点とする。存在しない場合は現在のブランチ（`git branch --show-current`）を使う。ユーザー指定があればそれに従う。

```bash
if git show-ref --verify --quiet "refs/heads/main"; then
  BASE="main"
else
  BASE="$(git branch --show-current)"
fi
```

## 3. 既存ブランチの確認

同名ブランチが既に存在する場合は再利用し、新規作成は行わない。

```bash
git branch --list "$BRANCH"
```

## 4. リポジトリルートの取得と `.gitignore` の準備

リポジトリルートで作業する。`.worktree/` は git の管理対象外にするため、`.gitignore` に無ければ追記する。

```bash
ROOT="$(git rev-parse --show-toplevel)"
cd "$ROOT"

if [ -f .gitignore ] && ! grep -qxF '.worktree/' .gitignore; then
  printf '\n# git worktrees\n.worktree/\n' >> .gitignore
fi
```

## 5. worktree の作成

新規ブランチの場合は `-b` を付けて作成し、既存ブランチの場合は `-b` を付けずにチェックアウトする。

```bash
# 新規ブランチ
git worktree add -b "$BRANCH" "$ROOT/.worktree/$BRANCH" "$BASE"

# 既存ブランチの再利用
# git worktree add "$ROOT/.worktree/$BRANCH" "$BRANCH"
```

注意:

- パスは絶対パスで指定する。
- ブランチ名に `/` が含まれる場合（`feat/9-foo` 等）は、`.worktree/` 配下でフォルダとしてネストされる。フォルダ名はブランチ名そのままという要件を満たす。
- 既存ブランチを再利用する場合は、その旨をユーザーに伝える。

## 6. `.worktreeinclude` のファイルコピー

`.worktreeinclude` が存在する場合、各エントリ（ファイルまたはフォルダ）をリポジトリルートからの相対パスとして新 worktree へコピーする。bash で実装する。

```bash
if [ -f "$ROOT/.worktreeinclude" ]; then
  TARGET="$ROOT/.worktree/$BRANCH"
  while IFS= read -r entry || [ -n "$entry" ]; do
    case "$entry" in
      '' | \#*) continue ;;
    esac
    if [ -e "$ROOT/$entry" ]; then
      PARENT="$(dirname "$entry")"
      mkdir -p "$TARGET/$PARENT"
      cp -r "$ROOT/$entry" "$TARGET/$PARENT"
    else
      echo "warning: $entry does not exist, skipped"
    fi
  done < "$ROOT/.worktreeinclude"
fi
```

`.worktreeinclude` の書式は 1 行に 1 エントリである。空行と `#` で始まる行は無視する。

```text
AGENTS.md
.agents/skills
conductor/
```

## 7. 検証と報告

worktree の作成結果を確認する。

```bash
git worktree list
```

作成完了後、以下をユーザーに報告する。

- worktree のディレクトリパス: `$ROOT/.worktree/$BRANCH`
- ブランチ名と開始地点
- `.worktreeinclude` からコピーしたファイルまたはフォルダの一覧（コピー対象があった場合）
- スキップされたエントリ（存在しなかった場合）