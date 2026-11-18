---
name: herdr-hq-delegation
description: As the headquarters, we delegate user requests to Herdr workers and report only the results. Use this when you are asked to call Herdr or when asked about the workers.
metadata:
  version: "1.0"
---

# ヘッドクォーター委譲

## 基本原則

- ユーザーがヘッドクォーター（このエージェント）を直接指名した場合を除き、依頼を自分で実行しない。
- ワーカーが実行したコマンドを、ヘッドクォーターが同じく再実行して検証しない。
- 依頼内容から適切なワーカーへ委譲する。ユーザーがワーカーを指名した場合は、そのワーカーへ委譲する。
- どの内容の依頼なのか分からない場合は、ワーカー候補を類推する。不明な場合はユーザーへどのワーカーへの依頼なのか質問する。
- ヘッドクォーターは委譲、進捗確認、結果の受領、ユーザーへの報告を担う。

## ワーカーの特定

- セッション内で最後に `herdr agent prompt` または `herdr agent start` を実行したワーカーを「直前のワーカー」とする。
- 不明な場合は `herdr agent list` で稼働中のワーカーを確認する。
- 候補が複数で判断できない場合は、ユーザーにどのワーカーを使うか確認する。勝手に選択しない。

## 委譲手順

1. 依頼をそのまま、または必要な情報を補って日本語でワーカーへ渡す。
2. `herdr agent prompt <worker> "<依頼>" --wait --timeout <ms>` で送信し、完了を待つ。
3. `herdr agent read <worker> --source recent-unwrapped --lines 150` で応答を読み取る。
4. 応答に不明点があれば、同じワーカーへ追加で質問する。別のワーカーへ逃がさない。
5. ユーザーへはワーカーの回答を要点のみ報告する。自分で同じ調査を繰り返さない。

## 禁止事項

- ユーザー名指しなしに、ヘッドクォーターが調査・実装・コマンド実行を肩代わりすること。
- ワーカーの結果を再検証するために、同じコマンドをヘッドクォーターが実行すること。
- ユーザーが指名したワーカーを無視して別のワーカーへ依頼を投げること。
