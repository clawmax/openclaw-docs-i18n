

  协议与 API

  
# 本地模型

本地部署是可行的，但 OpenClaw 需要大上下文窗口和强大的提示注入防御能力。小型显卡会截断上下文并降低安全性。目标要高：**≥2 台满配 Mac Studio 或同等 GPU 设备（约 3 万美元以上）**。单个 **24 GB** GPU 仅适用于延迟较高的轻量提示。使用**您能运行的最大/完整尺寸模型变体**；过度量化或“小型”检查点会增加提示注入风险（参见[安全](./security.md)）。

## 推荐方案：LM Studio + MiniMax M2.5（Responses API，完整尺寸）

当前最佳的本地技术栈。在 LM Studio 中加载 MiniMax M2.5，启用本地服务器（默认 `http://127.0.0.1:1234`），并使用 Responses API 将推理过程与最终文本分离。

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**设置清单**

-   安装 LM Studio：[https://lmstudio.ai](https://lmstudio.ai)
-   在 LM Studio 中，下载**可用的最大 MiniMax M2.5 版本**（避免“小型”/重度量化变体），启动服务器，确认 `http://127.0.0.1:1234/v1/models` 列表中包含该模型。
-   保持模型加载状态；冷启动会增加启动延迟。
-   如果您的 LM Studio 版本不同，请调整 `contextWindow`/`maxTokens`。
-   对于 WhatsApp，坚持使用 Responses API，以便只发送最终文本。

即使运行本地模型，也请保持托管模型的配置；使用 `models.mode: "merge"` 以确保回退方案可用。

### 混合配置：托管模型为主，本地模型为回退

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### 本地优先，托管模型作为安全网

交换主模型和回退模型的顺序；保持相同的 providers 块和 `models.mode: "merge"`，以便在本地设备宕机时可以回退到 Sonnet 或 Opus。

### 区域托管 / 数据路由

-   托管的 MiniMax/Kimi/GLM 变体也存在于 OpenRouter 上，并具有区域固定的端点（例如，美国托管）。在那里选择区域变体，以保持流量在您选择的司法管辖区内，同时仍可使用 `models.mode: "merge"` 来获得 Anthropic/OpenAI 回退。
-   纯本地部署仍然是隐私最强的路径；当您需要提供商功能但又想控制数据流时，托管区域路由是中间方案。

## 其他 OpenAI 兼容的本地代理

vLLM、LiteLLM、OAI-proxy 或自定义网关，只要它们暴露一个 OpenAI 风格的 `/v1` 端点，就可以工作。将上面的 provider 块替换为您的端点和模型 ID：

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

保持 `models.mode: "merge"`，以便托管模型作为回退方案保持可用。

## 故障排除

-   网关能否访问代理？运行 `curl http://127.0.0.1:1234/v1/models`。
-   LM Studio 模型是否已卸载？重新加载；冷启动是常见的“挂起”原因。
-   上下文错误？降低 `contextWindow` 或提高您的服务器限制。
-   安全性：本地模型跳过了提供商端的过滤器；保持代理范围狭窄并启用压缩，以限制提示注入的攻击范围。

[CLI 后端](./cli-backends.md)[网络模型](./network-model.md)