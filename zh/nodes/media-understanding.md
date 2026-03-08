

  媒体与设备

  
# 媒体理解

OpenClaw 可以在回复流水线运行之前**总结传入的媒体**（图像/音频/视频）。它会自动检测本地工具或提供商密钥是否可用，并且可以被禁用或自定义。如果理解功能关闭，模型仍会像往常一样接收原始文件/URL。

## 目标

-   可选：将传入的媒体预先消化为简短文本，以实现更快的路由和更好的命令解析。
-   保留原始媒体传递给模型（始终）。
-   支持**提供商 API** 和 **CLI 回退**。
-   允许多个模型按顺序回退（错误/大小/超时）。

## 高层行为

1.  收集传入的附件（`MediaPaths`、`MediaUrls`、`MediaTypes`）。
2.  对于每个启用的能力（图像/音频/视频），根据策略选择附件（默认：**第一个**）。
3.  选择第一个符合条件的模型条目（大小 + 能力 + 授权）。
4.  如果模型失败或媒体过大，**回退到下一个条目**。
5.  成功时：
    -   `Body` 变为 `[Image]`、`[Audio]` 或 `[Video]` 块。
    -   音频设置 `{{Transcript}}`；命令解析在有字幕文本时使用字幕，否则使用转录文本。
    -   字幕作为 `User text:` 保留在块内。

如果理解失败或被禁用，**回复流程将继续**，使用原始正文和附件。

## 配置概述

`tools.media` 支持**共享模型**以及按能力覆盖：

-   `tools.media.models`：共享模型列表（使用 `capabilities` 进行门控）。
-   `tools.media.image` / `tools.media.audio` / `tools.media.video`：
    -   默认值（`prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`）
    -   提供商覆盖（`baseUrl`、`headers`、`providerOptions`）
    -   通过 `tools.media.audio.providerOptions.deepgram` 配置 Deepgram 音频选项
    -   音频转录回显控制（`echoTranscript`，默认 `false`；`echoFormat`）
    -   可选的**按能力 `models` 列表**（优先于共享模型）
    -   `attachments` 策略（`mode`、`maxAttachments`、`prefer`）
    -   `scope`（可选，按频道/聊天类型/会话密钥门控）
-   `tools.media.concurrency`：最大并发能力运行数（默认 **2**）。

```json
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### 模型条目

每个 `models[]` 条目可以是**提供商**或**CLI**：

```json
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLI 模板还可以使用：

-   `{{MediaDir}}`（包含媒体文件的目录）
-   `{{OutputDir}}`（为此运行创建的临时目录）
-   `{{OutputBase}}`（临时文件基本路径，无扩展名）

## 默认值与限制

推荐默认值：

-   `maxChars`：图像/视频为 **500**（简短，便于命令解析）
-   `maxChars`：音频为 **未设置**（完整转录，除非您设置了限制）
-   `maxBytes`：
    -   图像：**10MB**
    -   音频：**20MB**
    -   视频：**50MB**

规则：

-   如果媒体超过 `maxBytes`，则跳过该模型并**尝试下一个模型**。
-   小于 **1024 字节**的音频文件被视为空/损坏，在提供商/CLI 转录之前被跳过。
-   如果模型返回的内容超过 `maxChars`，输出将被截断。
-   `prompt` 默认为简单的“Describe the .”加上 `maxChars` 指导（仅限图像/视频）。
-   如果 `.enabled: true` 但未配置模型，OpenClaw 会在其提供商支持该能力时尝试使用**活动回复模型**。

### 自动检测媒体理解（默认）

如果 `tools.media..enabled` **未**设置为 `false` 且您未配置模型，OpenClaw 按以下顺序自动检测并**在第一个可用选项处停止**：

