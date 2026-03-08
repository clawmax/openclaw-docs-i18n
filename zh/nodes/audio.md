

  媒体与设备

  
# 音频与语音笔记

## 功能概述

-   **媒体理解（音频）**：如果启用了音频理解（或自动检测到），OpenClaw 会：
    1.  定位第一个音频附件（本地路径或 URL），并在需要时下载。
    2.  在发送到每个模型条目之前强制执行 `maxBytes` 限制。
    3.  按顺序运行第一个符合条件的模型条目（提供商或 CLI）。
    4.  如果失败或跳过（大小/超时），则尝试下一个条目。
    5.  成功后，用 `[Audio]` 块替换 `Body` 并设置 `{{Transcript}}`。
-   **命令解析**：转录成功后，`CommandBody`/`RawBody` 会被设置为转录文本，以便斜杠命令仍然有效。
-   **详细日志**：在 `--verbose` 模式下，我们会记录转录何时运行以及何时替换正文。

## 自动检测（默认）

如果您**未配置模型**且 `tools.media.audio.enabled` **未**设置为 `false`，OpenClaw 将按此顺序自动检测，并在第一个可用选项处停止：

1.  **本地 CLI**（如果已安装）
    -   `sherpa-onnx-offline`（需要包含编码器/解码器/连接器/令牌的 `SHERPA_ONNX_MODEL_DIR`）
    -   `whisper-cli`（来自 `whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或捆绑的 tiny 模型）
    -   `whisper`（Python CLI；自动下载模型）
2.  **Gemini CLI** (`gemini`)，使用 `read_many_files`
3.  **提供商密钥**（OpenAI → Groq → Deepgram → Google）

要禁用自动检测，请设置 `tools.media.audio.enabled: false`。要进行自定义，请设置 `tools.media.audio.models`。注意：二进制检测在 macOS/Linux/Windows 上是尽力而为的；请确保 CLI 在 `PATH` 上（我们会扩展 `~`），或者使用完整的命令路径设置显式的 CLI 模型。

## 配置示例

### 提供商 + CLI 回退（OpenAI + Whisper CLI）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### 仅提供商并带范围控制

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### 仅提供商（Deepgram）

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

### 仅提供商（Mistral Voxtral）

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

### 将转录文本回显到聊天（可选）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // 默认为 false
        echoFormat: '📝 "{transcript}"', // 可选，支持 {transcript}
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## 注意事项与限制

-   提供商身份验证遵循标准模型身份验证顺序（身份验证配置文件、环境变量、`models.providers.*.apiKey`）。
-   当使用 `provider: "deepgram"` 时，Deepgram 会获取 `DEEPGRAM_API_KEY`。
-   Deepgram 设置详情：[Deepgram（音频转录）](../providers/deepgram.md)。
-   Mistral 设置详情：[Mistral](../providers/mistral.md)。
-   音频提供商可以通过 `tools.media.audio` 覆盖 `baseUrl`、`headers` 和 `providerOptions`。
-   默认大小上限为 20MB (`tools.media.audio.maxBytes`)。超大的音频文件会被该模型跳过，并尝试下一个条目。
-   小于 1024 字节的微小/空音频文件会在提供商/CLI 转录之前被跳过。
-   音频的默认 `maxChars` 是**未设置**（完整转录）。设置 `tools.media.audio.maxChars` 或每个条目的 `maxChars` 来修剪输出。
-   OpenAI 自动默认使用 `gpt-4o-mini-transcribe`；设置 `model: "gpt-4o-transcribe"` 以获得更高准确度。
-   使用 `tools.media.audio.attachments` 处理多个语音笔记（`mode: "all"` + `maxAttachments`）。
-   转录文本在模板中可用作 `{{Transcript}}`。
-   `tools.media.audio.echoTranscript` 默认关闭；启用它可以在代理处理之前将转录确认发送回原始聊天。
-   `tools.media.audio.echoFormat` 可自定义回显文本（占位符：`{transcript}`）。
-   CLI 标准输出有上限（5MB）；请保持 CLI 输出简洁。

### 代理环境支持

基于提供商的音频转录遵循标准的出站代理环境变量：

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

如果未设置代理环境变量，则使用直接出口。如果代理配置格式错误，OpenClaw 会记录警告并回退到直接获取。

## 群组中的提及检测

当为群组聊天设置 `requireMention: true` 时，OpenClaw 现在会在检查提及**之前**转录音频。这使得语音笔记即使包含提及也能被处理。**工作原理：**

1.  如果语音消息没有文本正文且群组需要提及，OpenClaw 会执行“预检”转录。
2.  检查转录文本中是否存在提及模式（例如，`@BotName`、表情符号触发器）。
3.  如果找到提及，消息将通过完整的回复流程。
4.  转录文本用于提及检测，以便语音笔记可以通过提及门控。

**回退行为：**

-   如果预检期间转录失败（超时、API 错误等），则根据纯文本提及检测来处理消息。
-   这确保了混合消息（文本 + 音频）永远不会被错误地丢弃。

**按 Telegram 群组/主题选择退出：**

-   设置 `channels.telegram.groups..disableAudioPreflight: true` 以跳过该群组的预检转录提及检查。
-   设置 `channels.telegram.groups..topics..disableAudioPreflight` 以按主题覆盖（`true` 跳过，`false` 强制启用）。
-   默认为 `false`（当满足提及门控条件时启用预检）。

**示例：** 用户在 `requireMention: true` 的 Telegram 群组中发送语音笔记说“嘿 @Claude，天气怎么样？”。语音笔记被转录，检测到提及，然后代理回复。

## 常见问题

-   范围规则采用首次匹配优先。`chatType` 被规范化为 `direct`、`group` 或 `room`。
-   确保您的 CLI 以退出码 0 退出并打印纯文本；JSON 需要通过 `jq -r .text` 处理。
-   对于 `parakeet-mlx`，如果您传递 `--output-dir`，当 `--output-format` 是 `txt`（或省略）时，OpenClaw 会读取 `<output-dir>/<media-basename>.txt`；非 `txt` 输出格式会回退到标准输出解析。
-   保持合理的超时时间（`timeoutSeconds`，默认 60 秒），以避免阻塞回复队列。
-   预检转录仅处理**第一个**音频附件以进行提及检测。其他音频将在主媒体理解阶段处理。

[图像与媒体支持](./images.md)[相机捕获](./camera.md)