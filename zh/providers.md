

  概述

  
# 模型供应商

OpenClaw 可以使用许多 LLM 供应商。选择一个供应商，完成认证，然后将默认模型设置为 `供应商/模型`。寻找聊天渠道文档（WhatsApp/Telegram/Discord/Slack/Mattermost (插件)/等）？请参阅 [渠道](./channels.md)。

## 快速开始

1.  与供应商完成认证（通常通过 `openclaw onboard` 命令）。
2.  设置默认模型：

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 供应商文档

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [GLM 模型](./providers/glm.md)
-   [Hugging Face (推理)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (统一网关)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (本地模型)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, 注重隐私)](./providers/venice.md)
-   [vLLM (本地模型)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## 转录供应商

-   [Deepgram (音频转录)](./providers/deepgram.md)

## 社区工具

-   [Claude Max API 代理](./providers/claude-max-api-proxy.md) - 用于 Claude 订阅凭证的社区代理（使用前请核实 Anthropic 政策/条款）

有关完整的供应商目录（xAI、Groq、Mistral 等）和高级配置，请参阅 [模型供应商](./concepts/model-providers.md)。

[模型供应商快速入门](./providers/models.md)