

  消息平台

  
# Zalo

状态：实验性。支持私信；群组消息需配置群组策略后才能使用。

## 需要安装插件

Zalo 以插件形式提供，未包含在核心安装包中。

-   通过 CLI 安装：`openclaw plugins install @openclaw/zalo`
-   或在初始化引导时选择 **Zalo** 并确认安装
-   详细说明：[插件](../tools/plugin.md)

## 快速设置（新手友好）

1.  安装 Zalo 插件：
    -   从源码安装：`openclaw plugins install ./extensions/zalo`
    -   从 npm 安装（如已发布）：`openclaw plugins install @openclaw/zalo`
    -   或在初始化引导时选择 **Zalo** 并确认安装
2.  设置令牌：
    -   环境变量方式：`ZALO_BOT_TOKEN=...`
    -   配置文件方式：`channels.zalo.botToken: "..."`
3.  重启网关（或完成初始化引导）
4.  私信访问默认使用配对模式，首次联系机器人时需要批准配对码

最小配置示例：

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

## 这是什么？

Zalo 是一款主打越南市场的即时通讯应用。通过其 Bot API，网关可以运行一个机器人来处理一对一对话。如果你的场景需要将消息确定性路由回 Zalo（比如客服支持或通知推送），这个频道会很适合你。

核心特性：

-   网关拥有一个 Zalo Bot API 频道
-   确定性路由：回复会返回 Zalo，模型不会自行选择频道
-   私信共享智能体（agent）的主会话
-   支持群组消息，但需要配置策略（`groupPolicy` + `groupAllowFrom`），默认采用安全的允许列表模式

## 设置步骤（快速通道）

### 1) 创建机器人令牌（Zalo Bot Platform）

1.  访问 [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) 并登录
2.  创建一个新机器人并完成配置
3.  复制机器人令牌（格式：`12345689:abc-xyz`）

### 2) 配置令牌（环境变量或配置文件）

配置示例：

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

环境变量方式：`ZALO_BOT_TOKEN=...`（仅适用于默认账户）。如需支持多账户，请使用 `channels.zalo.accounts` 配置每个账户的令牌和可选的 `name`。

3.  重启网关。当令牌被正确识别（通过环境变量或配置文件）后，Zalo 频道会自动启动
4.  私信访问默认使用配对模式。当机器人首次被联系时，请批准配对码

## 工作原理

-   入站消息会被标准化为带有媒体占位符的统一频道信封格式
-   回复始终路由回同一个 Zalo 聊天
-   默认使用长轮询模式；也可以通过 `channels.zalo.webhookUrl` 启用 Webhook 模式

## 限制说明

-   出站文本会被分块为 2000 字符（Zalo API 限制）
-   媒体下载/上传受 `channels.zalo.mediaMaxMb` 限制（默认 5MB）
-   流式传输默认被禁用，因为 2000 字符的限制使其效果有限

## 访问控制（私信）

### 私信访问策略

-   默认配置：`channels.zalo.dmPolicy = "pairing"`。未知发件人会收到一个配对码，在批准前消息会被忽略（配对码 1 小时后过期）
-   批准方式：
    -   `openclaw pairing list zalo`
    -   `openclaw pairing approve zalo `
-   配对是默认的令牌交换方式。详情请参阅：[配对](./pairing.md)
-   `channels.zalo.allowFrom` 接受数字用户 ID（暂不支持用户名查找）

## 访问控制（群组）

-   `channels.zalo.groupPolicy` 控制群组入站消息处理方式，可选值：`open | allowlist | disabled`
-   默认行为是安全优先：`allowlist`
-   `channels.zalo.groupAllowFrom` 限制哪些发件人 ID 可以在群组中触发机器人
-   如果未设置 `groupAllowFrom`，Zalo 会回退到 `allowFrom` 进行发件人检查
-   `groupPolicy: "disabled"` 会阻止所有群组消息
-   `groupPolicy: "open"` 允许任何群组成员触发（需要 @提及机器人）
-   注意：如果配置中完全缺少 `channels.zalo`，运行时仍会出于安全考虑回退到 `groupPolicy="allowlist"`

## 长轮询 vs Webhook

