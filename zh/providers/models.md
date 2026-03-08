

  概述

  
# 模型提供商快速入门

OpenClaw 可以使用许多 LLM 提供商。选择一个提供商，完成认证，然后将默认模型设置为 `provider/model`。

## 快速开始（两步）

1.  向提供商认证（通常通过 `openclaw onboard` 命令）。
2.  设置默认模型：

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 支持的提供商（入门集）

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

有关完整的提供商目录（xAI、Groq、Mistral 等）和高级配置，请参阅[模型提供商](../concepts/model-providers.md)。

[模型提供商](../providers.md)[模型 CLI](../concepts/models.md)

---