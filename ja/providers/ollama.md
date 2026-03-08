

  プロバイダー

  
# Ollama

Ollamaは、オープンソースモデルをマシン上で簡単に実行できるローカルLLMランタイムです。OpenClawはOllamaのネイティブAPI (`/api/chat`) と統合し、ストリーミングとツール呼び出しをサポートします。また、`OLLAMA_API_KEY`（または認証プロファイル）を設定し、明示的な `models.providers.ollama` エントリを定義**しない**場合、**ツール対応モデルを自動検出**できます。

> **⚠️** **リモートOllamaユーザー**: OpenClawでOpenAI互換URL (`http://host:11434/v1`) を使用**しないでください**。これによりツール呼び出しが壊れ、モデルが生のツールJSONをプレーンテキストとして出力する可能性があります。代わりにネイティブOllama API URLを使用してください: `baseUrl: "http://host:11434"` (`/v1`なし)。

## クイックスタート

1.  Ollamaをインストール: [https://ollama.ai](https://ollama.ai)
2.  モデルをプル:

```bash
ollama pull gpt-oss:20b
# または
ollama pull llama3.3
# または
ollama pull qwen2.5-coder:32b
# または
ollama pull deepseek-r1:32b
```

3.  OpenClawでOllamaを有効化（任意の値で動作します。Ollamaは実際のキーを必要としません）:

```bash
# 環境変数を設定
export OLLAMA_API_KEY="ollama-local"

# または設定ファイルで構成
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  Ollamaモデルを使用:

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## モデル検出（暗黙的プロバイダー）

`OLLAMA_API_KEY`（または認証プロファイル）を設定し、**かつ** `models.providers.ollama` を定義**しない**場合、OpenClawは `http://127.0.0.1:11434` のローカルOllamaインスタンスからモデルを検出します:

-   `/api/tags` と `/api/show` をクエリ
-   `tools` 機能を報告するモデルのみを保持
-   モデルが `thinking` を報告する場合、`reasoning` をマーク
-   利用可能な場合、`model_info[".context_length"]` から `contextWindow` を読み取り
-   `maxTokens` をコンテキストウィンドウの10倍に設定
-   すべてのコストを `0` に設定

これにより、手動でのモデルエントリを避けつつ、カタログをOllamaの機能に合わせて維持できます。利用可能なモデルを確認するには:

```bash
ollama list
openclaw models list
```

新しいモデルを追加するには、Ollamaでプルするだけです:

```bash
ollama pull mistral
```

新しいモデルは自動的に検出され、使用可能になります。明示的に `models.providers.ollama` を設定すると、自動検出はスキップされ、モデルを手動で定義する必要があります（下記参照）。

## 設定

### 基本セットアップ（暗黙的検出）

Ollamaを有効にする最も簡単な方法は環境変数を使用することです:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 明示的セットアップ（手動モデル）

以下の場合に明示的設定を使用します:

-   Ollamaが別のホスト/ポートで実行されている。
-   特定のコンテキストウィンドウやモデルリストを強制したい。
-   ツールサポートを報告しないモデルを含めたい。

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY` が設定されている場合、プロバイダーエントリの `apiKey` を省略でき、OpenClawは可用性チェックのためにそれを埋めます。

### カスタムベースURL（明示的設定）

Ollamaが別のホストやポートで実行されている場合（明示的設定は自動検出を無効にするため、モデルを手動で定義してください）:

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // /v1 なし - ネイティブOllama API URLを使用
        api: "ollama", // ネイティブのツール呼び出し動作を保証するために明示的に設定
      },
    },
  },
}
```

> **⚠️** URLに `/v1` を追加しないでください。`/v1` パスはOpenAI互換モードを使用し、ツール呼び出しが信頼できません。パスサフィックスなしのベースOllama URLを使用してください。

### モデル選択

設定が完了すると、すべてのOllamaモデルが利用可能になります:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 高度な設定

### 推論モデル

OpenClawは、Ollamaが `/api/show` で `thinking` を報告する場合、モデルを推論対応としてマークします:

```bash
ollama pull deepseek-r1:32b
```

### モデルコスト

Ollamaは無料でローカルで実行されるため、すべてのモデルコストは $0 に設定されます。

### ストリーミング設定

OpenClawのOllama統合は、デフォルトで**ネイティブOllama API** (`/api/chat`) を使用し、ストリーミングとツール呼び出しの同時実行を完全にサポートします。特別な設定は必要ありません。

#### レガシーOpenAI互換モード

> **⚠️** **OpenAI互換モードではツール呼び出しは信頼できません。** このモードは、プロキシのためにOpenAI形式が必要で、ネイティブのツール呼び出し動作に依存しない場合にのみ使用してください。

 OpenAI互換エンドポイントを代わりに使用する必要がある場合（例: OpenAI形式のみをサポートするプロキシの背後）、明示的に `api: "openai-completions"` を設定します:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // デフォルト: true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

このモードでは、ストリーミングとツール呼び出しの同時実行をサポートしない場合があります。モデル設定で `params: { streaming: false }` を使用してストリーミングを無効にする必要があるかもしれません。`api: "openai-completions"` がOllamaと共に使用される場合、OpenClawはデフォルトで `options.num_ctx` を注入し、Ollamaが4096コンテキストウィンドウに暗黙的にフォールバックしないようにします。プロキシ/アップストリームが未知の `options` フィールドを拒否する場合、この動作を無効にします:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### コンテキストウィンドウ

自動検出されたモデルの場合、OpenClawは利用可能な場合にOllamaが報告するコンテキストウィンドウを使用し、それ以外の場合はデフォルトで `8192` を使用します。明示的プロバイダー設定で `contextWindow` と `maxTokens` をオーバーライドできます。

## トラブルシューティング

### Ollamaが検出されない

Ollamaが実行中であり、`OLLAMA_API_KEY`（または認証プロファイル）を設定し、かつ明示的な `models.providers.ollama` エントリを定義**していない**ことを確認してください:

```bash
ollama serve
```

また、APIにアクセスできることを確認してください:

```bash
curl http://localhost:11434/api/tags
```

### 利用可能なモデルがない

OpenClawはツールサポートを報告するモデルのみを自動検出します。モデルがリストにない場合は、以下のいずれかを行ってください:

-   ツール対応モデルをプルする、または
-   モデルを `models.providers.ollama` で明示的に定義する。

モデルを追加するには:

```bash
ollama list  # インストール済みを確認
ollama pull gpt-oss:20b  # ツール対応モデルをプル
ollama pull llama3.3     # または別のモデル
```

### 接続拒否

Ollamaが正しいポートで実行されているか確認してください:

```bash
# Ollamaが実行中か確認
ps aux | grep ollama

# またはOllamaを再起動
ollama serve
```

## 関連項目

-   [モデルプロバイダー](../concepts/model-providers.md) - すべてのプロバイダーの概要
-   [モデル選択](../concepts/models.md) - モデルの選び方
-   [設定](../gateway/configuration.md) - 完全な設定リファレンス

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)