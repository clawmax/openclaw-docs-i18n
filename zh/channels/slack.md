

  消息平台

  
# Slack

状态：通过 Slack 应用集成，私信和频道功能已可用于生产环境。默认模式为 Socket 模式；也支持 HTTP Events API 模式。

## 快速设置

## 令牌模型

- `botToken` + `appToken` 是 Socket 模式所必需的。
- HTTP 模式需要 `botToken` + `signingSecret`。
- 配置令牌会覆盖环境变量回退。
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` 环境变量回退仅适用于默认账户。
- `userToken` (`xoxp-...`) 仅支持配置（无环境变量回退），并默认为只读行为 (`userTokenReadOnly: true`)。
- 可选：添加 `chat:write.customize` 权限，如果你希望外发消息使用活跃代理身份（自定义 `username` 和图标）。`icon_emoji` 使用 `:emoji_name:` 语法。

> **💡** 对于操作/目录读取，配置后可以优先使用用户令牌。对于写入操作，仍优先使用机器人令牌；仅当 `userTokenReadOnly: false` 且机器人令牌不可用时，才允许使用用户令牌写入。

## 访问控制与路由

`channels.slack.dmPolicy` 控制私信访问（旧版：`channels.slack.dm.policy`）：

- `pairing`（默认）
- `allowlist`
- `open`（要求 `channels.slack.allowFrom` 包含 `"*"`；旧版：`channels.slack.dm.allowFrom`）
- `disabled`

私信标志：

- `dm.enabled`（默认 true）
- `channels.slack.allowFrom`（首选）
- `dm.allowFrom`（旧版）
- `dm.groupEnabled`（群组私信默认 false）
- `dm.groupChannels`（可选的多方即时消息允许列表）

多账户优先级：

- `channels.slack.accounts.default.allowFrom` 仅适用于 `default` 账户。
- 命名账户在其自身的 `allowFrom` 未设置时，会继承 `channels.slack.allowFrom`。
- 命名账户不会继承 `channels.slack.accounts.default.allowFrom`。

私信中的配对使用 `openclaw pairing approve slack `。

`channels.slack.groupPolicy` 控制频道处理：

- `open`
- `allowlist`
- `disabled`

频道允许列表位于 `channels.slack.channels` 下。运行时注意：如果 `channels.slack` 完全缺失（仅环境变量设置），运行时会回退到 `groupPolicy="allowlist"` 并记录警告（即使设置了 `channels.defaults.groupPolicy`）。名称/ID 解析：

- 频道允许列表条目和私信允许列表条目在启动时解析（当令牌访问允许时）
- 未解析的条目保持配置原样
- 入站授权匹配默认优先使用 ID；直接用户名/别名匹配需要 `channels.slack.dangerouslyAllowNameMatching: true`

频道消息默认需要提及才能触发。提及来源：

- 显式应用提及 (`<@botId>`)
- 提及正则表达式模式 (`agents.list[].groupChat.mentionPatterns`，回退到 `messages.groupChat.mentionPatterns`)
- 隐式回复给机器人的线程行为

每频道控制 (`channels.slack.channels.<id|name>`)：

- `requireMention`
- `users`（允许列表）
- `allowBots`
- `skills`
- `systemPrompt`
- `tools`、`toolsBySender`
- `toolsBySender` 键格式：`id:`、`e164:`、`username:`、`name:` 或 `"*"` 通配符（旧版无前缀键仍仅映射到 `id:`）

## 命令与斜杠行为

- 原生命令自动模式在 Slack 中**关闭**（`commands.native: "auto"` 不会启用 Slack 原生命令）。
- 使用 `channels.slack.commands.native: true`（或全局 `commands.native: true`）启用原生 Slack 命令处理器。
- 启用原生命令后，在 Slack 中注册匹配的斜杠命令 (`/` 名称)，但有一个例外：
    - 为状态命令注册 `/agentstatus`（Slack 保留了 `/status`）
- 如果未启用原生命令，你可以通过 `channels.slack.slashCommand` 运行单个配置的斜杠命令。
- 原生参数菜单现在会适配其渲染策略：
    - 最多 5 个选项：按钮块
    - 6-100 个选项：静态选择菜单
    - 超过 100 个选项：外部选择菜单，当交互性选项处理器可用时进行异步选项过滤
    - 如果编码的选项值超过 Slack 限制，流程将回退到按钮
- 对于长选项负载，斜杠命令参数菜单在分派选定值之前使用确认对话框。

默认斜杠命令设置：

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

斜杠会话使用隔离的键：

- `agent::slack:slash:`

并且仍然针对目标会话路由命令执行 (`CommandTargetSessionKey`)。

## 线程、会话与回复标签

- 私信路由为 `direct`；频道为 `channel`；多方即时消息为 `group`。
- 使用默认 `session.dmScope=main` 时，Slack 私信会合并到代理主会话。
- 频道会话：`agent::slack:channel:`。
- 线程回复在适用时可以创建线程会话后缀 (`:thread:`)。
- `channels.slack.thread.historyScope` 默认为 `thread`；`thread.inheritParent` 默认为 `false`。
- `channels.slack.thread.initialHistoryLimit` 控制新线程会话开始时获取多少现有线程消息（默认 `20`；设置为 `0` 以禁用）。

回复线程控制：

- `channels.slack.replyToMode`: `off|first|all`（默认 `off`）
- `channels.slack.replyToModeByChatType`：按 `direct|group|channel`
- 私聊的旧版回退：`channels.slack.dm.replyToMode`

支持手动回复标签：

- `[[reply_to_current]]`
- `[[reply_to:]]`

注意：`replyToMode="off"` 会禁用 Slack 中**所有**回复线程，包括显式的 `[[reply_to_*]]` 标签。这与 Telegram 不同，在 Telegram 中，显式标签在 `"off"` 模式下仍会被遵守。这种差异反映了平台的线程模型：Slack 线程将消息隐藏在频道中，而 Telegram 回复仍保留在主聊天流中可见。

## 媒体、分块与投递

Slack 文件附件从 Slack 托管的私有 URL（令牌认证请求流）下载，并在获取成功且大小限制允许时写入媒体存储。运行时入站大小上限默认为 `20MB`，除非被 `channels.slack.mediaMaxMb` 覆盖。

- 文本分块使用 `channels.slack.textChunkLimit`（默认 4000）
- `channels.slack.chunkMode="newline"` 启用段落优先分割
- 文件发送使用 Slack 上传 API，并可包含线程回复 (`thread_ts`)
- 出站媒体上限遵循配置的 `channels.slack.mediaMaxMb`；否则频道发送使用媒体管道中基于 MIME 类型的默认值

首选的显式目标：

- `user:` 用于私信
- `channel:` 用于频道

发送到用户目标时，Slack 私信通过 Slack 对话 API 打开。

## 操作与门控

Slack 操作由 `channels.slack.actions.*` 控制。当前 Slack 工具中可用的操作组：

| 组 | 默认 |
| --- | --- |
| messages | 启用 |
| reactions | 启用 |
| pins | 启用 |
| memberInfo | 启用 |
| emojiList | 启用 |

## 事件与操作行为

- 消息编辑/删除/线程广播被映射到系统事件。
- 反应添加/移除事件被映射到系统事件。
- 成员加入/离开、频道创建/重命名以及置顶添加/移除事件被映射到系统事件。
- 助手线程状态更新（用于线程中的"正在输入…"指示器）使用 `assistant.threads.setStatus` 并需要机器人权限 `assistant:write`。
- `channel_id_changed` 可以在启用 `configWrites` 时迁移频道配置键。
- 频道主题/目的元数据被视为不受信任的上下文，可以注入到路由上下文中。
- 块操作和模态交互会发出结构化的 `Slack interaction: ...` 系统事件，包含丰富的负载字段：
    - 块操作：选定的值、标签、选择器值和 `workflow_*` 元数据
    - 模态 `view_submission` 和 `view_closed` 事件，包含路由的频道元数据和表单输入

## 确认反应

`ackReaction` 在 OpenClaw 处理入站消息时发送一个确认表情符号。解析顺序：

- `channels.slack.accounts..ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- 代理身份表情符号回退（`agents.list[].identity.emoji`，否则为 "👀"）

