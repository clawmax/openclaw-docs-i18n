

  消息平台

  
# Mattermost

状态：通过插件支持（机器人令牌 + WebSocket 事件）。支持频道、群组和私信。Mattermost 是一个可自托管的团队消息平台；有关产品详情和下载，请访问官方网站 [mattermost.com](https://mattermost.com)。

## 需要插件

Mattermost 以插件形式提供，不包含在核心安装包中。通过 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/mattermost
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/mattermost
```

如果在配置/引导过程中选择 Mattermost 并检测到 git 检出，OpenClaw 将自动提供本地安装路径。详情：[插件](../tools/plugin.md)

## 快速设置

1.  安装 Mattermost 插件。
2.  创建一个 Mattermost 机器人账户并复制**机器人令牌**。
3.  复制 Mattermost **基础 URL**（例如，`https://chat.example.com`）。
4.  配置 OpenClaw 并启动网关。

最小配置：

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## 原生斜杠命令

原生斜杠命令是可选功能。启用后，OpenClaw 通过 Mattermost API 注册 `oc_*` 斜杠命令，并在网关 HTTP 服务器上接收回调 POST 请求。

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // 当 Mattermost 无法直接访问网关时使用（反向代理/公共 URL）。
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

注意：

-   `native: "auto"` 在 Mattermost 中默认为禁用。设置 `native: true` 以启用。
-   如果省略 `callbackUrl`，OpenClaw 将从网关主机/端口 + `callbackPath` 推导出一个。
-   对于多账户设置，`commands` 可以在顶层设置，也可以在 `channels.mattermost.accounts..commands` 下设置（账户值会覆盖顶层字段）。
-   命令回调使用每个命令的令牌进行验证，令牌检查失败时会严格拒绝。
-   可达性要求：回调端点必须能从 Mattermost 服务器访问。
    -   除非 Mattermost 与 OpenClaw 运行在同一主机/网络命名空间，否则不要将 `callbackUrl` 设置为 `localhost`。
    -   除非该 URL 将 `/api/channels/mattermost/command` 反向代理到 OpenClaw，否则不要将 `callbackUrl` 设置为你的 Mattermost 基础 URL。
    -   快速检查：`curl https://<gateway-host>/api/channels/mattermost/command`；GET 请求应返回 OpenClaw 的 `405 Method Not Allowed`，而不是 `404`。
-   Mattermost 出口允许列表要求：
    -   如果你的回调目标是私有/tailnet/内部地址，请设置 Mattermost 的 `ServiceSettings.AllowedUntrustedInternalConnections` 以包含回调主机/域名。
    -   使用主机/域名条目，而不是完整 URL。
        -   正确：`gateway.tailnet-name.ts.net`
        -   错误：`https://gateway.tailnet-name.ts.net`

## 环境变量（默认账户）

如果更喜欢使用环境变量，请在网关主机上设置这些：

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

环境变量仅适用于**默认**账户（`default`）。其他账户必须使用配置值。

## 聊天模式

Mattermost 会自动响应私信。频道行为由 `chatmode` 控制：

-   `oncall`（默认）：仅在频道中被 @提及 时响应。
-   `onmessage`：响应每条频道消息。
-   `onchar`：当消息以触发前缀开头时响应。

配置示例：

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

注意：

-   `onchar` 模式仍会响应显式的 @提及。
-   对于旧版配置，`channels.mattermost.requireMention` 仍然有效，但推荐使用 `chatmode`。

## 访问控制（私信）

-   默认：`channels.mattermost.dmPolicy = "pairing"`（未知发件人会收到配对码）。
-   通过以下方式批准：
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   开放私信：`channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`。

## 频道（群组）

-   默认：`channels.mattermost.groupPolicy = "allowlist"`（提及门控）。
-   使用 `channels.mattermost.groupAllowFrom` 设置允许的发件人列表（推荐使用用户 ID）。
-   `@用户名` 匹配是可变的，仅在 `channels.mattermost.dangerouslyAllowNameMatching: true` 时启用。
-   开放频道：`channels.mattermost.groupPolicy="open"`（提及门控）。
-   运行时注意：如果 `channels.mattermost` 完全缺失，运行时将回退到 `groupPolicy="allowlist"` 进行群组检查（即使设置了 `channels.defaults.groupPolicy`）。

## 出站投递目标

使用以下目标格式与 `openclaw message send` 或 cron/webhooks：

-   `channel:` 用于频道
-   `user:` 用于私信
-   `@username` 用于私信（通过 Mattermost API 解析）

裸 ID 被视为频道。

## 反应（消息工具）

-   使用 `message action=react` 并设置 `channel=mattermost`。
-   `messageId` 是 Mattermost 帖子 ID。
-   `emoji` 接受名称如 `thumbsup` 或 `:+1:`（冒号可选）。
-   设置 `remove=true`（布尔值）以移除反应。
-   反应添加/移除事件作为系统事件转发到路由的代理会话。

示例：

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

配置：

-   `channels.mattermost.actions.reactions`：启用/禁用反应操作（默认为 true）。
-   每账户覆盖：`channels.mattermost.accounts..actions.reactions`。

## 交互式按钮（消息工具）

发送带有可点击按钮的消息。当用户点击按钮时，代理会收到选择并可以响应。通过将 `inlineButtons` 添加到频道能力中来启用按钮：

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

使用 `message action=send` 并附带 `buttons` 参数。按钮是一个二维数组（按钮行）：

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Yes","callback_data":"yes"},{"text":"No","callback_data":"no"}]]
```

按钮字段：

-   `text`（必需）：显示标签。
-   `callback_data`（必需）：点击时发送回的值（用作操作 ID）。
-   `style`（可选）：`"default"`、`"primary"` 或 `"danger"`。

当用户点击按钮时：

1.  所有按钮被替换为一行确认信息（例如，“✓ **Yes** selected by @user”）。
2.  代理将选择作为入站消息接收并响应。

注意：

-   按钮回调使用 HMAC-SHA256 验证（自动，无需配置）。
-   Mattermost 会从其 API 响应中剥离回调数据（安全特性），因此所有按钮在点击时都会被移除——无法部分移除。
-   包含连字符或下划线的操作 ID 会自动清理（Mattermost 路由限制）。

配置：

-   `channels.mattermost.capabilities`：能力字符串数组。添加 `"inlineButtons"` 以在代理系统提示中启用按钮工具描述。
-   `channels.mattermost.interactions.callbackBaseUrl`：按钮回调的可选外部基础 URL（例如 `https://gateway.example.com`）。当 Mattermost 无法直接访问其绑定主机上的网关时使用此选项。
-   在多账户设置中，你也可以在 `channels.mattermost.accounts..interactions.callbackBaseUrl` 下设置相同的字段。
-   如果省略 `interactions.callbackBaseUrl`，OpenClaw 将从 `gateway.customBindHost` + `gateway.port` 推导回调 URL，然后回退到 `http://localhost:`。
-   可达性规则：按钮回调 URL 必须能从 Mattermost 服务器访问。`localhost` 仅在 Mattermost 和 OpenClaw 运行在同一主机/网络命名空间时有效。
-   如果你的回调目标是私有/tailnet/内部地址，请将其主机/域名添加到 Mattermost 的 `ServiceSettings.AllowedUntrustedInternalConnections` 中。

### 直接 API 集成（外部脚本）

外部脚本和 webhooks 可以直接通过 Mattermost REST API 发布按钮，而不是通过代理的 `message` 工具。尽可能使用扩展中的 `buildButtonAttachments()`；如果发布原始 JSON，请遵循以下规则：**负载结构：**

```json
{
  channel_id: "<channelId>",
  message: "Choose an option:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // 仅限字母数字——见下文
            type: "button", // 必需，否则点击会被静默忽略
            name: "Approve", // 显示标签
            style: "primary", // 可选："default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // 必须与按钮 id 匹配（用于名称查找）
                action: "approve",
                // ... 任何自定义字段 ...
                _token: "<hmac>", // 见下面的 HMAC 部分
              },
            },
          },
        ],
      },
    ],
  },
}
```

**关键规则：**

1.  附件放在 `props.attachments` 中，而不是顶层的 `attachments`（会被静默忽略）。
2.  每个操作都需要 `type: "button"`——没有它，点击会被静默吞掉。
3.  每个操作都需要一个 `id` 字段——Mattermost 会忽略没有 ID 的操作。
4.  操作 `id` 必须**仅包含字母数字**（`[a-zA-Z0-9]`）。连字符和下划线会破坏 Mattermost 服务器端的操作路由（返回 404）。使用前请移除它们。
5.  `context.action_id` 必须与按钮的 `id` 匹配，以便确认消息显示按钮名称（例如，“Approve”）而不是原始 ID。
6.  `context.action_id` 是必需的——没有它，交互处理器会返回 400。

**HMAC 令牌生成：** 网关使用 HMAC-SHA256 验证按钮点击。外部脚本必须生成与网关验证逻辑匹配的令牌：

1.  从机器人令牌派生密钥：`HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  构建上下文对象，包含**除** `_token` 外的所有字段。
3.  使用**排序后的键**和**无空格**进行序列化（网关使用带排序键的 `JSON.stringify`，产生紧凑输出）。
4.  签名：`HMAC-SHA256(key=secret, data=serializedContext)`
5.  将生成的十六进制摘要作为 `_token` 添加到上下文中。

Python 示例：

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

常见的 HMAC 陷阱：

-   Python 的 `json.dumps` 默认添加空格（`{"key": "val"}`）。使用 `separators=(",", ":")` 以匹配 JavaScript 的紧凑输出（`{"key":"val"}`）。
-   始终签名**所有**上下文字段（减去 `_token`）。网关会剥离 `_token`，然后对剩余的所有内容进行签名。签名子集会导致静默验证失败。
-   使用 `sort_keys=True`——网关在签名前对键进行排序，并且 Mattermost 在存储负载时可能会重新排序上下文字段。
-   从机器人令牌（确定性）派生密钥，而不是随机字节。在创建按钮的进程和进行验证的网关之间，密钥必须相同。

## 目录适配器

Mattermost 插件包含一个目录适配器，可通过 Mattermost API 解析频道和用户名。这使得在 `openclaw message send` 和 cron/webhook 投递中可以使用 `#channel-name` 和 `@username` 目标。无需配置——适配器使用账户配置中的机器人令牌。

## 多账户

Mattermost 支持在 `channels.mattermost.accounts` 下配置多个账户：

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## 故障排除

-   频道中无回复：确保机器人在频道中并提及它（oncall），使用触发前缀（onchar），或设置 `chatmode: "onmessage"`。
-   认证错误：检查机器人令牌、基础 URL 以及账户是否启用。
-   多账户问题：环境变量仅适用于 `default` 账户。
-   按钮显示为白框：代理可能发送了格式错误的按钮数据。检查每个按钮是否同时具有 `text` 和 `callback_data` 字段。
-   按钮渲染但点击无反应：验证 Mattermost 服务器配置中的 `AllowedUntrustedInternalConnections` 是否包含 `127.0.0.1 localhost`，以及 ServiceSettings 中的 `EnablePostActionIntegration` 是否为 `true`。
-   按钮点击返回 404：按钮 `id` 可能包含连字符或下划线。Mattermost 的操作路由器在非字母数字 ID 上会中断。仅使用 `[a-zA-Z0-9]`。
-   网关日志 `invalid _token`：HMAC 不匹配。检查你是否对所有上下文字段（不是子集）进行了签名，使用了排序的键，并使用了紧凑的 JSON（无空格）。参见上面的 HMAC 部分。
-   网关日志 `missing _token in context`：按钮的上下文中没有 `_token` 字段。确保在构建集成负载时包含它。
-   确认信息显示原始 ID 而不是按钮名称：`context.action_id` 与按钮的 `id` 不匹配。将两者设置为相同的清理后的值。
-   代理不知道按钮：将 `capabilities: ["inlineButtons"]` 添加到 Mattermost 频道配置中。

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)