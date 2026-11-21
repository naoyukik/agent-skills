---
name: herdr-hq-delegation
description: As the headquarters, we delegate user requests to Herdr workers and report only the results. Use this when you are asked to call Herdr or when asked about the workers.
metadata:
    version: "1.1"
---
# ヘッドクォーター委譲

## 基本原則

- ユーザーがヘッドクォーター（このエージェント）を直接指名した場合を除き、依頼を自分で実行しない。
- ワーカーが実行したコマンドを、ヘッドクォーターが同じく再実行して検証しない。
- 依頼内容から適切なワーカーへ委譲する。ユーザーがワーカーを指名した場合は、そのワーカーへ委譲する。
- どの内容の依頼なのか分からない場合は、ワーカー候補を類推する。不明な場合はユーザーへどのワーカーへの依頼なのか質問する。
- ヘッドクォーターは委譲、進捗確認、結果の受領、ユーザーへの報告を担う。
- 委譲は非同期で行う。依頼を送信したら完了を待たずにユーザーへ制御を戻す。

## ワーカーの特定

- セッション内で最後に `herdr agent prompt` または `herdr agent start` を実行したワーカーを「直前のワーカー」とする。
- 不明な場合は `herdr agent list` で稼働中のワーカーを確認する。
- 候補が複数で判断できない場合は、ユーザーにどのワーカーを使うか確認する。勝手に選択しない。

## 委譲手順（非同期）

1. 依頼をそのまま、または必要な情報を補って日本語でワーカーへ渡す。
2. `herdr agent prompt <worker> "<依頼>"` を `--wait` なしで実行する。送信が受理されたらすぐに戻る。
3. 委譲したワーカーごとに、バックグラウンド監視を開始する。シェルツールの `background: true`（run_in_background）を使い、次を実行する。

   ```bash
   herdr agent wait <worker> --until working --timeout 60000 && herdr agent wait <worker> --timeout 600000
   ```

   最初の待機で処理開始を確認し、二つ目の待機で応答完了を待つ。送信直後の `idle` を完了と誤判定しないため、この二段構えが必須だ。
4. バックグラウンドタスクの完了通知を受けたら、`herdr agent read <worker> --source recent-unwrapped --lines 150` で応答を読み取る。
5. ユーザーへはワーカーの回答を要点のみ報告する。完了前なら「委譲済みで応答待ち」とだけ伝える。自分で同じ調査を繰り返さない。
6. ワーカーがもともと `working` だった場合は、完了待機が前のターンを拾うことがある。`agent read` で今回の依頼への応答か確認し、別ターンなら再度待機する。

## 禁止事項

- `herdr agent prompt ... --wait` を同期的に実行して、ユーザーの制御をブロックすること。
- ユーザー名指しなしに、ヘッドクォーターが調査・実装・コマンド実行を肩代わりすること。
- ワーカーの結果を再検証するために、同じコマンドをヘッドクォーターが実行すること。
- ユーザーが指名したワーカーを無視して別のワーカーへ依頼を投げること。

## 参考資料

- OpenCode ワーカーのエージェント設定手順: `Reference/opencode-agent-setup.md`
  エージェント ID の調べ方、`session.switchAgent` による切り替え、`opencode run --agent` との使い分けを記載している。必要なときに読む。
