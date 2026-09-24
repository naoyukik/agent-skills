# web-artifacts-builder

このスキルは、Anthropic が公開する [anthropics/skills](https://github.com/anthropics/skills) リポジトリ内の **web-artifacts-builder** を変更した物である。

- 元リポジトリ: <https://github.com/anthropics/skills/tree/main/skills/web-artifacts-builder>

## 概要

React 19 + TypeScript + Vite + Tailwind CSS v4 + shadcn/ui を用いて、複雑な claude.ai 向け HTML artifact を構築するためのスキルである。利用手順は [SKILL.md](./SKILL.md) を参照すること。

## 元との差分

元の anthropics/skills 版は React 18 + Parcel + Tailwind CSS 3 を前提としていた。本スキルはスタックを React 19 + Vite + Tailwind CSS v4 に更新し、単一 HTML ファイルへのバンドルを Parcel から Vite へ移行した。

主な変更点は以下の通り。

### スタック

- React 18 → React 19
- バンドラ: Parcel → Vite（`vite build --base ./` + html-inline）
- Tailwind CSS 3.4.1（tailwind.config.js 必須）→ Tailwind CSS v4（CSS-first、設定ファイル不要）
- shadcn/ui のアニメーション依存を `tailwindcss-animate` → `tw-animate-css` に更新

### scripts/init-artifact.sh

- Tailwind v4 対応の `index.css` を生成（`@import "tailwindcss"`、`@custom-variant dark`、`@theme inline`）
- `tailwind.config.js` と `postcss.config.js` を v4 形式に更新
- `tsconfig.json` から TypeScript 6 で非推奨となった `baseUrl` を削除し、`paths` のみでパスエイリアスを設定
- `vite.config.ts` に `base: "./"` を追加し、`__dirname` を `import.meta.dirname`（ESM）に変更
- 単一ファイルバンドルで解決できない favicon の `<link rel="icon"...>` を全削除するよう修正
- `components.json` から `tailwind.config` 参照を削除
- 誤った Usage エラー文言（`create-react-shadcn-complete.sh`）を `init-artifact.sh` に修正

### scripts/bundle-artifact.sh

- Parcel（`.parcelrc`、`parcel build`）を削除し、`pnpm exec vite build --base ./` に置き換え
- ビルド前に `dist` と `bundle.html` を削除するクリーンアップ処理を追加
- `du` が無い環境向けに `wc -c` によるファイルサイズ取得のフォールバックを追加

### SKILL.md

- frontmatter に `metadata.version: "1.0"` と `license` を追加
- Stack 表記と手順説明を Vite / Tailwind v4 前提に更新
- 「Common Development Tasks」（テーマ切り替え、ダークモード配色、shadcn/ui 追加方法）を追加

## ライセンス

Apache License 2.0 に基づく。元の著作権表示およびライセンス全文は [LICENSE.txt](./LICENSE.txt) に記載する。