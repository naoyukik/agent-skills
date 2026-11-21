# OpenCode エージェント設定手順

Herdr ワーカーとして起動した OpenCode セッションで、使用するエージェントを切り替える手順。

## 前提

- OpenCode V2 のエージェント ID は `~/.config/opencode/agents/<name>.md` または `.opencode/agents/<name>.md` のファイル名、あるいは `opencode.json(c)` の `agents` キーで定義される
- ビルトインは `build`, `plan`, `general`, `explore` の4つだけだ。`review` というビルトインは存在しない
- エージェントの選択はセッション単位で保持される。`default_agent` を変更しても既存セッションのエージェントは置き換わらない

## 恒常ワーカーのエージェントとモデルを切り替える

`opencode run --agent <agent> "<prompt>"` は一回限りの実行であり、常設ワーカーには使えない。既存セッションに対して `session.switchAgent` API と `session.switchModel` API を呼び出す。

1. ワーカーに一度プロンプトを送り、セッションを作成する。OpenCode は最初の入力を受けるまでセッションを作らない。

   ```bash
   herdr agent prompt <worker> "エージェント設定の確認用です。一言だけ返答してください。" --wait --timeout 60000
   ```

2. セッション ID を特定する。

   ```bash
   opencode api session.list | jq -r '.data[:5][] | [.id, .agent, .title, (.location.directory // "")] | @tsv'
   ```

   ワーカーに送ったプロンプトのタイトル、または `location.directory` が対象の作業ディレクトリである行を選ぶ。

3. エージェントを切り替える。

   ```bash
   opencode api session.switchAgent --param sessionID=<session-id> --data '{"agent":"<agent-id>"}'
   ```

   成功時は出力なし（HTTP 204）。

4. モデルも切り替える。`session.switchAgent` はエージェント定義内の `model:` を適用しない。エージェント定義のモデルは TUI のクライアント側でのみ解決され、API 経路では無視される。そのためエージェントとモデルは必ず2回の API 呼び出しで切り替える。

   ```bash
   opencode api session.switchModel --param sessionID=<session-id> --data '{"model":{"id":"<model-id>","providerID":"<provider-id>","variant":"<variant>"}}'
   ```

   例: `{"model":{"id":"kimi-k3.1-pro","providerID":"opencode-go","variant":"high"}}`。`variant` は省略できる。

5. 結果を確認する。

   ```bash
   opencode api session.list | jq -r '.data[] | select(.id == "<session-id>") | [.id, .agent, .model.id, .model.providerID, (.model.variant // "default")] | @tsv'
   ```

   ワーカーのペイン下端のステータス行に `<agent> · <model>` が表示されるので、目視確認にも使える。

## 一回限りの実行

```bash
opencode run --agent <agent> "<prompt>"
```

例: `opencode run --agent reviewer "Review the current diff"`

## プロジェクト全体のデフォルトを変更する場合

`.opencode/opencode.jsonc` に `default_agent` を設定する。新規セッションだけに適用され、既存セッションは変わらない。

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "reviewer"
}
```

スコープが広いため、特定のワーカーだけを変更したい場合は `session.switchAgent` を使う。

## 注意

- TUI の `/agents` や `Shift+Tab` でも切り替えられるが、Herdr 経由のキー操作では画面状態に依存し、失敗しやすい
- `ctrl+c` は OpenCode プロセス自体を終了する。ワーカーを残したまま設定を変更したい場合は送らない
- この環境のレビュー用エージェント ID は `reviewer` であり、`review` ではない
