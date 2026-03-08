

  提供商

  
# Xiaomi MiMo

Xiaomi MiMo 是 **MiMo** 模型的 API 平台。它提供与 OpenAI 和 Anthropic 格式兼容的 REST API，并使用 API 密钥进行身份验证。请在 [Xiaomi MiMo 控制台](https://platform.xiaomimimo.com/#/console/api-keys) 创建您的 API 密钥。OpenClaw 使用 `xiaomi` 提供商及 Xiaomi MiMo API 密钥。

## 模型概览

-   **mimo-v2-flash**: 262144 个令牌的上下文窗口，兼容 Anthropic Messages API。
-   基础 URL: `https://api.xiaomimimo.com/anthropic`
-   授权方式: `Bearer $XIAOMI_API_KEY`

## CLI 设置

```bash
openclaw onboard --auth-choice xiaomi-api-key
# 或非交互式
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## 配置片段

```json
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 注意事项

-   模型引用: `xiaomi/mimo-v2-flash`。
-   当设置了 `XIAOMI_API_KEY`（或存在身份验证配置文件）时，提供商会自动注入。
-   有关提供商规则，请参阅 [/concepts/model-providers](../concepts/model-providers.md)。

[vLLM](./vllm.md)[Z.AI](./zai.md)