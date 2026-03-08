

  消息平台

  
# BlueBubbles

状态：捆绑插件，通过 HTTP 与 BlueBubbles macOS 服务器通信。**推荐用于 iMessage 集成**，因其 API 更丰富且设置比传统的 imsg 通道更简单。

## 概述

-   通过 BlueBubbles 助手应用 ([bluebubbles.app](https://bluebubbles.app)) 在 macOS 上运行。
-   推荐/测试版本：macOS Sequoia (15)。macOS Tahoe (26) 可用；编辑功能目前在 Tahoe 上已损坏，群组图标更新可能报告成功但未同步。
-   OpenClaw 通过其 REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`) 与之通信。
-   传入消息通过 webhook 到达；外发回复、输入指示器、已读回执和轻触反馈（tapback）通过 REST 调用实现。
-   附件和贴纸作为入站媒体被接收（并在可能时呈现给代理）。
-   配对/允许列表的工作方式与其他通道相同 (`/channels/pairing` 等)，使用 `channels.bluebubbles.allowFrom` + 配对码。
-   反应（Reactions）像 Slack/Telegram 一样作为系统事件呈现，因此代理可以在回复前“提及”它们。
-   高级功能：编辑、取消发送、回复线程、消息效果、群组管理。

## 快速开始

1.  在您的 Mac 上安装 BlueBubbles 服务器（按照 [bluebubbles.app/install](https://bluebubbles.app/install) 的说明操作）。
2.  在 BlueBubbles 配置中，启用 Web API 并设置密码。
3.  运行 `openclaw onboard` 并选择 BlueBubbles，或手动配置：
    
    复制
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  将 BlueBubbles webhook 指向您的网关（例如：`https://your-gateway-host:3000/bluebubbles-webhook?password=`）。
5.  启动网关；它将注册 webhook 处理程序并开始配对。

安全说明：

-   始终设置 webhook 密码。
-   Webhook 身份验证始终是必需的。除非包含与 `channels.bluebubbles.password` 匹配的密码/guid（例如 `?password=` 或 `x-password`），否则 OpenClaw 会拒绝 BlueBubbles webhook 请求，无论回环/代理拓扑如何。
-   密码身份验证在读取/解析完整的 webhook 主体之前进行检查。

## 保持 Messages.app 活跃（虚拟机 / 无头设置）

某些 macOS 虚拟机 / 常开设置可能导致 Messages.app 进入“空闲”状态（传入事件停止，直到应用被打开/置于前台）。一个简单的解决方法是**每 5 分钟使用 AppleScript + LaunchAgent 唤醒 Messages**。

### 1) 保存 AppleScript

保存为：

-   `~/Scripts/poke-messages.scpt`

示例脚本（非交互式；不会抢占焦点）：

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- 触摸脚本接口以保持进程响应。
    set _chatCount to (count of chats)
  end tell
on error
  -- 忽略瞬时故障（首次运行提示、会话锁定等）。
end try
```

### 2) 安装 LaunchAgent

保存为：

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

注意：

-   此任务**每 300 秒**运行一次，并在**登录时**运行。
-   首次运行可能触发 macOS **自动化**提示 (`osascript` → Messages)。在运行 LaunchAgent 的同一用户会话中批准它们。

加载它：

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## 入门引导

BlueBubbles 在交互式设置向导中可用：

```bash
openclaw onboard
```

向导会提示输入：

-   **服务器 URL**（必需）：BlueBubbles 服务器地址（例如，`http://192.168.1.100:1234`）
-   **密码**（必需）：来自 BlueBubbles 服务器设置的 API 密码
-   **Webhook 路径**（可选）：默认为 `/bluebubbles-webhook`
-   **私信策略**：配对、允许列表、开放或禁用
-   **允许列表**：电话号码、电子邮件或聊天目标

您也可以通过 CLI 添加 BlueBubbles：

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## 访问控制（私信 + 群组）

私信：

-   默认：`channels.bluebubbles.dmPolicy = "pairing"`。
-   未知发件人会收到配对码；在批准前消息将被忽略（配对码在 1 小时后过期）。
-   通过以下方式批准：
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   配对是默认的令牌交换方式。详情：[配对](./pairing.md)

群组：

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled`（默认：`allowlist`）。
-   当设置为 `allowlist` 时，`channels.bluebubbles.groupAllowFrom` 控制谁可以在群组中触发。

### 提及门控（群组）

BlueBubbles 支持群组聊天的提及门控，匹配 iMessage/WhatsApp 行为：

-   使用 `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）来检测提及。
-   当为群组启用 `requireMention` 时，代理仅在提及时才响应。
-   来自授权发件人的控制命令绕过提及门控。

每群组配置：

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // 所有群组的默认设置
        "iMessage;-;chat123": { requireMention: false }, // 特定群组的覆盖设置
      },
    },
  },
}
```

### 命令门控

-   控制命令（例如，`/config`、`/model`）需要授权。
-   使用 `allowFrom` 和 `groupAllowFrom` 来确定命令授权。
-   授权发件人即使未在群组中被提及也可以运行控制命令。

## 输入指示 + 已读回执

-   **输入指示器**：在响应生成之前和期间自动发送。
-   **已读回执**：由 `channels.bluebubbles.sendReadReceipts` 控制（默认：`true`）。
-   **输入指示器**：OpenClaw 发送输入开始事件；BlueBubbles 在发送或超时后自动清除输入（通过 DELETE 手动停止不可靠）。

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // 禁用已读回执
    },
  },
}
```

