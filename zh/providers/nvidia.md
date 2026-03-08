

  提供商

  
# NVIDIA

NVIDIA 为 Nemotron 和 NeMo 模型提供了一个 OpenAI 兼容的 API，地址为 `https://integrate.api.nvidia.com/v1`。使用来自 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) 的 API 密钥进行身份验证。

## CLI 设置

导出密钥一次，然后运行引导程序并设置一个 NVIDIA 模型：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

如果你仍然传递 `--token`，请记住它会进入 shell 历史记录和 `ps` 输出；尽可能优先使用环境变量。

## 配置片段

```json
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## 模型 ID

-   `nvidia/llama-3.1-nemotron-70b-instruct` (默认)
-   `meta/llama-3.3-70b-instruct`
-   `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 注意事项

-   OpenAI 兼容的 `/v1` 端点；使用来自 NVIDIA NGC 的 API 密钥。
-   当设置 `NVIDIA_API_KEY` 时，提供商自动启用；使用静态默认值（131,072 个令牌的上下文窗口，4,096 个最大令牌数）。

[Mistral](./mistral.md)[Ollama](./ollama.md)