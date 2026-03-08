

  提供商

  
# Kilocode

Kilo Gateway 提供了一个**统一 API**，通过单一端点和 API 密钥将请求路由到众多模型。它与 OpenAI 兼容，因此大多数 OpenAI SDK 只需切换基础 URL 即可使用。

## 获取 API 密钥

1.  访问 [app.kilo.ai](https://app.kilo.ai)
2.  登录或创建账户
3.  导航至 API 密钥页面并生成一个新密钥

## CLI 设置

```bash
openclaw onboard --kilocode-api-key <key>
```

或者设置环境变量：

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## 配置片段

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## 默认模型

默认模型是 `kilocode/kilo/auto`，这是一个智能路由模型，能根据任务自动选择最佳底层模型：

-   规划、调试和编排任务会路由到 Claude Opus
-   代码编写和探索任务会路由到 Claude Sonnet

## 可用模型

OpenClaw 在启动时会动态地从 Kilo Gateway 发现可用模型。使用 `/models kilocode` 命令查看您的账户可用的完整模型列表。网关上可用的任何模型都可以通过 `kilocode/` 前缀使用：

```
kilocode/kilo/auto              (默认 - 智能路由)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...以及更多
```

## 注意事项

-   模型引用格式为 `kilocode/<model-id>`（例如，`kilocode/anthropic/claude-sonnet-4`）。
-   默认模型：`kilocode/kilo/auto`
-   基础 URL：`https://api.kilo.ai/api/gateway/`
-   如需更多模型/提供商选项，请参阅 [/concepts/model-providers](../concepts/model-providers.md)。
-   Kilo Gateway 在底层使用带有您 API 密钥的 Bearer 令牌。

[Hugging Face (推理)](./huggingface.md)[Litellm](./litellm.md)