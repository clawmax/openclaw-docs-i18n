

  消息平台

  
# Zalo

状态：实验性。支持私信；群组处理需配合明确的群组策略控制。

## 需要插件

Zalo 作为插件提供，不包含在核心安装包中。

-   通过 CLI 安装：`openclaw plugins install @openclaw/zalo`
-   或在初始化引导过程中选择 **Zalo** 并确认安装提示
-   详情：[插件](../tools/plugin.md)

## 快速设置（新手）

1.  安装 Zalo 插件：
    -   从源码检出：`openclaw plugins install ./extensions/zalo`
    -   从 npm（如果已发布）：`openclaw plugins install @openclaw/zalo`
    -   或在初始化引导中选择 **Zalo** 并确认安装提示
2.  设置令牌：
    -   环境变量：`ZALO_BOT_TOKEN=...`
    -   或配置文件：`channels.zalo.botToken: "..."`。
3.  重启网关（或完成初始化引导）。
4.  私信访问默认采用配对方式；首次联系时批准配对码。

最小配置：

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## 这是什么

Zalo 是一款专注于越南市场的消息应用；其 Bot API 允许网关运行一个机器人进行一对一对话。它非常适合需要确定性路由回 Zalo 的支持或通知场景。

-   一个由网关拥有的 Zalo Bot API 频道。
-   确定性路由：回复会返回 Zalo；模型从不选择频道。
-   私信共享代理的主会话。
-   支持群组，但需配合策略控制（`groupPolicy` + `groupAllowFrom`），默认采用故障关闭的允许列表行为。

## 设置（快速路径）

### 1) 创建机器人令牌（Zalo Bot Platform）

1.  访问 [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) 并登录。
2.  创建一个新的机器人并配置其设置。
3.  复制机器人令牌（格式：`12345689:abc-xyz`）。

### 2) 配置令牌（环境变量或配置文件）

示例：

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

环境变量选项：`ZALO_BOT_TOKEN=...`（仅适用于默认账户）。多账户支持：使用 `channels.zalo.accounts` 并配置每个账户的令牌和可选的 `name`。

3.  重启网关。当令牌被解析（通过环境变量或配置）时，Zalo 将启动。
4.  私信访问默认为配对。当机器人首次被联系时，请批准配对码。

## 工作原理（行为）

-   入站消息被标准化为带有媒体占位符的共享频道信封。
-   回复总是路由回同一个 Zalo 聊天。
-   默认使用长轮询；可通过 `channels.zalo.webhookUrl` 启用 Webhook 模式。

## 限制

-   出站文本被分块为 2000 个字符（Zalo API 限制）。
-   媒体下载/上传受 `channels.zalo.mediaMaxMb` 限制（默认 5）。
-   流式传输默认被阻止，因为 2000 字符的限制使其用处不大。

## 访问控制（私信）

### 私信访问

-   默认：`channels.zalo.dmPolicy = "pairing"`。未知发件人会收到一个配对码；在批准前消息将被忽略（配对码在 1 小时后过期）。
-   批准方式：
    -   `openclaw pairing list zalo`
    -   `openclaw pairing approve zalo `
-   配对是默认的令牌交换方式。详情：[配对](./pairing.md)
-   `channels.zalo.allowFrom` 接受数字用户 ID（不支持用户名查找）。

## 访问控制（群组）

-   `channels.zalo.groupPolicy` 控制群组入站消息处理：`open | allowlist | disabled`。
-   默认行为是故障关闭：`allowlist`。
-   `channels.zalo.groupAllowFrom` 限制哪些发件人 ID 可以在群组中触发机器人。
-   如果未设置 `groupAllowFrom`，Zalo 将回退到 `allowFrom` 进行发件人检查。
-   `groupPolicy: "disabled"` 阻止所有群组消息。
-   `groupPolicy: "open"` 允许任何群组成员（需提及机器人）。
-   运行时注意：如果配置中完全缺少 `channels.zalo`，运行时仍会出于安全考虑回退到 `groupPolicy="allowlist"`。

## 长轮询 vs Webhook

