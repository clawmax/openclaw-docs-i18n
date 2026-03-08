

  提供商

  
# Vercel AI Gateway

[Vercel AI Gateway](https://vercel.com/ai-gateway) 提供了一个统一的 API，通过单一端点访问数百个模型。

-   提供商：`vercel-ai-gateway`
-   认证：`AI_GATEWAY_API_KEY`
-   API：兼容 Anthropic Messages

## 快速开始

1.  设置 API 密钥（推荐：为 Gateway 存储它）：

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  设置一个默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## 非交互式示例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## 环境说明

如果 Gateway 作为守护进程（launchd/systemd）运行，请确保 `AI_GATEWAY_API_KEY` 对该进程可用（例如，在 `~/.openclaw/.env` 中或通过 `env.shellEnv`）。

## 模型 ID 简写

OpenClaw 接受 Vercel Claude 简写模型引用，并在运行时将其规范化：

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)

---