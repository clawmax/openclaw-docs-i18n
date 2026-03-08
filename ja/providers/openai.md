title: "OpenClaw APIキーとCodexのためのOpenAIプロバイダー設定"
description: "OpenClawでOpenAI APIキーまたはCodex OAuthを設定する方法を学びます。GPTモデルの設定、トランスポートの管理、サーバーサイド圧縮の有効化を行います。"
keywords: ["openai", "openclaw", "apiキー", "codex", "gpt-5.4", "oauth", "websocket", "responses api"]
---

  プロバイダー

  
# OpenAI

OpenAIはGPTモデルのための開発者向けAPIを提供しています。Codexは、サブスクリプションアクセスのための**ChatGPTサインイン**、または使用量ベースのアクセスのための**APIキー**サインインをサポートしています。CodexクラウドではChatGPTサインインが必要です。OpenAIは、OpenClawのような外部ツール/ワークフローでのサブスクリプションOAuthの使用を明示的にサポートしています。

## オプションA: OpenAI APIキー (OpenAI Platform)

**最適な用途:** 直接APIアクセスと使用量ベースの課金。OpenAIダッシュボードからAPIキーを取得してください。

### CLI設定

```bash
openclaw onboard --auth-choice openai-api-key
# または非対話型
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### 設定スニペット

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

OpenAIの現在のAPIモデルドキュメントでは、直接OpenAI API使用のために`gpt-5.4`と`gpt-5.4-pro`がリストされています。OpenClawは両方を`openai/*` Responsesパスを通じて転送します。

## オプションB: OpenAI Code (Codex) サブスクリプション

**最適な用途:** APIキーの代わりにChatGPT/Codexサブスクリプションアクセスを使用する場合。CodexクラウドではChatGPTサインインが必要ですが、Codex CLIはChatGPTまたはAPIキーサインインの両方をサポートしています。

### CLI設定 (Codex OAuth)

```bash
# ウィザードでCodex OAuthを実行
openclaw onboard --auth-choice openai-codex

# またはOAuthを直接実行
openclaw models auth login --provider openai-codex
```

### 設定スニペット (Codexサブスクリプション)

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

OpenAIの現在のCodexドキュメントでは、現在のCodexモデルとして`gpt-5.4`がリストされています。OpenClawは、ChatGPT/Codex OAuth使用のためにこれを`openai-codex/gpt-5.4`にマッピングします。

### トランスポートのデフォルト

OpenClawはモデルストリーミングに`pi-ai`を使用します。`openai/*`と`openai-codex/*`の両方について、デフォルトのトランスポートは`"auto"`です（WebSocket優先、その後SSEフォールバック）。`agents.defaults.models.<provider/model>.params.transport`を設定できます:

-   `"sse"`: SSEを強制
-   `"websocket"`: WebSocketを強制
-   `"auto"`: WebSocketを試し、その後SSEにフォールバック

`openai/*` (Responses API) の場合、OpenClawはWebSocketトランスポート使用時にデフォルトでWebSocketウォームアップも有効にします (`openaiWsWarmup: true`)。関連するOpenAIドキュメント:

-   [WebSocketを使ったRealtime API](https://platform.openai.com/docs/guides/realtime-websocket)
-   [ストリーミングAPIレスポンス (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### OpenAI WebSocketウォームアップ

OpenAIドキュメントではウォームアップはオプションと説明されています。OpenClawは、WebSocketトランスポート使用時の初回ターンの遅延を減らすために、`openai/*`に対してデフォルトでこれを有効にします。

### ウォームアップを無効化

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### ウォームアップを明示的に有効化

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### OpenAI優先処理

OpenAIのAPIは、`service_tier=priority`を介して優先処理を公開しています。OpenClawでは、`agents.defaults.models["openai/"].params.serviceTier`を設定して、直接`openai/*` Responsesリクエストでこのフィールドを通過させることができます。

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

サポートされる値は`auto`、`default`、`flex`、`priority`です。

### OpenAI Responsesサーバーサイド圧縮

直接OpenAI Responsesモデル (`openai/*` で `api: "openai-responses"` と `baseUrl` が `api.openai.com` の場合) の場合、OpenClawはOpenAIサーバーサイド圧縮ペイロードヒントを自動的に有効にします:

-   `store: true`を強制 (モデル互換性が`supportsStore: false`を設定しない限り)
-   `context_management: [{ type: "compaction", compact_threshold: ... }]`を注入

デフォルトでは、`compact_threshold`はモデルの`contextWindow`の`70%` (または利用できない場合は`80000`) です。

### サーバーサイド圧縮を明示的に有効化

互換性のあるResponsesモデル (例えばAzure OpenAI Responses) で`context_management`注入を強制したい場合に使用します:

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### カスタムしきい値で有効化

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### サーバーサイド圧縮を無効化

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

`responsesServerCompaction`は`context_management`注入のみを制御します。直接OpenAI Responsesモデルは、互換性が`supportsStore: false`を設定しない限り、引き続き`store: true`を強制します。

## 注意事項

-   モデル参照は常に`provider/model`を使用します ([/concepts/models](../concepts/models.md)を参照)。
-   認証の詳細と再利用ルールは[/concepts/oauth](../concepts/oauth.md)にあります。

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)

---