-   默认：长轮询（无需公开 URL）。
-   Webhook 模式：设置 `channels.zalo.webhookUrl` 和 `channels.zalo.webhookSecret`。
    -   Webhook 密钥必须为 8-256 个字符。
    -   Webhook URL 必须使用 HTTPS。
    -   Zalo 发送事件时会附带 `X-Bot-Api-Secret-Token` 请求头用于验证。
    -   网关 HTTP 服务在 `channels.zalo.webhookPath` 处理 Webhook 请求（默认为 Webhook URL 的路径）。
    -   请求必须使用 `Content-Type: application/json`（或 `+json` 媒体类型）。
    -   重复事件（`event_name + message_id`）在短时间内会被忽略。
    -   突发流量会根据路径/来源进行速率限制，并可能返回 HTTP 429。

**注意：** 根据 Zalo API 文档，getUpdates（轮询）和 Webhook 是互斥的。

## 支持的消息类型

-   **文本消息**：完全支持，支持 2000 字符分块。
-   **图片消息**：下载并处理入站图片；通过 `sendPhoto` 发送图片。
-   **贴纸**：会被记录但不会完全处理（无代理回复）。
-   **不支持的类型**：会被记录（例如，来自受保护用户的消息）。

## 功能支持

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 群组 | ⚠️ 支持，需配合策略控制（默认允许列表） |
| 媒体（图片） | ✅ 支持 |
| 消息回应 | ❌ 不支持 |
| 主题 | ❌ 不支持 |
| 投票 | ❌ 不支持 |
| 原生命令 | ❌ 不支持 |
| 流式传输 | ⚠️ 被阻止（2000 字符限制） |

## 发送目标（CLI/定时任务）

-   使用聊天 ID 作为目标。
-   示例：`openclaw message send --channel zalo --target 123456789 --message "hi"`。

## 故障排除

**机器人无响应：**

-   检查令牌是否有效：`openclaw channels status --probe`
-   验证发件人是否已获批准（配对或 allowFrom）
-   检查网关日志：`openclaw logs --follow`

**Webhook 未收到事件：**

-   确保 Webhook URL 使用 HTTPS
-   验证密钥令牌长度为 8-256 个字符
-   确认网关 HTTP 端点可在配置的路径上访问
-   检查 getUpdates 轮询是否未在运行（它们是互斥的）

## 配置参考（Zalo）

完整配置：[配置](../gateway/configuration.md) 提供者选项：

-   `channels.zalo.enabled`：启用/禁用频道启动。
-   `channels.zalo.botToken`：来自 Zalo Bot Platform 的机器人令牌。
-   `channels.zalo.tokenFile`：从文件路径读取令牌。
-   `channels.zalo.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
-   `channels.zalo.allowFrom`：私信允许列表（用户 ID）。`open` 策略需要 `"*"`。向导会要求输入数字 ID。
-   `channels.zalo.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
-   `channels.zalo.groupAllowFrom`：群组发件人允许列表（用户 ID）。未设置时回退到 `allowFrom`。
-   `channels.zalo.mediaMaxMb`：入站/出站媒体上限（MB，默认 5）。
-   `channels.zalo.webhookUrl`：启用 Webhook 模式（需要 HTTPS）。
-   `channels.zalo.webhookSecret`：Webhook 密钥（8-256 字符）。
-   `channels.zalo.webhookPath`：网关 HTTP 服务器上的 Webhook 路径。
-   `channels.zalo.proxy`：API 请求的代理 URL。

多账户选项：

-   `channels.zalo.accounts..botToken`：每个账户的令牌。
-   `channels.zalo.accounts..tokenFile`：每个账户的令牌文件。
-   `channels.zalo.accounts..name`：显示名称。
-   `channels.zalo.accounts..enabled`：启用/禁用账户。
-   `channels.zalo.accounts..dmPolicy`：每个账户的私信策略。
-   `channels.zalo.accounts..allowFrom`：每个账户的允许列表。
-   `channels.zalo.accounts..groupPolicy`：每个账户的群组策略。
-   `channels.zalo.accounts..groupAllowFrom`：每个账户的群组发件人允许列表。
-   `channels.zalo.accounts..webhookUrl`：每个账户的 Webhook URL。
-   `channels.zalo.accounts..webhookSecret`：每个账户的 Webhook 密钥。
-   `channels.zalo.accounts..webhookPath`：每个账户的 Webhook 路径。
-   `channels.zalo.accounts..proxy`：每个账户的代理 URL。

[WhatsApp](./whatsapp.md)[Zalo Personal](./zalouser.md)