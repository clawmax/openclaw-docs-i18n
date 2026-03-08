

  提供商

  
# Mistral

OpenClaw 支持 Mistral 用于文本/图像模型路由 (`mistral/...`) 以及通过媒体理解中的 Voxtral 进行音频转录。Mistral 也可用于记忆嵌入 (`memorySearch.provider = "mistral"`)。

## CLI 设置

```bash
openclaw onboard --auth-choice mistral-api-key
# 或非交互式
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## 配置片段 (LLM 提供商)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## 配置片段 (使用 Voxtral 进行音频转录)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## 注意事项

-   Mistral 身份验证使用 `MISTRAL_API_KEY`。
-   提供商基础 URL 默认为 `https://api.mistral.ai/v1`。
-   入门默认模型是 `mistral/mistral-large-latest`。
-   Mistral 的媒体理解默认音频模型是 `voxtral-mini-latest`。
-   媒体转录路径使用 `/v1/audio/transcriptions`。
-   记忆嵌入路径使用 `/v1/embeddings` (默认模型: `mistral-embed`)。

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)