

  RPCとAPI

  
# RPCアダプター

OpenClawはJSON-RPCを介して外部CLIを統合します。現在、2つのパターンが使用されています。

## パターンA: HTTPデーモン (signal-cli)

-   `signal-cli`は、HTTP経由のJSON-RPCを備えたデーモンとして実行されます。
-   イベントストリームはSSE (`/api/v1/events`) です。
-   ヘルスプローブ: `/api/v1/check`。
-   `channels.signal.autoStart=true`の場合、OpenClawがライフサイクルを管理します。

セットアップとエンドポイントについては、[Signal](../channels/signal.md)を参照してください。

## パターンB: stdio子プロセス (レガシー: imsg)

> **注:** 新しいiMessageセットアップでは、代わりに[BlueBubbles](../channels/bluebubbles.md)を使用してください。

-   OpenClawは子プロセスとして`imsg rpc`を生成します（レガシーiMessage統合）。
-   JSON-RPCはstdin/stdoutを介した行区切り形式です（1行に1つのJSONオブジェクト）。
-   TCPポートやデーモンは不要です。

使用されるコアメソッド:

-   `watch.subscribe` → 通知 (`method: "message"`)
-   `watch.unsubscribe`
-   `send`
-   `chats.list` (プローブ/診断)

レガシーセットアップとアドレッシング（`chat_id`が推奨）については、[iMessage](../channels/imessage.md)を参照してください。

## アダプターのガイドライン

-   ゲートウェイがプロセスを所有します（起動/停止はプロバイダーのライフサイクルに紐づけられます）。
-   RPCクライアントは耐障害性を持たせます：タイムアウト、終了時の再起動。
-   表示文字列よりも安定したID（例: `chat_id`）を優先します。

[webhooks](../cli/webhooks.md)[デバイスモデルデータベース](./device-models.md)

---