## 高级操作

在配置中启用时，BlueBubbles 支持高级消息操作：

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // 轻触反馈（默认：true）
        edit: true, // 编辑已发送的消息（macOS 13+，在 macOS 26 Tahoe 上已损坏）
        unsend: true, // 取消发送消息（macOS 13+）
        reply: true, // 通过消息 GUID 进行回复线程
        sendWithEffect: true, // 消息效果（猛烈、大声等）
        renameGroup: true, // 重命名群组聊天
        setGroupIcon: true, // 设置群组聊天图标/照片（在 macOS 26 Tahoe 上不稳定）
        addParticipant: true, // 向群组添加参与者
        removeParticipant: true, // 从群组移除参与者
        leaveGroup: true, // 离开群组聊天
        sendAttachment: true, // 发送附件/媒体
      },
    },
  },
}
```

可用操作：

-   **react**：添加/移除轻触反馈反应 (`messageId`, `emoji`, `remove`)
-   **edit**：编辑已发送的消息 (`messageId`, `text`)
-   **unsend**：取消发送消息 (`messageId`)
-   **reply**：回复特定消息 (`messageId`, `text`, `to`)
-   **sendWithEffect**：使用 iMessage 效果发送 (`text`, `to`, `effectId`)
-   **renameGroup**：重命名群组聊天 (`chatGuid`, `displayName`)
-   **setGroupIcon**：设置群组聊天的图标/照片 (`chatGuid`, `media`) — 在 macOS 26 Tahoe 上不稳定（API 可能返回成功但图标未同步）。
-   **addParticipant**：将某人添加到群组 (`chatGuid`, `address`)
-   **removeParticipant**：从群组移除某人 (`chatGuid`, `address`)
-   **leaveGroup**：离开群组聊天 (`chatGuid`)
-   **sendAttachment**：发送媒体/文件 (`to`, `buffer`, `filename`, `asVoice`)
    -   语音备忘录：设置 `asVoice: true` 并使用 **MP3** 或 **CAF** 音频以作为 iMessage 语音消息发送。BlueBubbles 在发送语音备忘录时将 MP3 转换为 CAF。

### 消息 ID（短 ID 与完整 ID）

OpenClaw 可能呈现*短*消息 ID（例如，`1`、`2`）以节省令牌。

-   `MessageSid` / `ReplyToId` 可以是短 ID。
-   `MessageSidFull` / `ReplyToIdFull` 包含提供商的完整 ID。
-   短 ID 是内存中的；它们可能在重启或缓存驱逐后过期。
-   操作接受短 ID 或完整 `messageId`，但如果短 ID 不再可用则会报错。

为持久性自动化和存储使用完整 ID：

-   模板：`{{MessageSidFull}}`、`{{ReplyToIdFull}}`
-   上下文：入站负载中的 `MessageSidFull` / `ReplyToIdFull`

有关模板变量，请参阅[配置](../gateway/configuration.md)。

## 分块流式传输

控制响应是作为单条消息发送还是分块流式传输：

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // 启用分块流式传输（默认关闭）
    },
  },
}
```

## 媒体 + 限制

-   入站附件被下载并存储在媒体缓存中。
-   通过 `channels.bluebubbles.mediaMaxMb` 限制入站和出站媒体的容量（默认：8 MB）。
-   出站文本被分块到 `channels.bluebubbles.textChunkLimit`（默认：4000 字符）。

## 配置参考

完整配置：[配置](../gateway/configuration.md) 提供者选项：

