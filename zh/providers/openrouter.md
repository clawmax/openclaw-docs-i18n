

  提供商

  
# OpenRouter

OpenRouter 提供了一个**统一 API**，通过单个端点和 API 密钥将请求路由到众多模型。它与 OpenAI 兼容，因此大多数 OpenAI SDK 只需切换基础 URL 即可使用。

## CLI 设置

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 配置片段

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## 注意事项

-   模型引用格式为 `openrouter//`。
-   更多模型/提供商选项，请参阅 [/concepts/model-providers](../concepts/model-providers.md)。
-   OpenRouter 在底层使用带有您 API 密钥的 Bearer 令牌。

[OpenCode Zen](./opencode.md)[千帆](./qianfan.md)