注意：

- Slack 期望短代码（例如 `"eyes"`）。
- 使用 `""` 可为 Slack 账户或全局禁用该反应。

## 输入反应回退

`typingReaction` 在 OpenClaw 处理回复时，向入站 Slack 消息添加一个临时反应，然后在运行完成时移除它。当 Slack 原生助手输入状态不可用时（尤其是在私信中），这是一个有用的回退。解析顺序：

- `channels.slack.accounts..typingReaction`
- `channels.slack.typingReaction`

注意：

- Slack 期望短代码（例如 `"hourglass_flowing_sand"`）。
- 该反应是尽力而为的，并在回复或失败路径完成后尝试自动清理。

## 清单与权限范围清单

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

如果你配置了 `channels.slack.userToken`，典型的读取权限范围是：

- `channels:history`、`groups:history`、`im:history`、`mpim:history`
- `channels:read`、`groups:read`、`im:read`、`mpim:read`
- `users:read`
- `reactions:read`
- `pins:read`
- `emoji:read`
- `search:read`（如果你依赖 Slack 搜索读取）

## 故障排除

按顺序检查：

- `groupPolicy`
- 频道允许列表 (`channels.slack.channels`)
- `requireMention`
- 每频道 `users` 允许列表

有用的命令：

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

检查：