-   `channels.bluebubbles.enabled`：启用/禁用通道。
-   `channels.bluebubbles.serverUrl`：BlueBubbles REST API 基础 URL。
-   `channels.bluebubbles.password`：API 密码。
-   `channels.bluebubbles.webhookPath`：Webhook 端点路径（默认：`/bluebubbles-webhook`）。
-   `channels.bluebubbles.dmPolicy`：`pairing | allowlist | open | disabled`（默认：`pairing`）。
-   `channels.bluebubbles.allowFrom`：私信允许列表（句柄、电子邮件、E.164 号码、`chat_id:*`、`chat_guid:*`）。
-   `channels.bluebubbles.groupPolicy`：`open | allowlist | disabled`（默认：`allowlist`）。
-   `channels.bluebubbles.groupAllowFrom`：群组发件人允许列表。
-   `channels.bluebubbles.groups`：每群组配置（`requireMention` 等）。
-   `channels.bluebubbles.sendReadReceipts`：发送已读回执（默认：`true`）。
-   `channels.bluebubbles.blockStreaming`：启用分块流式传输（默认：`false`；流式回复需要）。
-   `channels.bluebubbles.textChunkLimit`：出站分块大小（字符数，默认：4000）。
-   `channels.bluebubbles.chunkMode`：`length`（默认）仅在超过 `textChunkLimit` 时分割；`newline` 在长度分块之前按空行（段落边界）分割。
-   `channels.bluebubbles.mediaMaxMb`：入站/出站媒体容量限制（MB，默认：8）。
-   `channels.bluebubbles.mediaLocalRoots`：允许用于出站本地媒体路径的绝对本地目录的显式允许列表。默认情况下，除非配置此项，否则本地路径发送将被拒绝。每账户覆盖：`channels.bluebubbles.accounts..mediaLocalRoots`。
-   `channels.bluebubbles.historyLimit`：上下文的最大群组消息数（0 表示禁用）。
-   `channels.bluebubbles.dmHistoryLimit`：私信历史限制。
-   `channels.bluebubbles.actions`：启用/禁用特定操作。
-   `channels.bluebubbles.accounts`：多账户配置。

相关全局选项：

-   `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）。
-   `messages.responsePrefix`。

## 寻址 / 投递目标

优先使用 `chat_guid` 进行稳定路由：

-   `chat_guid:iMessage;-;+15555550123`（群组首选）
-   `chat_id:123`
-   `chat_identifier:...`
-   直接句柄：`+15555550123`、`user@example.com`
    -   如果直接句柄没有现有的私信聊天，OpenClaw 将通过 `POST /api/v1/chat/new` 创建一个。这需要启用 BlueBubbles 私有 API。

## 安全

-   Webhook 请求通过比较 `guid`/`password` 查询参数或请求头与 `channels.bluebubbles.password` 进行身份验证。来自 `localhost` 的请求也被接受。
-   保持 API 密码和 webhook 端点机密（将它们视为凭据）。
-   Localhost 信任意味着同一主机上的反向代理可能无意中绕过密码。如果您代理网关，请在代理处要求身份验证并配置 `gateway.trustedProxies`。请参阅[网关安全](../gateway/security.md#reverse-proxy-configuration)。
-   如果在 LAN 外部暴露 BlueBubbles 服务器，请启用 HTTPS + 防火墙规则。

## 故障排除

-   如果输入/已读事件停止工作，请检查 BlueBubbles webhook 日志并验证网关路径是否与 `channels.bluebubbles.webhookPath` 匹配。
-   配对码在一小时后过期；使用 `openclaw pairing list bluebubbles` 和 `openclaw pairing approve bluebubbles `。
-   反应需要 BlueBubbles 私有 API (`POST /api/v1/message/react`)；确保服务器版本暴露了它。
-   编辑/取消发送需要 macOS 13+ 和兼容的 BlueBubbles 服务器版本。在 macOS 26 (Tahoe) 上，由于私有 API 更改，编辑功能目前损坏。
-   群组图标更新在 macOS 26 (Tahoe) 上可能不稳定：API 可能返回成功但新图标未同步。
-   OpenClaw 根据 BlueBubbles 服务器的 macOS 版本自动隐藏已知损坏的操作。如果编辑功能在 macOS 26 (Tahoe) 上仍然出现，请使用 `channels.bluebubbles.actions.edit=false` 手动禁用它。
-   状态/健康信息：`openclaw status --all` 或 `openclaw status --deep`。

有关通用通道工作流参考，请参阅[通道](../channels.md)和[插件](../tools/plugin.md)指南。

[聊天通道](../channels.md)[Discord](./discord.md)