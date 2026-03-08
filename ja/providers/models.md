title: "OpenClaw モデルプロバイダー設定と構成ガイド"
description: "OpenClaw で OpenAI、Anthropic、Mistral などの LLM プロバイダーを認証・設定する方法を学びます。クイックスタートガイドと完全なサポートプロバイダー一覧。"
keywords: ["openclaw モデル", "llm プロバイダー", "モデル構成", "anthropic claude", "openai api", "モデル設定", "openrouter", "bedrock"]
---

  概要

  
# モデルプロバイダー クイックスタート

OpenClaw は多くの LLM プロバイダーを利用できます。1つを選択し、認証を行い、デフォルトモデルを `プロバイダー/モデル` として設定してください。

## クイックスタート (2ステップ)

1.  プロバイダーで認証を行う (通常は `openclaw onboard` を使用)。
2.  デフォルトモデルを設定する:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## サポートされているプロバイダー (スターターセット)

-   [OpenAI (API + Codex)](./openai.md)
-   [Anthropic (API + Claude Code CLI)](./anthropic.md)
-   [OpenRouter](./openrouter.md)
-   [Vercel AI Gateway](./vercel-ai-gateway.md)
-   [Cloudflare AI Gateway](./cloudflare-ai-gateway.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./moonshot.md)
-   [Mistral](./mistral.md)
-   [Synthetic](./synthetic.md)
-   [OpenCode Zen](./opencode.md)
-   [Z.AI](./zai.md)
-   [GLM models](./glm.md)
-   [MiniMax](./minimax.md)
-   [Venice (Venice AI)](./venice.md)
-   [Amazon Bedrock](./bedrock.md)
-   [Qianfan](./qianfan.md)

完全なプロバイダーカタログ (xAI, Groq, Mistral など) と高度な構成については、[モデルプロバイダー](../concepts/model-providers.md) を参照してください。

[モデルプロバイダー](../providers.md)[モデル CLI](../concepts/models.md)