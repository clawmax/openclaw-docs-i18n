

  消息平台

  
# WhatsApp

状态：通过 WhatsApp Web (Baileys) 生产就绪。网关拥有已链接的会话。

## 快速设置

### 步骤 1：配置 WhatsApp 访问策略

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### 步骤 2：链接 WhatsApp (二维码)

```bash
openclaw channels login --channel whatsapp
```

对于特定账户：

```bash
openclaw channels login --channel whatsapp --account work
```

### 步骤 3：启动网关

```bash
openclaw gateway
```

### 步骤 4：批准首次配对请求（如果使用配对模式）

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

配对请求在 1 小时后过期。每个频道的待处理请求上限为 3 个。

 

> **ℹ️** OpenClaw 建议在可能的情况下在单独的号码上运行 WhatsApp。（频道元数据和入门流程已针对该设置进行了优化，但也支持个人号码设置。）

## 部署模式

这是最清晰的操作模式：

-   为 OpenClaw 使用独立的 WhatsApp 身份
-   更清晰的 DM 允许列表和路由边界
-   降低自聊混淆的可能性

最小策略模式：

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

入门流程支持个人号码模式，并写入一个对自聊友好的基线：

-   `dmPolicy: "allowlist"`
-   `allowFrom` 包含您的个人号码
-   `selfChatMode: true`

在运行时，自聊保护机制基于链接的自有号码和 `allowFrom` 触发。

在当前 OpenClaw 频道架构中，消息平台频道基于 WhatsApp Web (`Baileys`)。内置聊天频道注册表中没有单独的 Twilio WhatsApp 消息频道。

## 运行时模型

-   网关拥有 WhatsApp 套接字和重连循环。
-   出站发送需要目标账户的活跃 WhatsApp 监听器。
-   状态和广播聊天被忽略 (`@status`, `@broadcast`)。
-   直接聊天使用 DM 会话规则 (`session.dmScope`；默认 `main` 将 DM 合并到代理主会话)。
-   群组会话是隔离的 (`agent::whatsapp:group:`)。

## 访问控制与激活

`channels.whatsapp.dmPolicy` 控制直接聊天访问：

-   `pairing` (默认)
-   `allowlist`
-   `open` (要求 `allowFrom` 包含 `"*"`)
-   `disabled`

`allowFrom` 接受 E.164 格式的号码（内部会进行规范化）。多账户覆盖：`channels.whatsapp.accounts..dmPolicy` (以及 `allowFrom`) 对该账户优先于频道级默认值。运行时行为详情：

-   配对信息持久化在频道允许存储中，并与配置的 `allowFrom` 合并
-   如果未配置允许列表，默认允许链接的自有号码
-   出站 `fromMe` DM 永远不会自动配对

群组访问有两层：

1.  **群组成员允许列表** (`channels.whatsapp.groups`)
    -   如果省略 `groups`，则所有群组都符合条件
    -   如果存在 `groups`，则它充当群组允许列表 (`"*"` 被允许)
2.  **群组发送者策略** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
    -   `open`: 绕过发送者允许列表
    -   `allowlist`: 发送者必须匹配 `groupAllowFrom` (或 `*`)
    -   `disabled`: 阻止所有群组入站

发送者允许列表回退：

-   如果未设置 `groupAllowFrom`，运行时在可用时回退到 `allowFrom`
-   发送者允许列表在提及/回复激活之前进行评估

注意：如果完全不存在 `channels.whatsapp` 块，运行时群组策略回退是 `allowlist`（并记录警告日志），即使设置了 `channels.defaults.groupPolicy`。

群组回复默认需要提及。提及检测包括：

-   对机器人身份的显式 WhatsApp 提及
-   配置的提及正则表达式模式 (`agents.list[].groupChat.mentionPatterns`, 回退 `messages.groupChat.mentionPatterns`)
-   隐式回复给机器人的检测（回复发送者匹配机器人身份）

安全说明：

-   引用/回复仅满足提及门控；它**不**授予发送者授权
-   使用 `groupPolicy: "allowlist"` 时，非允许列表的发送者即使回复了允许列表用户的消息，仍然会被阻止

会话级激活命令：

-   `/activation mention`
-   `/activation always`

`activation` 更新会话状态（非全局配置）。它受所有者门控。

## 个人号码与自聊行为

当链接的自有号码也存在于 `allowFrom` 中时，WhatsApp 自聊保护机制激活：

-   跳过自聊回合的已读回执
-   忽略提及 JID 的自动触发行为，否则会 ping 您自己
-   如果未设置 `messages.responsePrefix`，自聊回复默认为 `[{identity.name}]` 或 `[openclaw]`

## 消息规范化与上下文

传入的 WhatsApp 消息被包装在共享的入站信封中。如果存在引用回复，上下文会以以下形式追加：

```json
[回复 <sender> id:<stanzaId>]
<引用的正文或媒体占位符>
[/回复]
```