1.  **本地 CLI**（仅限音频；如果已安装）
    -   `sherpa-onnx-offline`（需要包含编码器/解码器/连接器/令牌的 `SHERPA_ONNX_MODEL_DIR`）
    -   `whisper-cli`（`whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或捆绑的 tiny 模型）
    -   `whisper`（Python CLI；自动下载模型）
2.  **Gemini CLI**（`gemini`）使用 `read_many_files`
3.  **提供商密钥**
    -   音频：OpenAI → Groq → Deepgram → Google
    -   图像：OpenAI → Anthropic → Google → MiniMax
    -   视频：Google

要禁用自动检测，请设置：

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

注意：二进制检测在 macOS/Linux/Windows 上是尽力而为的；确保 CLI 在 `PATH` 上（我们会扩展 `~`），或者使用完整命令路径设置显式 CLI 模型。

### 代理环境支持（提供商模型）

当启用基于提供商的**音频**和**视频**媒体理解时，OpenClaw 会遵循标准的出站代理环境变量用于提供商 HTTP 调用：

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

如果未设置代理环境变量，媒体理解将使用直接出口。如果代理值格式错误，OpenClaw 会记录警告并回退到直接获取。

## 能力（可选）

如果设置了 `capabilities`，则该条目仅针对那些媒体类型运行。对于共享列表，OpenClaw 可以推断默认值：

-   `openai`、`anthropic`、`minimax`：**图像**
-   `google`（Gemini API）：**图像 + 音频 + 视频**
-   `groq`：**音频**
-   `deepgram`：**音频**

对于 CLI 条目，**请显式设置 `capabilities`** 以避免意外匹配。如果省略 `capabilities`，则该条目有资格出现在其所在的列表中。

## 提供商支持矩阵（OpenClaw 集成）

| 能力 | 提供商集成 | 备注 |
| --- | --- | --- |
| 图像 | OpenAI / Anthropic / Google / 其他通过 `pi-ai` | 注册表中的任何支持图像的模型都有效。 |
| 音频 | OpenAI, Groq, Deepgram, Google, Mistral | 提供商转录（Whisper/Deepgram/Gemini/Voxtral）。 |
| 视频 | Google（Gemini API） | 提供商视频理解。 |

## 模型选择指南

-   当质量和安全性很重要时，优先为每种媒体能力选择可用的最强最新一代模型。
-   对于处理不受信任输入的工具赋能代理，避免使用较旧/较弱的媒体模型。
-   为每种能力至少保留一个回退模型以确保可用性（高质量模型 + 更快/更便宜的模型）。
-   当提供商 API 不可用时，CLI 回退（`whisper-cli`、`whisper`、`gemini`）很有用。
-   `parakeet-mlx` 注意：使用 `--output-dir` 时，当输出格式为 `txt`（或未指定）时，OpenClaw 会读取 `<output-dir>/<media-basename>.txt`；非 `txt` 格式则回退到 stdout。

## 附件策略

按能力的 `attachments` 控制处理哪些附件：

-   `mode`：`first`（默认）或 `all`
-   `maxAttachments`：限制处理的数量（默认 **1**）
-   `prefer`：`first`、`last`、`path`、`url`

当 `mode: "all"` 时，输出会标记为 `[Image 1/2]`、`[Audio 2/2]` 等。

## 配置示例

### 1) 共享模型列表 + 覆盖

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) 仅音频 + 视频（图像关闭）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) 可选的图像理解

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) 多模态单一条目（显式能力）

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## 状态输出

当媒体理解运行时，`/status` 包含一个简短的摘要行：

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

这显示了按能力的结果以及在适用时选择的提供商/模型。

## 注意事项

-   理解是**尽力而为的**。错误不会阻塞回复。
-   即使理解被禁用，附件仍会传递给模型。
-   使用 `scope` 来限制理解运行的位置（例如仅限私信）。

## 相关文档

-   [配置](../gateway/configuration.md)
-   [图像和媒体支持](./images.md)

[节点故障排除](./troubleshooting.md)[图像和媒体支持](./images.md)