

  提供商

  
# Deepgram

Deepgram 是一个语音转文本 API。在 OpenClaw 中，它通过 `tools.media.audio` 用于**入站音频/语音笔记转录**。启用后，OpenClaw 会将音频文件上传到 Deepgram，并将转录文本注入回复管道（`{{Transcript}}` + `[Audio]` 块）。这是**非流式**的；它使用预录制的转录端点。网站：[https://deepgram.com](https://deepgram.com)  
文档：[https://developers.deepgram.com](https://developers.deepgram.com)

## 快速开始

1.  设置您的 API 密钥：

```
DEEPGRAM_API_KEY=dg_...
```

2.  启用提供商：

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## 选项

-   `model`: Deepgram 模型 ID（默认：`nova-3`）
-   `language`: 语言提示（可选）
-   `tools.media.audio.providerOptions.deepgram.detect_language`: 启用语言检测（可选）
-   `tools.media.audio.providerOptions.deepgram.punctuate`: 启用标点符号（可选）
-   `tools.media.audio.providerOptions.deepgram.smart_format`: 启用智能格式化（可选）

带语言设置的示例：

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

带 Deepgram 选项的示例：

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## 注意事项

-   身份验证遵循标准的提供商认证顺序；`DEEPGRAM_API_KEY` 是最简单的路径。
-   使用代理时，可以使用 `tools.media.audio.baseUrl` 和 `tools.media.audio.headers` 覆盖端点或请求头。
-   输出遵循与其他提供商相同的音频规则（大小限制、超时、转录文本注入）。

[Claude Max API 代理](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)