回复元数据字段在可用时也会被填充 (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, 发送者 JID/E.164)。

仅包含媒体的入站消息使用占位符进行规范化，例如：

-   `<media:image>`
-   `<media:video>`
-   `<media:audio>`
-   `<media:document>`
-   `<media:sticker>`

位置和联系人有效负载在路由前被规范化为文本上下文。

对于群组，未处理的消息可以被缓冲，并在机器人最终被触发时作为上下文注入。

-   默认限制：`50`
-   配置：`channels.whatsapp.historyLimit`
-   回退：`messages.groupChat.historyLimit`
-   `0` 表示禁用

注入标记：

-   `[自您上次回复以来的聊天消息 - 供上下文参考]`
-   `[当前消息 - 请回复此消息]`

对于已接受的入站 WhatsApp 消息，默认启用已读回执。全局禁用：

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

按账户覆盖：

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

自聊回合即使全局启用也会跳过已读回执。

## 送达、分块和媒体

-   默认分块限制：`channels.whatsapp.textChunkLimit = 4000`
-   `channels.whatsapp.chunkMode = "length" | "newline"`
-   `newline` 模式优先考虑段落边界（空行），然后回退到长度安全的分块

-   支持图像、视频、音频（PTT 语音便签）和文档有效负载
-   `audio/ogg` 被重写为 `audio/ogg; codecs=opus` 以兼容语音便签
-   通过视频发送时的 `gifPlayback: true` 支持动画 GIF 播放
-   在发送多媒体回复有效负载时，标题应用于第一个媒体项
-   媒体源可以是 HTTP(S)、`file://` 或本地路径

-   入站媒体保存上限：`channels.whatsapp.mediaMaxMb` (默认 `50`)
-   出站媒体发送上限：`channels.whatsapp.mediaMaxMb` (默认 `50`)
-   按账户覆盖使用 `channels.whatsapp.accounts..mediaMaxMb`
-   图像会自动优化（调整大小/质量扫描）以适应限制
-   媒体发送失败时，第一项回退会发送文本警告，而不是静默丢弃响应

## 确认反应

WhatsApp 支持通过 `channels.whatsapp.ackReaction` 在入站接收时立即发送确认反应。

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

行为说明：

-   在入站被接受后（回复前）立即发送
-   失败会被记录，但不会阻止正常回复的送达
-   群组模式 `mentions` 在提及触发的回合中反应；群组激活 `always` 可绕过此检查
-   WhatsApp 使用 `channels.whatsapp.ackReaction`（旧版 `messages.ackReaction` 在此处不使用）

## 多账户与凭据

-   账户 ID 来自 `channels.whatsapp.accounts`
-   默认账户选择：如果存在 `default`，否则是第一个配置的账户 ID（已排序）
-   账户 ID 在内部进行规范化以便查找

-   当前认证路径：`~/.openclaw/credentials/whatsapp//creds.json`
-   备份文件：`creds.json.bak`
-   位于 `~/.openclaw/credentials/` 的旧版默认认证在默认账户流程中仍被识别/迁移

`openclaw channels logout --channel whatsapp [--account ]` 清除该账户的 WhatsApp 认证状态。在旧版认证目录中，`oauth.json` 被保留，而 Baileys 认证文件被移除。

## 工具、操作和配置写入

-   代理工具支持包括 WhatsApp 反应操作 (`react`)。
-   操作门控：
    -   `channels.whatsapp.actions.reactions`
    -   `channels.whatsapp.actions.polls`
-   频道发起的配置写入默认启用（通过 `channels.whatsapp.configWrites=false` 禁用）。

## 故障排除

症状：频道状态报告未链接。修复：

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

症状：已链接账户出现重复断开连接或重连尝试。修复：

```bash
openclaw doctor
openclaw logs --follow
```

如果需要，使用 `channels login` 重新链接。

当目标账户没有活跃的网关监听器时，出站发送会快速失败。确保网关正在运行且账户已链接。

按此顺序检查：

-   `groupPolicy`
-   `groupAllowFrom` / `allowFrom`
-   `groups` 允许列表条目
-   提及门控 (`requireMention` + 提及模式)
-   `openclaw.json` (JSON5) 中的重复键：后面的条目会覆盖前面的，因此每个作用域只保留一个 `groupPolicy`

WhatsApp 网关运行时应使用 Node。Bun 被标记为与稳定的 WhatsApp/Telegram 网关操作不兼容。

## 配置参考指针

主要参考：

-   [配置参考 - WhatsApp](../gateway/configuration-reference.md#whatsapp)

高信号 WhatsApp 字段：

-   访问：`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
-   送达：`textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
-   多账户：`accounts..enabled`, `accounts..authDir`, 账户级覆盖
-   操作：`configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
-   会话行为：`session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms..historyLimit`

## 相关

-   [配对](./pairing.md)
-   [频道路由](./channel-routing.md)
-   [多代理路由](../concepts/multi-agent.md)
-   [故障排除](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)