- `channels.slack.dm.enabled`
- `channels.slack.dmPolicy`（或旧版 `channels.slack.dm.policy`）
- 配对批准 / 允许列表条目

```bash
openclaw pairing list slack
```

验证 Slack 应用设置中的机器人 + 应用令牌以及 Socket 模式启用状态。

验证：

- 签名密钥
- Webhook 路径
- Slack 请求 URL（事件 + 交互性 + 斜杠命令）
- 每个 HTTP 账户的 `webhookPath` 唯一性

验证你的意图是：

- 原生命令模式 (`channels.slack.commands.native: true`) 并在 Slack 中注册了匹配的斜杠命令
- 或单斜杠命令模式 (`channels.slack.slashCommand.enabled: true`)

同时检查 `commands.useAccessGroups` 和频道/用户允许列表。

## 文本流式传输

OpenClaw 通过 Agents and AI Apps API 支持 Slack 原生文本流式传输。`channels.slack.streaming` 控制实时预览行为：

- `off`：禁用实时预览流式传输。
- `partial`（默认）：用最新的部分输出替换预览文本。
- `block`：追加分块的预览更新。
- `progress`：在生成时显示进度状态文本，然后发送最终文本。

`channels.slack.nativeStreaming` 在 `streaming` 为 `partial` 时控制 Slack 的原生流式传输 API (`chat.startStream` / `chat.appendStream` / `chat.stopStream`)（默认：`true`）。禁用原生 Slack 流式传输（保留草稿预览行为）：

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

旧版键：

- `channels.slack.streamMode` (`replace | status_final | append`) 会自动迁移到 `channels.slack.streaming`。
- 布尔值 `channels.slack.streaming` 会自动迁移到 `channels.slack.nativeStreaming`。

### 要求

1. 在你的 Slack 应用设置中启用 **Agents and AI Apps**。
2. 确保应用拥有 `assistant:write` 权限范围。
3. 该消息必须有一个可用的回复线程。线程选择仍遵循 `replyToMode`。

### 行为

- 第一个文本块启动一个流 (`chat.startStream`)。
- 后续文本块追加到同一个流 (`chat.appendStream`)。
- 回复结束时最终化流 (`chat.stopStream`)。
- 媒体和非文本负载回退到正常投递。
- 如果流式传输在回复中途失败，OpenClaw 会对剩余负载回退到正常投递。

## 配置参考指针

主要参考：

- [配置参考 - Slack](../gateway/configuration-reference.md#slack) 高信号 Slack 字段：
    - 模式/认证：`mode`、`botToken`、`appToken`、`signingSecret`、`webhookPath`、`accounts.*`
    - 私信访问：`dm.enabled`、`dmPolicy`、`allowFrom`（旧版：`dm.policy`、`dm.allowFrom`）、`dm.groupEnabled`、`dm.groupChannels`
    - 兼容性开关：`dangerouslyAllowNameMatching`（应急措施；除非需要，否则保持关闭）
    - 频道访问：`groupPolicy`、`channels.*`、`channels.*.users`、`channels.*.requireMention`
    - 线程/历史记录：`replyToMode`、`replyToModeByChatType`、`thread.*`、`historyLimit`、`dmHistoryLimit`、`dms.*.historyLimit`
    - 投递：`textChunkLimit`、`chunkMode`、`mediaMaxMb`、`streaming`、`nativeStreaming`
    - 运维/功能：`configWrites`、`commands.native`、`slashCommand.*`、`actions.*`、`userToken`、`userTokenReadOnly`

## 相关链接

- [配对](./pairing.md)
- [频道路由](./channel-routing.md)
- [故障排除](./troubleshooting.md)
- [配置](../gateway/configuration.md)
- [斜杠命令](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)