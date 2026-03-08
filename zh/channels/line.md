

  消息平台

  
# LINE

LINE 通过 LINE Messaging API 连接到 OpenClaw。该插件在网关上作为 webhook 接收器运行，并使用您的频道访问令牌和频道密钥进行身份验证。状态：通过插件支持。支持私聊、群聊、媒体、位置、Flex 消息、模板消息和快速回复。不支持消息反应和线程。

## 需要插件

安装 LINE 插件：

```bash
openclaw plugins install @openclaw/line
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/line
```

## 设置

1.  创建 LINE Developers 账户并打开控制台：[https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  创建（或选择）一个 Provider 并添加一个 **Messaging API** 频道。
3.  从频道设置中复制 **频道访问令牌** 和 **频道密钥**。
4.  在 Messaging API 设置中启用 **使用 webhook**。
5.  将 webhook URL 设置为您的网关端点（需要 HTTPS）：

```
https://gateway-host/line/webhook
```

网关响应 LINE 的 webhook 验证（GET）和入站事件（POST）。如果您需要自定义路径，请设置 `channels.line.webhookPath` 或 `channels.line.accounts..webhookPath` 并相应地更新 URL。安全说明：

-   LINE 签名验证依赖于消息体（基于原始消息体的 HMAC），因此 OpenClaw 在验证前会应用严格的预授权消息体大小限制和超时。

## 配置

最小配置：

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

环境变量（仅限默认账户）：

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

令牌/密钥文件：

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

多账户：

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## 访问控制

私聊默认为配对模式。未知发件人会收到一个配对码，在批准之前他们的消息将被忽略。

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

允许列表和策略：

-   `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom`: 允许私聊的 LINE 用户 ID 列表
-   `channels.line.groupPolicy`: `allowlist | open | disabled`
-   `channels.line.groupAllowFrom`: 允许群聊的 LINE 用户 ID 列表
-   按群组覆盖：`channels.line.groups..allowFrom`
-   运行时说明：如果 `channels.line` 配置完全缺失，运行时对于群组检查会回退到 `groupPolicy="allowlist"`（即使设置了 `channels.defaults.groupPolicy`）。

LINE ID 区分大小写。有效的 ID 格式如下：

-   用户：`U` + 32 个十六进制字符
-   群组：`C` + 32 个十六进制字符
-   房间：`R` + 32 个十六进制字符

## 消息行为

-   文本在 5000 个字符处分块。
-   Markdown 格式会被剥离；代码块和表格在可能的情况下会转换为 Flex 卡片。
-   流式响应会被缓冲；当代理工作时，LINE 会收到带有加载动画的完整消息块。
-   媒体下载受 `channels.line.mediaMaxMb` 限制（默认 10）。

## 频道数据（富消息）

使用 `channelData.line` 发送快速回复、位置、Flex 卡片或模板消息。

```json
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

LINE 插件还附带一个用于 Flex 消息预设的 `/card` 命令：

```bash
/card info "Welcome" "Thanks for joining!"
```

## 故障排除

-   **Webhook 验证失败：** 确保 webhook URL 是 HTTPS 且 `channelSecret` 与 LINE 控制台匹配。
-   **没有入站事件：** 确认 webhook 路径与 `channels.line.webhookPath` 匹配，并且 LINE 可以访问到网关。
-   **媒体下载错误：** 如果媒体大小超过默认限制，请提高 `channels.line.mediaMaxMb` 的值。

[IRC](./irc.md)[Matrix](./matrix.md)

---