

  設定

  
# モデルプロバイダー

このページでは、**LLM/モデルプロバイダー**（WhatsApp/Telegramなどのチャットチャネルではありません）について説明します。モデル選択ルールについては、[/concepts/models](./models.md) を参照してください。

## クイックルール

-   モデル参照は `プロバイダー/モデル` を使用します（例: `opencode/claude-opus-4-6`）。
-   `agents.defaults.models` を設定すると、それが許可リストになります。
-   CLIヘルパー: `openclaw onboard`, `openclaw models list`, `openclaw models set <プロバイダー/モデル>`。

## APIキーローテーション

-   選択されたプロバイダーに対して、汎用プロバイダーローテーションをサポートします。
-   以下の方法で複数のキーを設定できます:
    -   `OPENCLAW_LIVE__KEY` (単一のライブオーバーライド、最高優先度)
    -   `_API_KEYS` (カンマまたはセミコロン区切りのリスト)
    -   `_API_KEY` (プライマリキー)
    -   `_API_KEY_*` (番号付きリスト、例: `_API_KEY_1`)
-   Googleプロバイダーの場合、`GOOGLE_API_KEY` もフォールバックとして含まれます。
-   キー選択順序は優先度を保持し、重複する値を排除します。
-   リクエストは、レート制限応答（例: `429`、`rate_limit`、`quota`、`resource exhausted`）の場合のみ、次のキーで再試行されます。
-   レート制限以外の失敗は即座に失敗し、キーローテーションは試行されません。
-   すべての候補キーが失敗した場合、最後の試行からの最終エラーが返されます。

## 組み込みプロバイダー (pi-ai カタログ)

OpenClawにはpi‑aiカタログが同梱されています。これらのプロバイダーは **`models.providers` 設定を必要としません**。認証を設定してモデルを選択するだけです。

### OpenAI

