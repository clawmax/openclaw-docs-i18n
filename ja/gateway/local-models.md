title: "OpenClaw Gateway向けLM Studioを使用したローカルモデルのセットアップ"
description: "OpenClaw Gateway向けにMiniMax M2.5などのローカルAIモデルをLM Studioでセットアップおよび設定する方法を学びます。ハイブリッド構成やフォールバック構成も含みます。"
keywords: ["openclaw ローカルモデル", "lm studio セットアップ", "minimax m2.5", "openai互換ローカルプロキシ", "ゲートウェイ設定", "ローカルaiデプロイ", "ハイブリッドモデルフォールバック", "vllm litellm"]
---

  プロトコルとAPI

  
# ローカルモデル

ローカル実行は可能ですが、OpenClawは大きなコンテキストとプロンプトインジェクションに対する強力な防御を想定しています。小さなカードはコンテキストを切り詰め、安全性を損ないます。高い目標を設定しましょう: **Mac Studioを最大構成で2台以上、または同等のGPU環境（約300万円以上）**。単一の**24 GB** GPUは、レイテンシが高くても良い軽いプロンプトでのみ機能します。**実行可能な最大/フルサイズのモデルバリアント**を使用してください。積極的に量子化されたモデルや「小型」チェックポイントはプロンプトインジェクションのリスクを高めます（[セキュリティ](./security.md)を参照）。

## 推奨: LM Studio + MiniMax M2.5 (Responses API, フルサイズ)

現在最良のローカルスタックです。LM StudioでMiniMax M2.5をロードし、ローカルサーバー（デフォルト `http://127.0.0.1:1234`）を有効にして、Responses APIを使用して推論と最終テキストを分離します。

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**セットアップチェックリスト**

-   LM Studioをインストール: [https://lmstudio.ai](https://lmstudio.ai)
-   LM Studioで、**利用可能な最大のMiniMax M2.5ビルド**（「小型」/過度に量子化されたバリアントは避ける）をダウンロードし、サーバーを起動、`http://127.0.0.1:1234/v1/models` にリストされていることを確認します。
-   モデルをロードしたままにします。コールドロードは起動時のレイテンシを増加させます。
-   使用するLM Studioビルドが異なる場合は、`contextWindow`/`maxTokens`を調整してください。
-   WhatsAppの場合は、最終テキストのみが送信されるようにResponses APIに固執してください。

ローカル実行時もホストされたモデルを設定しておきます。`models.mode: "merge"`を使用して、フォールバックが利用可能な状態を維持します。

### ハイブリッド構成: ホスト型プライマリ、ローカルフォールバック

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### ローカル優先、ホスト型セーフティネット

プライマリとフォールバックの順序を入れ替えます。同じプロバイダーブロックと`models.mode: "merge"`を維持し、ローカルマシンがダウンしたときにSonnetやOpusにフォールバックできるようにします。

### リージョナルホスティング / データルーティング

-   ホスト型のMiniMax/Kimi/GLMバリアントは、OpenRouter上にもリージョン固定エンドポイント（例: USホスト）で存在します。そこでリージョナルバリアントを選択することで、データフローを制御しつつ、Anthropic/OpenAIへのフォールバックのために`models.mode: "merge"`を使用できます。
-   ローカルのみが最も強力なプライバシー経路です。ホスト型リージョナルルーティングは、プロバイダーの機能が必要だがデータフローを制御したい場合の中間的な選択肢です。

## その他のOpenAI互換ローカルプロキシ

vLLM、LiteLLM、OAI-proxy、またはカスタムゲートウェイは、OpenAIスタイルの`/v1`エンドポイントを公開していれば動作します。上記のプロバイダーブロックをあなたのエンドポイントとモデルIDに置き換えてください:

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

ホストされたモデルがフォールバックとして利用可能な状態を維持するために、`models.mode: "merge"`を保持します。

## トラブルシューティング

-   ゲートウェイはプロキシに到達できますか？ `curl http://127.0.0.1:1234/v1/models`。
-   LM Studioのモデルがアンロードされていませんか？ 再ロードしてください。コールドスタートは「ハング」の一般的な原因です。
-   コンテキストエラーですか？ `contextWindow`を下げるか、サーバーの制限を上げてください。
-   セキュリティ: ローカルモデルはプロバイダー側のフィルターをスキップします。エージェントの範囲を狭くし、コンパクションを有効にして、プロンプトインジェクションの影響範囲を制限してください。

[CLIバックエンド](./cli-backends.md)[ネットワークモデル](./network-model.md)