

  提供商

  
# Z.AI

Z.AI 是 **GLM** 模型的 API 平台。它提供 GLM 的 REST API 并使用 API 密钥进行身份验证。请在 Z.AI 控制台中创建您的 API 密钥。OpenClaw 使用 `zai` 提供商并配合 Z.AI API 密钥。

## CLI 设置

```bash
openclaw onboard --auth-choice zai-api-key
# 或非交互式
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## 配置片段

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事项

-   GLM 模型以 `zai/` 形式提供（例如：`zai/glm-5`）。
-   `tool_stream` 默认启用，用于 Z.AI 工具调用的流式传输。将其设置为 `false` 以禁用它：`agents.defaults.models["zai/"].params.tool_stream`。
-   查看 [/providers/glm](./glm.md) 了解模型系列概览。
-   Z.AI 使用 Bearer 认证和您的 API 密钥。

[小米 MiMo](./xiaomi.md)

---