-   默认：长轮询（无需公开 URL）
-   Webhook 模式：设置 `channels.zalo.webhookUrl` 和 `channels.zalo.webhookSecret`
    -   Webhook 密钥长度必须为 8-256 个字符
    -   Webhook URL 必须使用 HTTPS
    -   Zalo 发送事件时会附带 `X-Bot-Api-Secret-Token` 请求头用于验证
    -   网关 HTTP 服务在 `channels.zalo.webhookPath` 处理 Webhook 请求（默认为 Webhook URL 的路径）
    -   请求必须使用 `Content-Type: application/json`（或 `+json` 媒体类型）
    -   重复事件（`event_name + message_id`）在短时间内会被忽略
    -   突发流量会根据路径/来源进行速率限制，可能返回 HTTP 429

**注意：** 根据 Zalo API 文档，getUpdates（轮询）和 Webhook 是互斥的，不能同时使用。

## 支持的消息类型

-   **文本消息**：完全支持，自动进行 2000 字符分块
-   **图片消息**：下载并处理入站图片；通过 `sendPhoto` 发送图片
-   **贴纸**：会被记录但不会完全处理（智能体不会回复）
-   **不支持的类型**：会被记录（例如来自受保护用户的消息）

## 功能支持一览

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 群组 | ⚠️ 支持，需配合策略控制（默认允许列表） |
| 媒体（图片） | ✅ 支持 |
| 消息回应 | ❌ 不支持 |
| 主题 | ❌ 不支持 |
| 投票 | ❌ 不支持 |
| 原生命令 | ❌ 不支持 |
| 流式传输 | ⚠️ 被禁用（2000 字符限制） |

## 发送目标（CLI/定时任务）

-   使用聊天 ID 作为目标
-   示例：`openclaw message send --channel zalo --target 123456789 --message "hi"`

## 故障排除

**机器人无响应：**

-   检查令牌是否有效：`openclaw channels status --probe`
-   验证发件人是否已获批准（配对或 allowFrom）
-   检查网关日志：`openclaw logs --follow`

**Webhook 未收到事件：**

-   确保 Webhook URL 使用 HTTPS
-   验证密钥令牌长度为 8-256 个字符
-   确认网关 HTTP 端点在配置的路径上可访问
-   检查 getUpdates 轮询是否未在运行（两者互斥）

## 配置参考（Zalo）

完整配置请参阅：[配置](../gateway/configuration.md)

提供者选项：

-   `channels.zalo.enabled`：启用/禁用频道启动
-   `channels.zalo.botToken`：来自 Zalo Bot Platform 的机器人令牌
-   `channels.zalo.tokenFile`：从文件路径读取令牌
-   `channels.zalo.dmPolicy`：私信策略，可选值 `pairing | allowlist | open | disabled`（默认：pairing）
-   `channels.zalo.allowFrom`：私信允许列表（用户 ID）。`open` 策略需要设置为 `"*"`。向导会要求输入数字 ID
-   `channels.zalo.groupPolicy`：群组策略，可选值 `open | allowlist | disabled`（默认：allowlist）
-   `channels.zalo.groupAllowFrom`：群组发件人允许列表（用户 ID）。未设置时回退到 `allowFrom`
-   `channels.zalo.mediaMaxMb`：入站/出站媒体上限（MB，默认 5）
-   `channels.zalo.webhookUrl`：启用 Webhook 模式（需要 HTTPS）
-   `channels.zalo.webhookSecret`：Webhook 密钥（8-256 字符）
-   `channels.zalo.webhookPath`：网关 HTTP 服务器上的 Webhook 路径
-   `channels.zalo.proxy`：API 请求的代理 URL

多账户选项：

-   `channels.zalo.accounts..botToken`：每个账户的令牌
-   `channels.zalo.accounts..tokenFile`：每个账户的令牌文件
-   `channels.zalo.accounts..name`：显示名称
-   `channels.zalo.accounts..enabled`：启用/禁用账户
-   `channels.zalo.accounts..dmPolicy`：每个账户的私信策略
-   `channels.zalo.accounts..allowFrom`：每个账户的允许列表
-   `channels.zalo.accounts..groupPolicy`：每个账户的群组策略
-   `channels.zalo.accounts..groupAllowFrom`：每个账户的群组发件人允许列表
-   `channels.zalo.accounts..webhookUrl`：每个账户的 Webhook URL
-   `channels.zalo.accounts..webhookSecret`：每个账户的 Webhook 密钥
-   `channels.zalo.accounts..webhookPath`：每个账户的 Webhook 路径
-   `channels.zalo.accounts..proxy`：每个账户的代理 URL

[WhatsApp](./whatsapp.md)[Zalo Personal](./zalouser.md)