-   プロバイダー: `openai`
-   認証: `OPENAI_API_KEY`
-   オプションのローテーション: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`、および `OPENCLAW_LIVE_OPENAI_KEY` (単一オーバーライド)
-   モデル例: `openai/gpt-5.1-codex`
-   CLI: `openclaw onboard --auth-choice openai-api-key`
-   デフォルトトランスポートは `auto` (WebSocket優先、SSEフォールバック)
-   モデルごとに `agents.defaults.models["openai/"].params.transport` (`"sse"`、`"websocket"`、または `"auto"`) でオーバーライド可能
-   OpenAI Responses WebSocketウォームアップは、`params.openaiWsWarmup` (`true`/`false`) によりデフォルトで有効

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   プロバイダー: `anthropic`
-   認証: `ANTHROPIC_API_KEY` または `claude setup-token`
-   オプションのローテーション: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`、および `OPENCLAW_LIVE_ANTHROPIC_KEY` (単一オーバーライド)
-   モデル例: `anthropic/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice token` (setup-tokenを貼り付け) または `openclaw models auth paste-token --provider anthropic`
-   ポリシー注意: setup-tokenサポートは技術的な互換性のためです。Anthropicは過去にClaude Code外での一部のサブスクリプション使用をブロックしたことがあります。現在のAnthropic利用規約を確認し、ご自身のリスク許容度に基づいて判断してください。
-   推奨: Anthropic APIキー認証は、サブスクリプションsetup-token認証よりも安全で推奨される方法です。

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   プロバイダー: `openai-codex`
-   認証: OAuth (ChatGPT)
-   モデル例: `openai-codex/gpt-5.3-codex`
-   CLI: `openclaw onboard --auth-choice openai-codex` または `openclaw models auth login --provider openai-codex`
-   デフォルトトランスポートは `auto` (WebSocket優先、SSEフォールバック)
-   モデルごとに `agents.defaults.models["openai-codex/"].params.transport` (`"sse"`、`"websocket"`、または `"auto"`) でオーバーライド可能
-   ポリシー注意: OpenAI Codex OAuthは、OpenClawのような外部ツール/ワークフローに対して明示的にサポートされています。

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   プロバイダー: `opencode`
-   認証: `OPENCODE_API_KEY` (または `OPENCODE_ZEN_API_KEY`)
-   モデル例: `opencode/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (APIキー)

-   プロバイダー: `google`
-   認証: `GEMINI_API_KEY`
-   オプションのローテーション: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, `GOOGLE_API_KEY` フォールバック、および `OPENCLAW_LIVE_GEMINI_KEY` (単一オーバーライド)
-   モデル例: `google/gemini-3-pro-preview`
-   CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex、Antigravity、および Gemini CLI

-   プロバイダー: `google-vertex`, `google-antigravity`, `google-gemini-cli`
-   認証: Vertexはgcloud ADCを使用。Antigravity/Gemini CLIはそれぞれの認証フローを使用
-   注意: OpenClawでのAntigravityおよびGemini CLI OAuthは非公式統合です。サードパーティクライアント使用後にGoogleアカウント制限を受けたというユーザーレポートがあります。Google利用規約を確認し、進める場合は重要でないアカウントを使用してください。
-   Antigravity OAuthはバンドルプラグイン (`google-antigravity-auth`、デフォルト無効) として同梱されています。
    -   有効化: `openclaw plugins enable google-antigravity-auth`
    -   ログイン: `openclaw models auth login --provider google-antigravity --set-default`
-   Gemini CLI OAuthはバンドルプラグイン (`google-gemini-cli-auth`、デフォルト無効) として同梱されています。
    -   有効化: `openclaw plugins enable google-gemini-cli-auth`
    -   ログイン: `openclaw models auth login --provider google-gemini-cli --set-default`
    -   注意: `openclaw.json` にクライアントIDやシークレットを貼り付ける必要は **ありません**。CLIログインフローは、ゲートウェイホスト上の認証プロファイルにトークンを保存します。

### Z.AI (GLM)

-   プロバイダー: `zai`
-   認証: `ZAI_API_KEY`
-   モデル例: `zai/glm-5`
-   CLI: `openclaw onboard --auth-choice zai-api-key`
    -   エイリアス: `z.ai/*` および `z-ai/*` は `zai/*` に正規化されます

### Vercel AI Gateway

-   プロバイダー: `vercel-ai-gateway`
-   認証: `AI_GATEWAY_API_KEY`
-   モデル例: `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   プロバイダー: `kilocode`
-   認証: `KILOCODE_API_KEY`
-   モデル例: `kilocode/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --kilocode-api-key `
-   ベースURL: `https://api.kilo.ai/api/gateway/`
-   拡張された組み込みカタログには、GLM-5 Free、MiniMax M2.5 Free、GPT-5.2、Gemini 3 Pro Preview、Gemini 3 Flash Preview、Grok Code Fast 1、および Kimi K2.5 が含まれます。

詳細なセットアップについては、[/providers/kilocode](../providers/kilocode.md) を参照してください。

### その他の組み込みプロバイダー

-   OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
-   モデル例: `openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway: `kilocode` (`KILOCODE_API_KEY`)
-   モデル例: `kilocode/anthropic/claude-opus-4.6`
-   xAI: `xai` (`XAI_API_KEY`)
-   Mistral: `mistral` (`MISTRAL_API_KEY`)
-   モデル例: `mistral/mistral-large-latest`
-   CLI: `openclaw onboard --auth-choice mistral-api-key`
-   Groq: `groq` (`GROQ_API_KEY`)
-   Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
    -   Cerebras上のGLMモデルは、ID `zai-glm-4.7` および `zai-glm-4.6` を使用します。
    -   OpenAI互換ベースURL: `https://api.cerebras.ai/v1`。
-   GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` または `HF_TOKEN`) — OpenAI互換ルーター。モデル例: `huggingface/deepseek-ai/DeepSeek-R1`。CLI: `openclaw onboard --auth-choice huggingface-api-key`。詳細は [Hugging Face (Inference)](../providers/huggingface.md) を参照。

## models.providers によるプロバイダー (カスタム/ベースURL)

`models.providers` (または `models.json`) を使用して、**カスタム**プロバイダーまたは OpenAI/Anthropic互換プロキシを追加します。

### Moonshot AI (Kimi)

MoonshotはOpenAI互換エンドポイントを使用するため、カスタムプロバイダーとして設定します:

-   プロバイダー: `moonshot`
-   認証: `MOONSHOT_API_KEY`
-   モデル例: `moonshot/kimi-k2.5`

Kimi K2 モデルID:

-   `moonshot/kimi-k2.5`
-   `moonshot/kimi-k2-0905-preview`
-   `moonshot/kimi-k2-turbo-preview`
-   `moonshot/kimi-k2-thinking`
-   `moonshot/kimi-k2-thinking-turbo`

```json
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      },
    },
  },
}
```

### Kimi Coding

Kimi CodingはMoonshot AIのAnthropic互換エンドポイントを使用します:

-   プロバイダー: `kimi-coding`
-   認証: `KIMI_API_KEY`
-   モデル例: `kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (無料ティア)

Qwenはデバイスコードフローを介してQwen Coder + VisionへのOAuthアクセスを提供します。バンドルプラグインを有効にしてからログインします:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

モデル参照:

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

セットアップの詳細と注意点については、[/providers/qwen](../providers/qwen.md) を参照してください。

### Volcano Engine (Doubao)

Volcano Engine (火山引擎) は、中国国内でのDoubaoおよびその他のモデルへのアクセスを提供します。

-   プロバイダー: `volcengine` (コーディング: `volcengine-plan`)
-   認証: `VOLCANO_ENGINE_API_KEY`
-   モデル例: `volcengine/doubao-seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

利用可能なモデル:

-   `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

コーディングモデル (`volcengine-plan`):

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (国際版)

BytePlus ARKは、国際ユーザー向けにVolcano Engineと同じモデルへのアクセスを提供します。

-   プロバイダー: `byteplus` (コーディング: `byteplus-plan`)
-   認証: `BYTEPLUS_API_KEY`
-   モデル例: `byteplus/seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

利用可能なモデル:

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

コーディングモデル (`byteplus-plan`):

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

Syntheticは、`synthetic`プロバイダーの背後でAnthropic互換モデルを提供します:

-   プロバイダー: `synthetic`
-   認証: `SYNTHETIC_API_KEY`
-   モデル例: `synthetic/hf:MiniMaxAI/MiniMax-M2.5`
-   CLI: `openclaw onboard --auth-choice synthetic-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.5", name: "MiniMax M2.5" }],
      },
    },
  },
}
```

### MiniMax

MiniMaxはカスタムエンドポイントを使用するため、`models.providers` を介して設定します:

-   MiniMax (Anthropic互換): `--auth-choice minimax-api`
-   認証: `MINIMAX_API_KEY`

セットアップの詳細、モデルオプション、設定スニペットについては、[/providers/minimax](../providers/minimax.md) を参照してください。

### Ollama

Ollamaは、OpenAI互換APIを提供するローカルLLMランタイムです:

-   プロバイダー: `ollama`
-   認証: 不要 (ローカルサーバー)
-   モデル例: `ollama/llama3.3`
-   インストール: [https://ollama.ai](https://ollama.ai)

```bash
# Ollamaをインストールし、モデルをプルします:
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollamaは、`http://127.0.0.1:11434/v1` でローカル実行時に自動検出されます。モデルの推奨事項とカスタム設定については、[/providers/ollama](../providers/ollama.md) を参照してください。

### vLLM

vLLMは、ローカル（またはセルフホスト）のOpenAI互換サーバーです:

-   プロバイダー: `vllm`
-   認証: オプション (サーバー設定による)
-   デフォルトベースURL: `http://127.0.0.1:8000/v1`

ローカルでの自動検出をオプトインするには（サーバーが認証を強制しない場合、任意の値で動作）:

```bash
export VLLM_API_KEY="vllm-local"
```

次にモデルを設定します (`/v1/models` から返されるIDのいずれかに置き換えてください):

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

詳細については、[/providers/vllm](../providers/vllm.md) を参照してください。

### ローカルプロキシ (LM Studio、vLLM、LiteLLMなど)

例 (OpenAI互換):

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

注意:

-   カスタムプロバイダーの場合、`reasoning`、`input`、`cost`、`contextWindow`、`maxTokens` はオプションです。省略すると、OpenClawは以下をデフォルトとします:
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   推奨: プロキシ/モデルの制限に一致する明示的な値を設定してください。

## CLI例

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

完全な設定例については、[/gateway/configuration](../gateway/configuration.md) も参照してください。

[Models CLI](./models.md)[Model Failover](./model-failover.md)