

  実験

  
# OpenResponses Gateway 計画

## 背景

OpenClaw Gateway は現在、`/v1/chat/completions` に最小限の OpenAI 互換 Chat Completions エンドポイントを公開しています（[OpenAI Chat Completions](../../gateway/openai-http-api.md) を参照）。Open Responses は、OpenAI Responses API に基づいたオープンな推論標準です。これはエージェントワークフロー向けに設計されており、アイテムベースの入力とセマンティックストリーミングイベントを使用します。OpenResponses 仕様は `/v1/chat/completions` ではなく `/v1/responses` を定義しています。

## 目標

-   OpenResponses のセマンティクスに準拠した `/v1/responses` エンドポイントを追加する。
-   Chat Completions は、無効化および最終的に削除が容易な互換性レイヤーとして維持する。
-   分離された再利用可能なスキーマで、検証とパースを標準化する。

## 非目標

-   最初の実装での完全な OpenResponses 機能パリティ（画像、ファイル、ホストされたツール）。
-   内部のエージェント実行ロジックやツールオーケストレーションの置き換え。
-   最初のフェーズでの既存の `/v1/chat/completions` の動作の変更。

## 調査概要

情報源: OpenResponses OpenAPI、OpenResponses 仕様サイト、Hugging Face ブログ記事。抽出された主なポイント:

-   `POST /v1/responses` は、`model`、`input`（文字列または `ItemParam[]`）、`instructions`、`tools`、`tool_choice`、`stream`、`max_output_tokens`、`max_tool_calls` などの `CreateResponseBody` フィールドを受け入れる。
-   `ItemParam` は以下の判別共用体:
    -   ロールが `system`、`developer`、`user`、`assistant` の `message` アイテム
    -   `function_call` および `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   成功したレスポンスは、`object: "response"`、`status`、`output` アイテムを持つ `ResponseResource` を返す。
-   ストリーミングは以下のようなセマンティックイベントを使用する:
    -   `response.created`、`response.in_progress`、`response.completed`、`response.failed`
    -   `response.output_item.added`、`response.output_item.done`
    -   `response.content_part.added`、`response.content_part.done`
    -   `response.output_text.delta`、`response.output_text.done`
-   仕様では以下が必要:
    -   `Content-Type: text/event-stream`
    -   `event:` は JSON の `type` フィールドと一致しなければならない
    -   終端イベントはリテラル `[DONE]` でなければならない
-   Reasoning アイテムは `content`、`encrypted_content`、`summary` を公開する場合がある。
-   HF の例では、リクエストに `OpenResponses-Version: latest` を含む（オプションヘッダー）。

## 提案アーキテクチャ

-   Zod スキーマのみを含む `src/gateway/open-responses.schema.ts` を追加する（gateway インポートなし）。
-   `/v1/responses` 用に `src/gateway/openresponses-http.ts`（または `open-responses-http.ts`）を追加する。
-   `src/gateway/openai-http.ts` はレガシー互換アダプターとしてそのまま維持する。
-   設定 `gateway.http.endpoints.responses.enabled`（デフォルト `false`）を追加する。
-   `gateway.http.endpoints.chatCompletions.enabled` は独立して維持し、両エンドポイントを個別に切り替え可能にする。
-   Chat Completions が有効な場合、レガシー状態を示す起動時の警告を出力する。

## Chat Completions の非推奨化パス

-   厳密なモジュール境界を維持する: responses と chat completions の間でスキーマ型を共有しない。
-   Chat Completions を設定でオプトイン可能にし、コード変更なしで無効化できるようにする。
-   `/v1/responses` が安定したら、Chat Completions をレガシーとして文書に明記する。
-   オプションの将来のステップ: Chat Completions リクエストを Responses ハンドラーにマッピングし、削除パスを簡素化する。

## フェーズ 1 サポートサブセット

-   `input` を文字列または `ItemParam[]`（メッセージロールと `function_call_output` を含む）として受け入れる。
-   システムメッセージと開発者メッセージを `extraSystemPrompt` に抽出する。
-   最新の `user` または `function_call_output` をエージェント実行用の現在のメッセージとして使用する。
-   サポートされていないコンテンツパーツ（画像/ファイル）は `invalid_request_error` で拒否する。
-   単一のアシスタントメッセージを `output_text` コンテンツで返す。
-   トークン計算が接続されるまで、ゼロ値の `usage` を返す。

## 検証戦略（SDK なし）

-   以下のサポート対象サブセットの Zod スキーマを実装する:
    -   `CreateResponseBody`
    -   `ItemParam` + メッセージコンテンツパーツの共用体
    -   `ResponseResource`
    -   ゲートウェイで使用されるストリーミングイベントの形状
-   スキーマを単一の分離されたモジュールに保持し、ずれを防ぎ、将来のコード生成を可能にする。

## ストリーミング実装（フェーズ 1）

-   `event:` と `data:` の両方を持つ SSE 行。
-   必要なシーケンス（最小限の実現可能なもの）:
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta`（必要に応じて繰り返し）
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## テストと検証計画

-   `/v1/responses` に対する e2e カバレッジを追加する:
    -   認証必須
    -   非ストリームレスポンスの形状
    -   ストリームイベントの順序と `[DONE]`
    -   ヘッダーと `user` によるセッションルーティング
-   `src/gateway/openai-http.test.ts` は変更しない。
-   手動: `stream: true` を指定して `/v1/responses` に curl を実行し、イベント順序と終端の `[DONE]` を確認する。

## ドキュメント更新（フォローアップ）

-   `/v1/responses` の使用方法と例に関する新しいドキュメントページを追加する。
-   `/gateway/openai-http-api` を更新し、レガシー注記と `/v1/responses` へのポインタを追加する。

[Browser Evaluate CDP リファクタリング](./browser-evaluate-cdp-refactor.md)[PTY とプロセス監視計画](./pty-process-supervision.md)