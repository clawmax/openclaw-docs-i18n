

  概要

  
# モデルプロバイダー

OpenClawは多くのLLMプロバイダーを利用できます。プロバイダーを選択し、認証を行い、デフォルトモデルを `provider/model` の形式で設定します。チャットチャネルのドキュメント（WhatsApp/Telegram/Discord/Slack/Mattermost (プラグイン)/など）をお探しですか？[チャネル](./channels.md)を参照してください。

## クイックスタート

1.  プロバイダーで認証を行います（通常は `openclaw onboard` を使用）。
2.  デフォルトモデルを設定します：

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## プロバイダードキュメント

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [GLM models](./providers/glm.md)
-   [Hugging Face (Inference)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (unified gateway)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (ローカルモデル)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, プライバシー重視)](./providers/venice.md)
-   [vLLM (ローカルモデル)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## 文字起こしプロバイダー

-   [Deepgram (音声文字起こし)](./providers/deepgram.md)

## コミュニティツール

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - Claudeサブスクリプション認証情報用のコミュニティプロキシ（使用前にAnthropicのポリシー/利用規約を確認してください）

完全なプロバイダーカタログ（xAI、Groq、Mistralなど）と高度な設定については、[モデルプロバイダー](./concepts/model-providers.md)を参照してください。

[モデルプロバイダー クイックスタート](./providers/models.md)

---