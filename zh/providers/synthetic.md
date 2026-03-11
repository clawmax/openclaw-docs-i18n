

  提供商

  
# Synthetic

想在 OpenClaw 中使用 Synthetic 提供的模型吗？本节帮你快速完成配置。Synthetic 提供 Anthropic 兼容的 API 端点，OpenClaw 将其注册为 `synthetic` 提供商，通过 Anthropic Messages API 进行调用。

## 快速设置

只需两步即可完成：

1. 设置 `SYNTHETIC_API_KEY` 环境变量（或使用下面的向导自动配置）。
2. 运行入门引导命令：

```bash
openclaw onboard --auth-choice synthetic-api-key
```

完成后的默认模型为：

```
synthetic/hf:MiniMaxAI/MiniMax-M2.5
```

## 配置示例

以下是一个完整的配置示例：

```json
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.5": { alias: "MiniMax M2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

需要注意的是，OpenClaw 的 Anthropic 客户端会自动在基础 URL 后追加 `/v1`，因此请填写 `https://api.synthetic.new/anthropic`，而不是 `/anthropic/v1`。如果 Synthetic 更改了其基础 URL，你可以通过覆盖 `models.providers.synthetic.baseUrl` 来更新。

## 模型目录

下表列出了所有可用模型，这些模型的使用成本均为 `0`（包括输入、输出和缓存）。

| 模型 ID | 上下文窗口 | 最大令牌数 | 推理能力 | 输入 |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.5` | 192000 | 65536 | false | text |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | text |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | text |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | text |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | text |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | text |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | text |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | text |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | text |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | text |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | text + image |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | text |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | text |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | text |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | text |

## 注意事项

- 引用模型时使用 `synthetic/` 格式。
- 如果你启用了模型允许列表（`agents.defaults.models`），记得把需要使用的每个模型都添加进去。
- 想了解更多提供商规则？请参阅 [模型提供商](../concepts/model-providers.md)。

[Qwen](./qwen.md)[Together](./together.md)