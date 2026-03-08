

  提供商

  
# Ollama

Ollama 是一个本地 LLM 运行时，可以轻松在您的机器上运行开源模型。OpenClaw 与 Ollama 的原生 API (`/api/chat`) 集成，支持流式传输和工具调用，并且当您通过设置 `OLLAMA_API_KEY`（或认证配置文件）选择加入且**未**定义显式的 `models.providers.ollama` 条目时，可以**自动发现支持工具调用的模型**。

> **⚠️** **远程 Ollama 用户**：请勿将 OpenAI 兼容的 `/v1` URL (`http://host:11434/v1`) 与 OpenClaw 一起使用。这会破坏工具调用，模型可能会将原始工具 JSON 作为纯文本输出。请改用原生 Ollama API URL：`baseUrl: "http://host:11434"`（不带 `/v1`）。

## 快速开始

1.  安装 Ollama：[https://ollama.ai](https://ollama.ai)
2.  拉取一个模型：

```bash
ollama pull gpt-oss:20b
# 或
ollama pull llama3.3
# 或
ollama pull qwen2.5-coder:32b
# 或
ollama pull deepseek-r1:32b
```

3.  为 OpenClaw 启用 Ollama（任何值都有效；Ollama 不需要真实的密钥）：

```bash
# 设置环境变量
export OLLAMA_API_KEY="ollama-local"

# 或在配置文件中配置
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  使用 Ollama 模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## 模型发现（隐式提供商）

当您设置 `OLLAMA_API_KEY`（或认证配置文件）并且**不**定义 `models.providers.ollama` 时，OpenClaw 会从位于 `http://127.0.0.1:11434` 的本地 Ollama 实例发现模型：

-   查询 `/api/tags` 和 `/api/show`
-   仅保留报告具有 `tools` 能力的模型
-   当模型报告 `thinking` 时标记 `reasoning`
-   在可用时从 `model_info[".context_length"]` 读取 `contextWindow`
-   将 `maxTokens` 设置为上下文窗口的 10 倍
-   将所有成本设置为 `0`

这避免了手动定义模型条目，同时使目录与 Ollama 的能力保持一致。要查看有哪些模型可用：

```bash
ollama list
openclaw models list
```

要添加新模型，只需使用 Ollama 拉取它：

```bash
ollama pull mistral
```

新模型将被自动发现并可供使用。如果您显式设置了 `models.providers.ollama`，则会跳过自动发现，您必须手动定义模型（见下文）。

## 配置

### 基本设置（隐式发现）

启用 Ollama 的最简单方法是通过环境变量：

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 显式设置（手动定义模型）

在以下情况下使用显式配置：

-   Ollama 运行在另一台主机/端口上。
-   您希望强制指定特定的上下文窗口或模型列表。
-   您希望包含未报告工具支持的模型。

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

如果设置了 `OLLAMA_API_KEY`，您可以在提供商条目中省略 `apiKey`，OpenClaw 会为可用性检查填充它。

### 自定义基础 URL（显式配置）

如果 Ollama 运行在不同的主机或端口上（显式配置会禁用自动发现，因此需要手动定义模型）：

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // 不要加 /v1 - 使用原生 Ollama API URL
        api: "ollama", // 显式设置以保证原生的工具调用行为
      },
    },
  },
}
```

> **⚠️** 不要在 URL 中添加 `/v1`。`/v1` 路径使用 OpenAI 兼容模式，其中工具调用不可靠。请使用不带路径后缀的基础 Ollama URL。

### 模型选择

配置完成后，您所有的 Ollama 模型都可用：

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 高级

### 推理模型

当 Ollama 在 `/api/show` 中报告 `thinking` 时，OpenClaw 会将模型标记为具有推理能力：

```bash
ollama pull deepseek-r1:32b
```

### 模型成本

Ollama 是免费的并在本地运行，因此所有模型成本都设置为 $0。

### 流式传输配置

OpenClaw 的 Ollama 集成默认使用**原生 Ollama API** (`/api/chat`)，它完全支持同时进行流式传输和工具调用。无需特殊配置。

#### 旧版 OpenAI 兼容模式

> **⚠️** **在 OpenAI 兼容模式下，工具调用不可靠。** 仅当您需要 OpenAI 格式用于代理且不依赖原生工具调用行为时才使用此模式。

如果您需要使用 OpenAI 兼容端点（例如，在仅支持 OpenAI 格式的代理后面），请显式设置 `api: "openai-completions"`：

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // 默认: true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

此模式可能不支持同时进行流式传输和工具调用。您可能需要在模型配置中使用 `params: { streaming: false }` 来禁用流式传输。当 `api: "openai-completions"` 与 Ollama 一起使用时，OpenClaw 默认会注入 `options.num_ctx`，以防止 Ollama 静默回退到 4096 上下文窗口。如果您的代理/上游拒绝未知的 `options` 字段，请禁用此行为：

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### 上下文窗口

对于自动发现的模型，OpenClaw 使用 Ollama 报告的上下文窗口（如果可用），否则默认为 `8192`。您可以在显式提供商配置中覆盖 `contextWindow` 和 `maxTokens`。

## 故障排除

### Ollama 未检测到

确保 Ollama 正在运行，并且您设置了 `OLLAMA_API_KEY`（或认证配置文件），并且您**没有**定义显式的 `models.providers.ollama` 条目：

```bash
ollama serve
```

并确保 API 可访问：

```bash
curl http://localhost:11434/api/tags
```

### 没有可用模型

OpenClaw 仅自动发现报告工具支持的模型。如果您的模型未列出，请：

-   拉取一个支持工具调用的模型，或
-   在 `models.providers.ollama` 中显式定义该模型。

添加模型：

```bash
ollama list  # 查看已安装的模型
ollama pull gpt-oss:20b  # 拉取一个支持工具调用的模型
ollama pull llama3.3     # 或另一个模型
```

### 连接被拒绝

检查 Ollama 是否在正确的端口上运行：

```bash
# 检查 Ollama 是否正在运行
ps aux | grep ollama

# 或重启 Ollama
ollama serve
```

## 另请参阅

-   [模型提供商](../concepts/model-providers.md) - 所有提供商的概述
-   [模型选择](../concepts/models.md) - 如何选择模型
-   [配置](../gateway/configuration.md) - 完整的配置参考

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)

---