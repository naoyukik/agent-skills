---
name: create-artifacts
description: Creates a single-file HTML artifact from user-provided content by orchestrating web-artifacts-builder (React 19 + TypeScript + Vite + Tailwind CSS v4 + shadcn/ui scaffolding and bundling) and modern-web-guidance (up-to-date web best practices). Use when asked to build a claude.ai-style artifact, a demo page, an interactive single-file HTML app, or a UI from a natural-language description.
---

# create-artifacts

ユーザーが指定した内容から、単一 HTML ファイルの artifact を作成する。ビルドパイプラインを web-artifacts-builder、実装パターンの調査を modern-web-guidance に委譲する。このスキル自身は、入力の解釈と両スキルの実行順序を統括する。

## 入力

artifact の内容は、ユーザーが `/create-artifacts <内容>` コマンド、または会話のプロンプトで渡す。渡された内容をそのまま HTML artifact として実装する。内容が曖昧で解釈が分かれる場合は、推測せず質問ツールで確認する。

## 前提

- Node.js 18 以上と pnpm が利用可能であること（web-artifacts-builder の要件）
- ネットワークアクセスが必要であること（Vite 雛形の生成、パッケージインストール、modern-web-guidance の npx 実行）
- [modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance) スキルが利用可能であること
- [web-artifacts-builder](../web-artifacts-builder) スキルが利用可能であること

## 実行手順

### 1. 実装パターンの調査（modern-web-guidance をロードする）

modern-web-guidance スキルを skill ツールでロードし、その手順に従って実装に必要なパターンを取得する。

- 対象: 実装内容に関連するカテゴリ（UI/UX、パフォーマンス、フォーム、システム API など）
- 実行: `npx -y modern-web-guidance@latest search "<クエリ>"` で検索し、`retrieve` で該当ガイドを取得する（Windows の Git Bash / mise 環境でも拡張子なしの `npx` が有効）
- 節約: 実装に必要なガイドだけを取得し、無関係なリスト全体の読み込みを避ける

### 2. プロジェクトの生成と実装（web-artifacts-builder をロードする）

web-artifacts-builder スキルを skill ツールでロードし、その手順に従って以下を実行する。

1. `init-artifact.sh` で React 19 + TypeScript + Vite + Tailwind CSS v4 + shadcn/ui プロジェクトを生成する
2. 工程 1 で取得した実装パターンとデザイン指針を反映してコンポーネントを実装する
3. `bundle-artifact.sh` で単一 HTML ファイル（`bundle.html`）にバンドルする

### 3. 成果物の提示と検証

- `bundle.html` のパスをユーザーに報告する
- 事前テストは原則行わない。ユーザーの要求、または問題発生時のみ、ブラウザ等で検証する

## 品質ガイドライン

### デザイン指針

web-artifacts-builder の指針に従い、AI slop 回避を優先する。中央配置の多用、紫色グラデーション、一律の角丸、Inter フォントを避ける。

### ブラウザサポート

modern-web-guidance の指針に従う。特に指定がなければ Baseline widely available の機能を前提とし、それ以外の機能はガイドの fallback を実装する。

### バージョン整合

雛形は最新の React 19 / Tailwind CSS v4 を使用する（Vite の最新テンプレート依存）。modern-web-guidance のガイドが前提とする機能は、この雛形でそのまま実装できる場合が多い。平準化の余地がある場合はガイドの fallback に従う。