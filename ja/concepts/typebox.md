

  コンセプト内部

  
# TypeBox

最終更新日: 2026-01-10 TypeBoxはTypeScriptファーストのスキーマライブラリです。私たちはこれを使用して**Gateway WebSocketプロトコル**（ハンドシェイク、リクエスト/レスポンス、サーバーイベント）を定義します。これらのスキーマは、macOSアプリのための**ランタイム検証**、**JSONスキーマエクスポート**、**Swiftコード生成**を駆動します。単一の情報源であり、それ以外はすべて生成されます。より高レベルのプロトコルコンテキストが必要な場合は、[Gatewayアーキテクチャ](./architecture.md)から始めてください。

## メンタルモデル (30秒)

すべてのGateway WSメッセージは、3つのフレームのいずれかです:

-   **リクエスト**: `{ type: "req", id, method, params }`
-   **レスポンス**: `{ type: "res", id, ok, payload | error }`
-   **イベント**: `{ type: "event", event, payload, seq?, stateVersion? }`

最初のフレームは**必ず** `connect` リクエストでなければなりません。その後、クライアントはメソッド（例: `health`, `send`, `chat.send`）を呼び出したり、イベント（例: `presence`, `tick`, `agent`）を購読したりできます。接続フロー（最小限）:

```
クライアント                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

一般的なメソッド + イベント:

| カテゴリー | 例 | 備考 |
| --- | --- | --- |
| コア | `connect`, `health`, `status` | `connect` が最初である必要あり |
| メッセージング | `send`, `poll`, `agent`, `agent.wait` | 副作用のある操作には `idempotencyKey` が必要 |
| チャット | `chat.history`, `chat.send`, `chat.abort`, `chat.inject` | WebChatで使用 |
| セッション | `sessions.list`, `sessions.patch`, `sessions.delete` | セッション管理 |
| ノード | `node.list`, `node.invoke`, `node.pair.*` | Gateway WS + ノードアクション |
| イベント | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | サーバープッシュ |

信頼できるリストは `src/gateway/server.ts` (`METHODS`, `EVENTS`) にあります。

## スキーマの場所

-   ソース: `src/gateway/protocol/schema.ts`
-   ランタイムバリデータ (AJV): `src/gateway/protocol/index.ts`
-   サーバーハンドシェイク + メソッドディスパッチ: `src/gateway/server.ts`
-   ノードクライアント: `src/gateway/client.ts`
-   生成されたJSONスキーマ: `dist/protocol.schema.json`
-   生成されたSwiftモデル: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## 現在のパイプライン

-   `pnpm protocol:gen`
    -   JSONスキーマ (draft‑07) を `dist/protocol.schema.json` に書き出します
-   `pnpm protocol:gen:swift`
    -   Swiftゲートウェイモデルを生成します
-   `pnpm protocol:check`
    -   両方のジェネレーターを実行し、出力がコミットされていることを確認します

## スキーマがランタイムでどのように使用されるか

-   **サーバー側**: すべての受信フレームはAJVで検証されます。ハンドシェイクは、パラメータが `ConnectParams` に一致する `connect` リクエストのみを受け入れます。
-   **クライアント側**: JSクライアントは、イベントおよびレスポンスフレームを使用する前に検証します。
-   **メソッドサーフェス**: Gatewayは、サポートされる `methods` と `events` を `hello-ok` で通知します。

## フレームの例

接続 (最初のメッセージ):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Hello-ok レスポンス:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

リクエスト + レスポンス:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

イベント:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## 最小限のクライアント (Node.js)

最小限の有用なフロー: 接続 + ヘルスチェック。

```typescript
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## 実践例: メソッドをエンドツーエンドで追加する

例: `{ ok: true, text }` を返す新しい `system.echo` リクエストを追加します。

1.  **スキーマ (信頼できる情報源)**

`src/gateway/protocol/schema.ts` に追加:

```bash
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

両方を `ProtocolSchemas` に追加し、型をエクスポート:

```yaml
SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```bash
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **検証**

`src/gateway/protocol/index.ts` で、AJVバリデータをエクスポート:

```bash
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **サーバーの動作**

`src/gateway/server-methods/system.ts` にハンドラーを追加:

```bash
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

`src/gateway/server-methods.ts` で登録し（すでに `systemHandlers` をマージ）、次に `src/gateway/server.ts` の `METHODS` に `"system.echo"` を追加します。

4.  **再生成**

```bash
pnpm protocol:check
```

5.  **テスト + ドキュメント**

`src/gateway/server.*.test.ts` にサーバーテストを追加し、ドキュメントにメソッドを記載します。

## Swiftコード生成の動作

Swiftジェネレーターは以下を出力します:

-   `req`, `res`, `event`, `unknown` ケースを持つ `GatewayFrame` enum
-   厳密に型付けされたペイロード構造体/enum
-   `ErrorCode` 値と `GATEWAY_PROTOCOL_VERSION`

未知のフレームタイプは、前方互換性のために生のペイロードとして保持されます。

## バージョニング + 互換性

-   `PROTOCOL_VERSION` は `src/gateway/protocol/schema.ts` にあります。
-   クライアントは `minProtocol` + `maxProtocol` を送信します。サーバーは不一致を拒否します。
-   Swiftモデルは、古いクライアントとの互換性を保つために未知のフレームタイプを保持します。

## スキーマパターンと規約

-   ほとんどのオブジェクトは、厳密なペイロードのために `additionalProperties: false` を使用します。
-   `NonEmptyString` は、IDやメソッド/イベント名のデフォルトです。
-   トップレベルの `GatewayFrame` は、`type` に基づく**判別子**を使用します。
-   副作用のあるメソッドは、通常、パラメータに `idempotencyKey` を必要とします（例: `send`, `poll`, `agent`, `chat.send`）。
-   `agent` は、ランタイム生成のオーケストレーションコンテキスト（例: サブエージェント/クロンタスク完了の引き継ぎ）のためにオプションの `internalEvents` を受け入れます。これを内部APIサーフェスとして扱ってください。

## ライブスキーマJSON

生成されたJSONスキーマは、リポジトリの `dist/protocol.schema.json` にあります。公開された生ファイルは通常、以下で利用可能です:

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## スキーマを変更するとき

1.  TypeBoxスキーマを更新します。
2.  `pnpm protocol:check` を実行します。
3.  再生成されたスキーマ + Swiftモデルをコミットします。

[日付と時刻](../date-time.md)[Markdownフォーマット](./markdown-formatting.md)