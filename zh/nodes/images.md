

  媒体与设备

  
# 图片与媒体支持

WhatsApp 频道通过 **Baileys Web** 运行。本文档记录了当前发送、网关和代理回复的媒体处理规则。

## 目标

-   通过 `openclaw message send --media` 发送带有可选说明文字的媒体。
-   允许来自网页收件箱的自动回复包含媒体和文本。
-   保持每种类型的限制合理且可预测。

## CLI 界面

-   `openclaw message send --media <path-or-url> [--message ]`
    -   `--media` 可选；对于仅发送媒体的情况，说明文字可以为空。
    -   `--dry-run` 打印解析后的负载；`--json` 输出 `{ channel, to, messageId, mediaUrl, caption }`。

## WhatsApp Web 频道行为

-   输入：本地文件路径 **或** HTTP(S) URL。
-   流程：加载到 Buffer 中，检测媒体类型，并构建正确的负载：
    -   **图片：** 调整大小并重新压缩为 JPEG（最大边长 2048px），目标为 `agents.defaults.mediaMaxMb`（默认 5 MB），上限为 6 MB。
    -   **音频/语音/视频：** 直通传输，上限 16 MB；音频作为语音笔记发送（`ptt: true`）。
    -   **文档：** 其他任何文件，上限 100 MB，在可用时保留文件名。
-   WhatsApp GIF 风格播放：发送带有 `gifPlayback: true` 的 MP4（CLI：`--gif-playback`），以便移动客户端内联循环播放。
-   MIME 检测优先使用魔数字节，然后是头部，最后是文件扩展名。
-   说明文字来自 `--message` 或 `reply.text`；允许空说明文字。
-   日志记录：非详细模式显示 `↩️`/`✅`；详细模式包括大小和源路径/URL。

## 自动回复流程

-   `getReplyFromConfig` 返回 `{ text?, mediaUrl?, mediaUrls? }`。
-   当存在媒体时，网页发送器使用与 `openclaw message send` 相同的流程解析本地路径或 URL。
-   如果提供了多个媒体条目，则按顺序发送。

## 入站媒体到命令 (Pi)

-   当入站网页消息包含媒体时，OpenClaw 会下载到临时文件并暴露模板变量：
    -   `{{MediaUrl}}` 入站媒体的伪 URL。
    -   `{{MediaPath}}` 运行命令前写入的本地临时路径。
-   当启用每个会话的 Docker 沙箱时，入站媒体被复制到沙箱工作区，并且 `MediaPath`/`MediaUrl` 被重写为相对路径，如 `media/inbound/`。
-   媒体理解（如果通过 `tools.media.*` 或共享的 `tools.media.models` 配置）在模板化之前运行，并可以将 `[Image]`、`[Audio]` 和 `[Video]` 块插入到 `Body` 中。
    -   音频设置 `{{Transcript}}` 并使用转录文本进行命令解析，因此斜杠命令仍然有效。
    -   视频和图片描述会保留任何说明文字用于命令解析。
-   默认情况下，仅处理第一个匹配的图片/音频/视频附件；设置 `tools.media..attachments` 以处理多个附件。

## 限制与错误

**出站发送上限（WhatsApp web 发送）**

-   图片：重新压缩后上限约 6 MB。
-   音频/语音/视频：上限 16 MB；文档：上限 100 MB。
-   超大或不可读的媒体 → 日志中显示明确错误，并且跳过回复。

**媒体理解上限（转录/描述）**

-   图片默认：10 MB (`tools.media.image.maxBytes`)。
-   音频默认：20 MB (`tools.media.audio.maxBytes`)。
-   视频默认：50 MB (`tools.media.video.maxBytes`)。
-   超大媒体跳过理解，但回复仍会以原始正文发送。

## 测试注意事项

-   覆盖图片/音频/文档情况的发送 + 回复流程。
-   验证图片的重新压缩（大小限制）和音频的语音笔记标志。
-   确保多媒体回复作为顺序发送展开。

[媒体理解](./media-understanding.md)[音频与语音笔记](./audio.md)

---