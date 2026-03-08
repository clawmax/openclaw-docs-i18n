title: "OpenClaw Hugging Face プロバイダー設定と構成ガイド"
description: "OpenClawでHugging Face Inferenceプロバイダーを設定・構成する方法を学びます。APIキーの接続、DeepSeekやLlamaなどのモデルの選択、ルーティングポリシーのカスタマイズを行います。"
keywords: ["hugging face inference", "openclaw プロバイダー", "deepseek", "llama", "openai互換api", "モデル構成", "ルーターapi", "チャット補完"]
---

  プロバイダー

  
# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) は、単一のルーターAPIを通じてOpenAI互換のチャット補完を提供します。1つのトークンで多くのモデル（DeepSeek、Llamaなど）にアクセスできます。OpenClawは **OpenAI互換エンドポイント**（チャット補完のみ）を使用します。テキストから画像、埋め込み、音声の場合は、[HF inference clients](https://huggingface.co/docs/api-inference/quicktour) を直接使用してください。

-   プロバイダー: `huggingface`
-   認証: `HUGGINGFACE_HUB_TOKEN` または `HF_TOKEN` (**Make calls to Inference Providers** 権限を持つファイングレーンドトークン)
-   API: OpenAI互換 (`https://router.huggingface.co/v1`)
-   課金: 単一のHFトークン; [料金](https://huggingface.co/docs/inference-providers/pricing)はプロバイダーのレートに従い、無料枠があります。

## クイックスタート

1.  [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) で、**Make calls to Inference Providers** 権限を持つファイングレーンドトークンを作成します。
2.  オンボーディングを実行し、プロバイダードロップダウンで **Hugging Face** を選択し、プロンプトが表示されたらAPIキーを入力します:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  **Default Hugging Face model** ドロップダウンで、使用したいモデルを選択します（このリストは、有効なトークンがある場合にInference APIから読み込まれます。それ以外の場合は組み込みリストが表示されます）。選択内容はデフォルトモデルとして保存されます。
4.  後で設定でデフォルトモデルを設定または変更することもできます:

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

これにより、`huggingface/deepseek-ai/DeepSeek-R1` がデフォルトモデルとして設定されます。

## 環境に関する注意

Gatewayがデーモン（launchd/systemd）として実行される場合、`HUGGINGFACE_HUB_TOKEN` または `HF_TOKEN` がそのプロセスで利用可能であることを確認してください（例: `~/.openclaw/.env` または `env.shellEnv` 経由）。

## モデル検出とオンボーディングドロップダウン

OpenClawは、**Inferenceエンドポイントを直接呼び出す**ことでモデルを検出します:

```bash
GET https://router.huggingface.co/v1/models
```

(オプション: 完全なリストを取得するために `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` または `$HF_TOKEN` を送信します。認証なしでは一部のエンドポイントはサブセットを返します。) レスポンスはOpenAI形式の `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }` です。Hugging Face APIキー（オンボーディング、`HUGGINGFACE_HUB_TOKEN`、または `HF_TOKEN` 経由）を設定すると、OpenClawはこのGETリクエストを使用して利用可能なチャット補完モデルを検出します。**対話型オンボーディング**中、トークンを入力した後、そのリスト（またはリクエストが失敗した場合は組み込みカタログ）から読み込まれた **Default Hugging Face model** ドロップダウンが表示されます。実行時（例: Gateway起動時）、キーが存在する場合、OpenClawは再度 **GET** `https://router.huggingface.co/v1/models` を呼び出してカタログを更新します。このリストは組み込みカタログ（コンテキストウィンドウやコストなどのメタデータ用）とマージされます。リクエストが失敗した場合、またはキーが設定されていない場合は、組み込みカタログのみが使用されます。

## モデル名と編集可能なオプション

-   **APIからの名前:** モデルの表示名は、APIが `name`、`title`、または `display_name` を返す場合、**GET /v1/models からハイドレートされます**。それ以外の場合は、モデルIDから派生します（例: `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”）。
-   **表示名の上書き:** 設定でモデルごとにカスタムラベルを設定し、CLIやUIで希望する方法で表示させることができます:

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (高速)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (低コスト)" },
      },
    },
  },
}
```

-   **プロバイダー / ポリシー選択:** **モデルID** にサフィックスを追加して、ルーターがバックエンドを選択する方法を指定します:
    
    -   **`:fastest`** — 最高スループット（ルーターが選択; プロバイダー選択は **ロック** されます — インタラクティブなバックエンド選択は表示されません）。
    -   **`:cheapest`** — 出力トークンあたりの最低コスト（ルーターが選択; プロバイダー選択は **ロック** されます）。
    -   **`:provider`** — 特定のバックエンドを強制（例: `:sambanova`, `:together`）。
    
    **:cheapest** または **:fastest**（例: オンボーディングのモデルドロップダウンで）を選択すると、プロバイダーはロックされます: ルーターがコストまたは速度に基づいて決定し、オプションの「特定のバックエンドを優先」ステップは表示されません。これらを `models.providers.huggingface.models` の個別のエントリとして追加するか、サフィックス付きで `model.primary` を設定できます。また、[Inference Provider settings](https://hf.co/settings/inference-providers) でデフォルトの順序を設定することもできます（サフィックスなし = その順序を使用）。
-   **設定のマージ:** `models.providers.huggingface.models` 内の既存のエントリ（例: `models.json` 内）は、設定がマージされても保持されます。したがって、そこに設定したカスタムの `name`、`alias`、またはモデルオプションは保持されます。

## モデルIDと構成例

モデル参照は `huggingface//`（Hub形式のID）の形式を使用します。以下のリストは **GET** `https://router.huggingface.co/v1/models` からのものです。あなたのカタログにはさらに多くのモデルが含まれる場合があります。**IDの例（Inferenceエンドポイントから）:**

| モデル | 参照 (`huggingface/` を接頭辞として) |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

モデルIDに `:fastest`、`:cheapest`、または `:provider`（例: `:together`、`:sambanova`）を追加できます。デフォルトの順序は [Inference Provider settings](https://hf.co/settings/inference-providers) で設定してください。完全なリストについては、[Inference Providers](https://huggingface.co/docs/inference-providers) と **GET** `https://router.huggingface.co/v1/models` を参照してください。

### 完全な構成例

**DeepSeek R1をプライマリ、Qwenをフォールバック:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwenをデフォルトに、:cheapestおよび:fastestバリアントを追加:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (最安値)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (最速)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSSにエイリアスを設定:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**:providerで特定のバックエンドを強制:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**ポリシーサフィックス付きの複数のQwenおよびDeepSeekモデル:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (低コスト)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (高速)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)