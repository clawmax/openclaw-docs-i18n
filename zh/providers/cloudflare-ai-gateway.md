

  提供商

  
# Cloudflare AI Gateway

Cloudflare AI Gateway 位于提供商 API 的前端，允许您添加分析、缓存和控制功能。对于 Anthropic，OpenClaw 通过您的 Gateway 端点使用 Anthropic Messages API。

-   提供商：`cloudflare-ai-gateway`
-   基础 URL：`https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   默认模型：`cloudflare-ai-gateway/claude-sonnet-4-5`
-   API 密钥：`CLOUDFLARE_AI_GATEWAY_API_KEY`（用于通过 Gateway 请求的提供商 API 密钥）

对于 Anthropic 模型，请使用您的 Anthropic API 密钥。

## 快速开始

1.  设置提供商 API 密钥和 Gateway 详细信息：

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  设置一个默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## 非交互式示例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## 已认证的网关

如果您在 Cloudflare 中启用了 Gateway 身份验证，请添加 `cf-aig-authorization` 请求头（这是对您的提供商 API 密钥的补充）。

```json
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## 环境说明

如果 Gateway 作为守护进程（launchd/systemd）运行，请确保 `CLOUDFLARE_AI_GATEWAY_API_KEY` 对该进程可用（例如，在 `~/.openclaw/.env` 中或通过 `env.shellEnv`）。

[Amazon Bedrock](./bedrock.md)[Claude Max API 代理](./claude-max-api-proxy.md)