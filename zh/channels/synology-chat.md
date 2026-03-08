

  消息平台

  
# Synology Chat

状态：通过插件支持，作为使用 Synology Chat webhook 的私信频道。该插件接收来自 Synology Chat 出站 webhook 的入站消息，并通过 Synology Chat 入站 webhook 发送回复。

## 需要插件

Synology Chat 基于插件，不属于默认核心频道安装的一部分。从本地代码库安装：

```bash
openclaw plugins install ./extensions/synology-chat
```

详情：[插件](../tools/plugin.md)

## 快速设置

1.  安装并启用 Synology Chat 插件。
2.  在 Synology Chat 集成中：
    -   创建一个入站 webhook 并复制其 URL。
    -   使用您的密钥令牌创建一个出站 webhook。
3.  将出站 webhook URL 指向您的 OpenClaw 网关：
    -   默认为 `https://gateway-host/webhook/synology`。
    -   或您的自定义 `channels.synology-chat.webhookPath`。
4.  在 OpenClaw 中配置 `channels.synology-chat`。
5.  重启网关并向 Synology Chat 机器人发送一条私信。

最小配置：

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## 环境变量

对于默认账户，您可以使用环境变量：

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (逗号分隔)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

配置值会覆盖环境变量。

## 私信策略与访问控制

-   `dmPolicy: "allowlist"` 是推荐的默认设置。
-   `allowedUserIds` 接受一个 Synology 用户 ID 的列表（或逗号分隔的字符串）。
-   在 `allowlist` 模式下，空的 `allowedUserIds` 列表将被视为配置错误，webhook 路由将不会启动（使用 `dmPolicy: "open"` 来允许所有用户）。
-   `dmPolicy: "open"` 允许任何发送者。
-   `dmPolicy: "disabled"` 阻止私信。
-   配对审批适用于：
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## 出站消息发送

使用数字形式的 Synology Chat 用户 ID 作为目标。示例：

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

支持通过基于 URL 的文件传递来发送媒体。

## 多账户

支持在 `channels.synology-chat.accounts` 下配置多个 Synology Chat 账户。每个账户可以覆盖令牌、入站 URL、webhook 路径、私信策略和限制。

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## 安全注意事项

-   保持 `token` 的机密性，如果泄露请进行轮换。
-   保持 `allowInsecureSsl: false`，除非您明确信任本地 NAS 的自签名证书。
-   入站 webhook 请求会进行令牌验证，并按发送者进行速率限制。
-   生产环境建议使用 `dmPolicy: "allowlist"`。

[Signal](./signal.md)[Slack](./slack.md)

---