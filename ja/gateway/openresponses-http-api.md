title: "OpenClaw Gatewayの設定と使用のためのOpenResponses HTTP API"
description: "OpenClaw GatewayでOpenResponses互換のPOST /v1/responsesエンドポイントを有効にして使用する方法を学びます。認証、セキュリティ、エージェント、ツール、ファイル処理の設定について説明します。"
keywords: ["openresponses api", "openclaw gateway", "http api", "エージェント統合", "api認証", "クライアントツール", "ファイルアップロード", "ストリーミング sse"]
---

  プロトコルとAPI

  
# OpenResponses API

OpenClawのGatewayは、OpenResponses互換の`POST /v1/responses`エンドポイントを提供できます。このエンドポイントは**デフォルトで無効**です。まず設定で有効にしてください。

-   `POST /v1/responses`
-   Gatewayと同じポート（WS + HTTP多重化）: `http://<gateway-host>:/v1/responses`

内部的には、リクエストは通常のGatewayエージェント実行として実行されます（`openclaw agent`と同じコードパス）。したがって、ルーティング/権限/設定はGatewayのものと一致します。

## 認証

Gatewayの認証設定を使用します。ベアラートークンを送信してください:

-   `Authorization: Bearer `

注意:

-   `gateway.auth.mode="token"`の場合、`gateway.auth.token`（または`OPENCLAW_GATEWAY_TOKEN`）を使用します。
-   `gateway.auth.mode="password"`の場合、`gateway.auth.password`（または`OPENCLAW_GATEWAY_PASSWORD`）を使用します。
-   `gateway.auth.rateLimit`が設定されており、認証失敗が多すぎる場合、エンドポイントは`Retry-After`を含む`429`を返します。

## セキュリティ境界（重要）

このエンドポイントを、Gatewayインスタンスへの**完全なオペレーターアクセス**を提供するインターフェースとして扱ってください。

-   ここでのHTTPベアラー認証は、狭いユーザー単位のスコープモデルではありません。
-   このエンドポイントに対する有効なGatewayトークン/パスワードは、所有者/オペレーターの資格情報として扱われるべきです。
-   リクエストは、信頼されたオペレーターアクションと同じコントロールプレーンエージェントパスを経由して実行されます。
-   このエンドポイントには、所有者以外/ユーザー単位のツール境界は別途存在しません。呼び出し元がここでGateway認証を通過すると、OpenClawはその呼び出し元をこのGatewayの信頼されたオペレーターとして扱います。
-   ターゲットエージェントのポリシーが機密性の高いツールを許可している場合、このエンドポイントはそれらを使用できます。
-   このエンドポイントはループバック/Tailnet/プライベートイングレスのみで使用し、直接パブリックインターネットに公開しないでください。

[セキュリティ](./security.md)と[リモートアクセス](./remote.md)を参照してください。

## エージェントの選択

カスタムヘッダーは不要です: エージェントIDをOpenResponsesの`model`フィールドにエンコードします:

-   `model: "openclaw:"` (例: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (エイリアス)

または、ヘッダーで特定のOpenClawエージェントをターゲットにします:

-   `x-openclaw-agent-id: ` (デフォルト: `main`)

高度な設定:

-   `x-openclaw-session-key: ` を指定して、セッションルーティングを完全に制御します。

## エンドポイントの有効化

`gateway.http.endpoints.responses.enabled`を`true`に設定します:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## エンドポイントの無効化

`gateway.http.endpoints.responses.enabled`を`false`に設定します:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## セッションの動作

デフォルトでは、エンドポイントは**リクエストごとにステートレス**です（呼び出しごとに新しいセッションキーが生成されます）。リクエストにOpenResponsesの`user`文字列が含まれている場合、Gatewayはそこから安定したセッションキーを導出するため、繰り返しの呼び出しでエージェントセッションを共有できます。

## リクエスト形式（サポート対象）

リクエストは、アイテムベースの入力を持つOpenResponses APIに従います。現在サポートされているもの:

-   `input`: 文字列またはアイテムオブジェクトの配列。
-   `instructions`: システムプロンプトにマージされます。
-   `tools`: クライアントツール定義（関数ツール）。
-   `tool_choice`: クライアントツールのフィルタリングまたは必須化。
-   `stream`: SSEストリーミングを有効にします。
-   `max_output_tokens`: ベストエフォートの出力制限（プロバイダー依存）。
-   `user`: 安定したセッションルーティング。

受け入れられるが**現在は無視される**もの:

-   `max_tool_calls`
-   `reasoning`
-   `metadata`
-   `store`
-   `previous_response_id`
-   `truncation`

## アイテム（入力）

### message

ロール: `system`, `developer`, `user`, `assistant`.

-   `system`と`developer`はシステムプロンプトに追加されます。
-   最新の`user`または`function_call_output`アイテムが「現在のメッセージ」になります。
-   以前のユーザー/アシスタントメッセージは、コンテキスト用の履歴として含まれます。

### function\_call\_output (ターンベースのツール)

ツールの結果をモデルに返送します:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### reasoning と item\_reference

スキーマ互換性のために受け入れられますが、プロンプト構築時には無視されます。

## ツール（クライアントサイド関数ツール）

`tools: [{ type: "function", function: { name, description?, parameters? } }]`でツールを提供します。エージェントがツールを呼び出すことを決定した場合、レスポンスは`function_call`出力アイテムを返します。その後、`function_call_output`を含むフォローアップリクエストを送信して、ターンを続行します。

## 画像 (input\_image)

base64またはURLソースをサポートします:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

許可されるMIMEタイプ（現在）: `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`。最大サイズ（現在）: 10MB。

## ファイル (input\_file)

base64またはURLソースをサポートします:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

許可されるMIMEタイプ（現在）: `text/plain`, `text/markdown`, `text/html`, `text/csv`, `application/json`, `application/pdf`。最大サイズ（現在）: 5MB。現在の動作:

-   ファイルの内容はデコードされ、**システムプロンプト**に追加されます（ユーザーメッセージではありません）。したがって、一時的なままです（セッション履歴に永続化されません）。
-   PDFはテキスト抽出のために解析されます。テキストがほとんど見つからない場合、最初の数ページが画像にラスタライズされ、モデルに渡されます。

PDF解析には、Nodeに適した`pdfjs-dist`レガシービルド（ワーカーなし）を使用します。最新のPDF.jsビルドはブラウザのワーカー/DOMグローバルを期待するため、Gatewayでは使用されません。URLフェッチのデフォルト:

-   `files.allowUrl`: `true`
-   `images.allowUrl`: `true`
-   `maxUrlParts`: `8` (リクエストごとのURLベースの`input_file` + `input_image`パーツの合計数)
-   リクエストは保護されます（DNS解決、プライベートIPブロック、リダイレクト上限、タイムアウト）。
-   入力タイプごとにオプションのホスト名許可リストがサポートされています（`files.urlAllowlist`, `images.urlAllowlist`）。
    -   完全一致ホスト: `"cdn.example.com"`
    -   ワイルドカードサブドメイン: `"*.assets.example.com"` (頂点ドメインには一致しません)

## ファイル + 画像制限（設定）

デフォルト値は`gateway.http.endpoints.responses`で調整できます:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: [
              "image/jpeg",
              "image/png",
              "image/gif",
              "image/webp",
              "image/heic",
              "image/heif",
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

省略時のデフォルト:

-   `maxBodyBytes`: 20MB
-   `maxUrlParts`: 8
-   `files.maxBytes`: 5MB
-   `files.maxChars`: 200k
-   `files.maxRedirects`: 3
-   `files.timeoutMs`: 10s
-   `files.pdf.maxPages`: 4
-   `files.pdf.maxPixels`: 4,000,000
-   `files.pdf.minTextChars`: 200
-   `images.maxBytes`: 10MB
-   `images.maxRedirects`: 3
-   `images.timeoutMs`: 10s
-   HEIC/HEIFの`input_image`ソースは受け入れられ、プロバイダーへの配信前にJPEGに正規化されます。

セキュリティ上の注意:

-   URL許可リストは、フェッチ前およびリダイレクトホップで適用されます。
-   ホスト名を許可リストに登録しても、プライベート/内部IPブロックはバイパスされません。
-   インターネットに公開されたGatewayの場合、アプリケーションレベルのガードに加えて、ネットワークエグレス制御を適用してください。[セキュリティ](./security.md)を参照してください。

## ストリーミング (SSE)

`stream: true`を設定して、Server-Sent Events (SSE) を受信します:

-   `Content-Type: text/event-stream`
-   各イベント行は`event: `と`data: `です
-   ストリームは`data: [DONE]`で終了します

現在出力されるイベントタイプ:

-   `response.created`
-   `response.in_progress`
-   `response.output_item.added`
-   `response.content_part.added`
-   `response.output_text.delta`
-   `response.output_text.done`
-   `response.content_part.done`
-   `response.output_item.done`
-   `response.completed`
-   `response.failed` (エラー時)

## 使用量

`usage`は、基盤となるプロバイダーがトークン数を報告した場合に設定されます。

## エラー

エラーは以下のようなJSONオブジェクトを使用します:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

一般的なケース:

-   `401` 認証がない/無効
-   `400` 無効なリクエストボディ
-   `405` 誤ったメソッド

## 例

非ストリーミング:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

ストリーミング:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```

[OpenAI Chat Completions](./openai-http-api.md)[Tools Invoke API](./tools-invoke-http-api.md)