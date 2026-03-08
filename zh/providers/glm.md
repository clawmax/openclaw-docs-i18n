

  提供商

  
# GLM 模型

GLM 是一个可通过 Z.AI 平台访问的**模型系列**（并非公司）。在 OpenClaw 中，GLM 模型通过 `zai` 提供商和模型 ID（如 `zai/glm-5`）进行访问。

## CLI 设置

```bash
openclaw onboard --auth-choice zai-api-key
```

## 配置片段

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事项

-   GLM 版本和可用性可能发生变化；请查阅 Z.AI 的文档以获取最新信息。
-   示例模型 ID 包括 `glm-5`、`glm-4.7` 和 `glm-4.6`。
-   有关提供商的详细信息，请参阅 [/providers/zai](./zai.md)。

[Litellm](./litellm.md)[MiniMax](./minimax.md)