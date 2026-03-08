

  プロバイダー

  
# NVIDIA

NVIDIAは、NemotronおよびNeMoモデル向けに、`https://integrate.api.nvidia.com/v1` でOpenAI互換のAPIを提供しています。[NVIDIA NGC](https://catalog.ngc.nvidia.com/)から取得したAPIキーで認証を行います。

## CLIセットアップ

キーを一度エクスポートし、その後オンボーディングを実行してNVIDIAモデルを設定します：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

もし `--token` を引き続き使用する場合は、シェルの履歴や `ps` コマンドの出力に残ることを覚えておいてください。可能な場合は環境変数の使用を推奨します。

## 設定スニペット

```json
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## モデルID

-   `nvidia/llama-3.1-nemotron-70b-instruct` (デフォルト)
-   `meta/llama-3.3-70b-instruct`
-   `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 注意点

-   OpenAI互換の `/v1` エンドポイントです。NVIDIA NGCから取得したAPIキーを使用してください。
-   `NVIDIA_API_KEY` が設定されるとプロバイダーは自動的に有効になります。静的デフォルト（131,072トークンのコンテキストウィンドウ、4,096トークンの最大出力）を使用します。

[Mistral](./mistral.md)[Ollama](./ollama.md)