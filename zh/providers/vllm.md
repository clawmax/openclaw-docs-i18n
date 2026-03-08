

  提供商

  
# vLLM

vLLM 可以通过 **OpenAI 兼容的** HTTP API 提供开源（以及一些自定义）模型服务。OpenClaw 可以使用 `openai-completions` API 连接到 vLLM。当您通过设置 `VLLM_API_KEY`（如果您的服务器不强制要求身份验证，任何值都有效）选择加入，并且没有定义显式的 `models.providers.vllm` 条目时，OpenClaw 还可以**自动发现** vLLM 中的可用模型。

## 快速入门

1.  使用 OpenAI 兼容的服务器启动 vLLM。

您的基础 URL 应暴露 `/v1` 端点（例如 `/v1/models`、`/v1/chat/completions`）。vLLM 通常运行在：

-   `http://127.0.0.1:8000/v1`

2.  选择加入（如果未配置身份验证，任何值都有效）：

```bash
export VLLM_API_KEY="vllm-local"
```

3.  选择一个模型（替换为您的 vLLM 模型 ID 之一）：

```json
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## 模型发现（隐式提供商）

当设置了 `VLLM_API_KEY`（或存在身份验证配置文件）并且您**没有**定义 `models.providers.vllm` 时，OpenClaw 将查询：

-   `GET http://127.0.0.1:8000/v1/models`

…并将返回的 ID 转换为模型条目。如果您显式设置了 `models.providers.vllm`，则会跳过自动发现，您必须手动定义模型。

## 显式配置（手动模型）

在以下情况下使用显式配置：

-   vLLM 运行在不同的主机/端口上。
-   您希望固定 `contextWindow`/`maxTokens` 值。
-   您的服务器需要真实的 API 密钥（或者您希望控制请求头）。

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "本地 vLLM 模型",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 故障排除

-   检查服务器是否可达：

```bash
curl http://127.0.0.1:8000/v1/models
```

-   如果请求因身份验证错误而失败，请设置与您的服务器配置匹配的真实 `VLLM_API_KEY`，或者在 `models.providers.vllm` 下显式配置提供商。

[Venice AI](./venice.md)[小米 MiMo](./xiaomi.md)