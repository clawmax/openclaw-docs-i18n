

  消息平台

  
# Telegram

状态：通过 grammY 实现机器人私聊 + 群组功能，已可用于生产环境。长轮询是默认模式；Webhook 模式为可选。

## 快速设置

### 步骤 1：在 BotFather 中创建机器人令牌

打开 Telegram 并与 **@BotFather** 聊天（确认其用户名确为 `@BotFather`）。运行 `/newbot`，按照提示操作，并保存令牌。

### 步骤 2：配置令牌和私聊策略

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

环境变量后备方案：`TELEGRAM_BOT_TOKEN=...`（仅适用于默认账户）。Telegram **不**使用 `openclaw channels login telegram`；请在配置/环境变量中配置令牌，然后启动网关。

### 步骤 3：启动网关并批准首次私聊

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

配对码在 1 小时后过期。

### 步骤 4：将机器人添加到群组

将机器人添加到您的群组，然后设置 `channels.telegram.groups` 和 `groupPolicy` 以匹配您的访问模型。

> **ℹ️** 令牌解析顺序是账户感知的。实际上，配置值优先于环境变量后备方案，且 `TELEGRAM_BOT_TOKEN` 仅适用于默认账户。

## Telegram 端设置

Telegram 机器人默认启用**隐私模式**，这会限制它们接收到的群组消息。如果机器人必须看到所有群组消息，请执行以下任一操作：

-   通过 `/setprivacy` 禁用隐私模式，或
-   将机器人设为群组管理员。

切换隐私模式后，请从每个群组中移除并重新添加机器人，以便 Telegram 应用更改。

管理员状态在 Telegram 群组设置中控制。管理员机器人接收所有群组消息，这对于始终在线的群组行为很有用。

-   `/setjoingroups` 允许/拒绝加入群组
-   `/setprivacy` 用于群组可见性行为

## 访问控制和激活

/getUpdates"', lang: 'bash' }, { label: '群组策略和允许列表', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: '提及行为', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## 运行时行为

-   Telegram 由网关进程拥有。
-   路由是确定性的：Telegram 的入站回复会返回给 Telegram（模型不会选择频道）。
-   入站消息会规范化为共享的信封格式，包含回复元数据和媒体占位符。
-   群组会话按群组 ID 隔离。论坛话题会附加 `:topic:` 以保持话题隔离。
-   私聊消息可以携带 `message_thread_id`；OpenClaw 使用线程感知的会话密钥路由它们，并为回复保留线程 ID。
-   长轮询使用 grammY runner，具有按聊天/按线程的排序功能。整体 runner 接收器的并发性使用 `agents.defaults.maxConcurrent`。
-   Telegram Bot API 不支持已读回执（`sendReadReceipts` 不适用）。

## 功能参考

OpenClaw 可以实时流式传输部分回复：

-   私聊：通过 `sendMessageDraft` 实现 Telegram 原生草稿流式传输
-   群组/话题：预览消息 + `editMessageText`

要求：

-   `channels.telegram.streaming` 设置为 `off | partial | block | progress`（默认：`partial`）
-   `progress` 在 Telegram 上映射为 `partial`（与跨频道命名兼容）
-   旧版 `channels.telegram.streamMode` 和布尔值 `streaming` 会自动映射

Telegram 在 Bot API 9.5（2026年3月1日）中为所有机器人启用了 `sendMessageDraft`。对于纯文本回复：

-   私聊：OpenClaw 在原地更新草稿（无需额外的预览消息）
-   群组/话题：OpenClaw 保持相同的预览消息，并在原地执行最终编辑（无需第二条消息）

对于复杂回复（例如媒体负载），OpenClaw 会回退到正常的最终交付，然后清理预览消息。预览流与块流是分开的。当为 Telegram 显式启用块流时，OpenClaw 会跳过预览流以避免双重流式传输。如果原生草稿传输不可用/被拒绝，OpenClaw 会自动回退到 `sendMessage` + `editMessageText`。仅限 Telegram 的推理流：

-   `/reasoning stream` 在生成时将推理发送到实时预览
-   最终答案发送时不包含推理文本

出站文本使用 Telegram 的 `parse_mode: "HTML"`。

-   Markdown 风格的文本会渲染为 Telegram 安全的 HTML。
-   原始模型 HTML 会被转义以减少 Telegram 解析失败。
-   如果 Telegram 拒绝解析 HTML，OpenClaw 会重试为纯文本。

链接预览默认启用，可以通过 `channels.telegram.linkPreview: false` 禁用。

Telegram 命令菜单注册在启动时通过 `setMyCommands` 处理。原生命令默认值：

-   `commands.native: "auto"` 为 Telegram 启用原生命令

添加自定义命令菜单项：

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git 备份" },
        { command: "generate", description: "创建图像" },
      ],
    },
  },
}
```

规则：

-   名称被规范化（去除前导 `/`，转为小写）
-   有效模式：`a-z`、`0-9`、`_`，长度 `1..32`
-   自定义命令不能覆盖原生命令
-   冲突/重复项会被跳过并记录

注意：

-   自定义命令仅是菜单项；它们不会自动实现行为
-   插件/技能命令即使未显示在 Telegram 菜单中，在键入时仍可工作

如果原生命令被禁用，内置命令会被移除。自定义/插件命令在配置后仍可能注册。常见设置失败：

-   `setMyCommands failed` 通常意味着到 `api.telegram.org` 的出站 DNS/HTTPS 被阻止。

### 设备配对命令（device-pair 插件）

安装 `device-pair` 插件时：

1.  `/pair` 生成设置码
2.  在 iOS 应用中粘贴代码
3.  `/pair approve` 批准最新的待处理请求

更多详情：[配对](./pairing.md#pair-via-telegram-recommended-for-ios)。

配置内联键盘范围：

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

按账户覆盖：

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

范围：

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist`（默认）

旧版 `capabilities: ["inlineButtons"]` 映射为 `inlineButtons: "all"`。消息操作示例：

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "选择一个选项：",
  buttons: [
    [
      { text: "是", callback_data: "yes" },
      { text: "否", callback_data: "no" },
    ],
    [{ text: "取消", callback_data: "cancel" }],
  ],
}
```

回调点击作为文本传递给代理：`callback_data: `

Telegram 工具操作包括：

-   `sendMessage`（`to`、`content`，可选 `mediaUrl`、`replyToMessageId`、`messageThreadId`）
-   `react`（`chatId`、`messageId`、`emoji`）
-   `deleteMessage`（`chatId`、`messageId`）
-   `editMessage`（`chatId`、`messageId`、`content`）
-   `createForumTopic`（`chatId`、`name`，可选 `iconColor`、`iconCustomEmojiId`）

频道消息操作暴露了符合人体工学的别名（`send`、`react`、`delete`、`edit`、`sticker`、`sticker-search`、`topic-create`）。门控控制：

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker`（默认：禁用）

注意：`edit` 和 `topic-create` 当前默认启用，没有单独的 `channels.telegram.actions.*` 开关。反应移除语义：[/tools/reactions](../tools/reactions.md)

Telegram 支持在生成的输出中使用显式回复线程标签：

-   `[[reply_to_current]]` 回复触发消息
-   `[[reply_to:]]` 回复特定的 Telegram 消息 ID

`channels.telegram.replyToMode` 控制处理方式：

-   `off`（默认）
-   `first`
-   `all`

注意：`off` 禁用隐式回复线程。显式的 `[[reply_to_*]]` 标签仍会被遵守。

论坛超级群组：

-   话题会话密钥附加 `:topic:`
-   回复和输入状态针对话题线程
-   话题配置路径：`channels.telegram.groups..topics.`

常规话题（`threadId=1`）特殊情况：

-   消息发送省略 `message_thread_id`（Telegram 拒绝 `sendMessage(...thread_id=1)`）
-   输入状态操作仍包含 `message_thread_id`

话题继承：话题条目继承群组设置，除非被覆盖（`requireMention`、`allowFrom`、`skills`、`systemPrompt`、`enabled`、`groupPolicy`）。`agentId` 是话题特有的，不从群组默认值继承。**按话题代理路由**：每个话题可以通过在话题配置中设置 `agentId` 路由到不同的代理。这为每个话题提供了独立的隔离工作区、内存和会话。示例：

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // 常规话题 → 主代理
            "3": { agentId: "zu" },        // 开发话题 → zu 代理
            "5": { agentId: "coder" }      // 代码审查 → coder 代理
          }
        }
      }
    }
  }
}
```

每个话题都有自己的会话密钥：`agent:zu:telegram:group:-1001234567890:topic:3`**持久化 ACP 话题绑定**：论坛话题可以通过顶层类型化 ACP 绑定固定 ACP 工具会话：

-   `bindings[]` 包含 `type: "acp"` 和 `match.channel: "telegram"`

示例：

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

这目前仅限于群组和超级群组中的论坛话题。**从聊天中线程绑定的 ACP 生成**：

-   `/acp spawn  --thread here|auto` 可以将当前 Telegram 话题绑定到新的 ACP 会话。
-   后续的话题消息直接路由到绑定的 ACP 会话（无需 `/acp steer`）。
-   OpenClaw 在成功绑定后会在话题内固定生成确认消息。
-   需要 `channels.telegram.threadBindings.spawnAcpSessions=true`。

模板上下文包括：

-   `MessageThreadId`
-   `IsForum`

私聊线程行为：

-   带有 `message_thread_id` 的私人聊天保持私聊路由，但使用线程感知的会话密钥/回复目标。

### 音频消息

Telegram 区分语音笔记和音频文件。

-   默认：音频文件行为
-   在代理回复中使用标签 `[[audio_as_voice]]` 强制发送语音笔记

消息操作示例：

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### 视频消息

Telegram 区分视频文件和视频笔记。消息操作示例：

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

视频笔记不支持标题；提供的消息文本会单独发送。

### 贴纸

入站贴纸处理：

-   静态 WEBP：下载并处理（占位符 `<media:sticker>`）
-   动画 TGS：跳过
-   视频 WEBM：跳过

贴纸上下文字段：

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

贴纸缓存文件：

-   `~/.openclaw/telegram/sticker-cache.json`

贴纸会被描述一次（如果可能）并缓存，以减少重复的视觉调用。启用贴纸操作：

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

发送贴纸操作：

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

搜索缓存的贴纸：

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Telegram 反应作为 `message_reaction` 更新到达（与消息负载分开）。启用后，OpenClaw 会排队系统事件，例如：

-   `Telegram 反应已添加：👍 由 Alice (@alice) 在消息 42 上`

配置：

-   `channels.telegram.reactionNotifications`：`off | own | all`（默认：未设置时为 `own`）
-   `channels.telegram.reactionLevel`：`off | ack | minimal | extensive`（默认：未设置时为 `minimal`）

注意：

-   `own` 表示仅用户对机器人发送消息的反应（通过已发送消息缓存尽力而为）。
-   反应事件仍遵守 Telegram 访问控制（`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`）；未经授权的发送者会被丢弃。
-   Telegram 不在反应更新中提供线程 ID。
    -   非论坛群组路由到群组聊天会话
    -   论坛群组路由到群组常规话题会话（`:topic:1`），而不是确切的原话题

用于轮询/webhook 的 `allowed_updates` 自动包含 `message_reaction`。

`ackReaction` 在 OpenClaw 处理入站消息时发送一个确认表情符号。解析顺序：

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   代理身份表情符号后备方案（`agents.list[].identity.emoji`，否则为 ”👀”）

注意：

-   Telegram 期望 Unicode 表情符号（例如 ”👀”）。
-   使用 `""` 为频道或账户禁用该反应。

频道配置写入默认启用（`configWrites !== false`）。Telegram 触发的写入包括：

-   群组迁移事件（`migrate_to_chat_id`）以更新 `channels.telegram.groups`
-   `/config set` 和 `/config unset`（需要命令启用）

禁用：

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

默认：长轮询。Webhook 模式：

-   设置 `channels.telegram.webhookUrl`
-   设置 `channels.telegram.webhookSecret`（设置 webhook URL 时需要）
-   可选 `channels.telegram.webhookPath`（默认 `/telegram-webhook`）
-   可选 `channels.telegram.webhookHost`（默认 `127.0.0.1`）
-   可选 `channels.telegram.webhookPort`（默认 `8787`）

Webhook 模式的默认本地监听器绑定到 `127.0.0.1:8787`。如果您的公共端点不同，请在前面放置反向代理，并将 `webhookUrl` 指向公共 URL。当您确实需要外部入口时，设置 `webhookHost`（例如 `0.0.0.0`）。

-   `channels.telegram.textChunkLimit` 默认值为 4000。
-   `channels.telegram.chunkMode="newline"` 在长度分割前优先考虑段落边界（空行）。
-   `channels.telegram.mediaMaxMb`（默认 100）限制入站和出站 Telegram 媒体大小。
-   `channels.telegram.timeoutSeconds` 覆盖 Telegram API 客户端超时（如果未设置，则应用 grammY 默认值）。
-   群组上下文历史记录使用 `channels.telegram.historyLimit` 或 `messages.groupChat.historyLimit`（默认 50）；`0` 表示禁用。
-   私聊历史记录控制：
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   `channels.telegram.retry` 配置适用于 Telegram 发送助手（CLI/工具/操作），用于处理可恢复的出站 API 错误。

CLI 发送目标可以是数字聊天 ID 或用户名：

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Telegram 投票使用 `openclaw message poll` 并支持论坛话题：

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

仅限 Telegram 的投票标志：

-   `--poll-duration-seconds`（5-600）
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` 用于论坛话题（或使用 `:topic:` 目标）

操作门控：

-   `channels.telegram.actions.sendMessage=false` 禁用出站 Telegram 消息，包括投票
-   `channels.telegram.actions.poll=false` 禁用 Telegram 投票创建，同时保持常规发送启用

## 故障排除

-   如果 `requireMention=false`，Telegram 隐私模式必须允许完全可见性。
    -   BotFather：`/setprivacy` -> 禁用
    -   然后从群组中移除并重新添加机器人
-   `openclaw channels status` 在配置期望未提及的群组消息时会发出警告。
-   `openclaw channels status --probe` 可以检查显式的数字群组 ID；通配符 `"*"` 无法进行成员资格探测。
-   快速会话测试：`/activation always`。

-   当 `channels.telegram.groups` 存在时，群组必须被列出（或包含 `"*"`）
-   验证机器人在群组中的成员资格
-   查看日志：`openclaw logs --follow` 以获取跳过原因

-   授权您的发送者身份（配对和/或数字 `allowFrom`）
-   即使群组策略是 `open`，命令授权仍然适用
-   `setMyCommands failed` 通常表示到 `api.telegram.org` 的 DNS/HTTPS 可达性问题

-   Node 22+ + 自定义 fetch/代理 如果 AbortSignal 类型不匹配，可能触发立即中止行为。
-   某些主机将 `api.telegram.org` 首先解析为 IPv6；损坏的 IPv6 出口可能导致间歇性 Telegram API 故障。
-   如果日志包含 `TypeError: fetch failed` 或 `Network request for 'getUpdates' failed!`，OpenClaw 现在会将这些作为可恢复的网络错误重试。
-   在具有不稳定直接出口/TLS 的 VPS 主机上，通过 `channels.telegram.proxy` 路由 Telegram API 调用：

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ 默认 `autoSelectFamily=true`（WSL2 除外）和 `dnsResultOrder=ipv4first`。
-   如果您的主机是 WSL2 或明确使用仅 IPv4 行为效果更好，强制选择族：

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   环境变量覆盖（临时）：
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   验证 DNS 答案：

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

更多帮助：[频道故障排除](./troubleshooting.md)。

## Telegram 配置参考要点

主要参考：

-   `channels.telegram.enabled`：启用/禁用频道启动。
-   `channels.telegram.botToken`：机器人令牌（BotFather）。
-   `channels.telegram.tokenFile`：从文件路径读取令牌。
-   `channels.telegram.dmPolicy`：`pairing | allowlist | open | disabled`（默认：配对）。
-   `channels.telegram.allowFrom`：私聊允许列表（数字 Telegram 用户 ID）。`allowlist` 需要至少一个发送者 ID。`open` 需要 `"*"`。`openclaw doctor --fix` 可以将旧版 `@username` 条目解析为 ID，并可以在允许列表迁移流程中从配对存储文件中恢复允许列表条目。
-   `channels.telegram.actions.poll`：启用或禁用 Telegram 投票创建（默认：启用；仍需要 `sendMessage`）。
-   `channels.telegram.defaultTo`：当没有显式提供 `--reply-to` 时，CLI `--deliver` 使用的默认 Telegram 目标。
-   `channels.telegram.groupPolicy`：`open | allowlist | disabled`（默认：允许列表）。
-   `channels.telegram.groupAllowFrom`：群组发送者允许列表（数字 Telegram 用户 ID）。`openclaw doctor --fix` 可以将旧版 `@username` 条目解析为 ID。非数字条目在授权时被忽略。群组授权不使用私聊配对存储后备方案（`2026.2.25+`）。
-   多账户优先级：
    -   当配置两个或更多账户 ID 时，设置 `channels.telegram.defaultAccount`（或包含 `channels.telegram.accounts.default`）以使默认路由明确。
    -   如果两者都未设置，OpenClaw 回退到第一个规范化的账户 ID，并且 `openclaw doctor` 会发出警告。
    -   `channels.telegram.accounts.default.allowFrom` 和 `channels.telegram.accounts.default.groupAllowFrom` 仅适用于 `default` 账户。
    -   命名账户在账户级值未设置时继承 `channels.telegram.allowFrom` 和 `channels.telegram.groupAllowFrom`。
    -   命名账户不继承 `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`。
-   `channels.telegram.groups`：每群组默认值 + 允许列表（使用 `"*"` 表示全局默认值）。
    -   `channels.telegram.groups..groupPolicy`：群组策略的每群组覆盖（`open | allowlist | disabled`）。
    -   `channels.telegram.groups..requireMention`：提及门控默认值。
    -   `channels.telegram.groups..skills`：技能过滤器（省略 = 所有技能，空 = 无）。
    -   `channels.telegram.groups..allowFrom`：每群组发送者允许列表覆盖。
    -   `channels.telegram.groups..systemPrompt`：群组的额外系统提示。
    -   `channels.telegram.groups..enabled`：当为 `false` 时禁用该群组。
    -   `channels.telegram.groups..topics..*`：每话题覆盖（群组字段 + 话题特有的 `agentId`）。
    -   `channels.telegram.groups..topics..agentId`：将此话题路由到特定代理（覆盖群组级和绑定路由）。
    -   `channels.telegram.groups..topics..groupPolicy`：群组策略的每话题覆盖（`open | allowlist | disabled`）。
    -   `channels.telegram.groups..topics..requireMention`：提及门控的每话题覆盖。
    -   顶层 `bindings[]` 包含 `type: "acp"` 和规范话题 id `chatId:topic:topicId` 在 `match.peer.id` 中：持久化 ACP 话题绑定字段（参见 [ACP 代理](../tools/acp-agents.md#channel-specific-settings)）。
    -   `channels.telegram.direct..topics..agentId`：将私聊话题路由到特定代理（与论坛话题行为相同）。
-   `channels.telegram.capabilities.inlineButtons`：`off | dm | group | all | allowlist`（默认：allowlist）。
-   `channels.telegram.accounts..capabilities.inlineButtons`：每账户覆盖。
-   `channels.telegram.commands.nativeSkills`：启用/禁用 Telegram 原生技能命令。
-   `channels.telegram.replyToMode`：`off | first | all`（默认：`off`）。
-   `channels.telegram.textChunkLimit`：出站块大小（字符）。
-   `channels.telegram.chunkMode`：`length`（默认）或 `newline` 以在长度分块前按空行（段落边界）分割。
-   `channels.telegram.linkPreview`：切换出站消息的链接预览（默认：true）。
-   `channels.telegram.streaming`：`off | partial | block | progress`（实时流预览；默认：`partial`；`progress` 映射为 `partial`；`block` 是旧版预览模式兼容性）。在私聊中，`partial` 在可用时使用原生 `sendMessageDraft`。
-   `channels.telegram.mediaMaxMb`：入站/出站 Telegram 媒体上限（MB，默认：100）。
-   `channels.telegram.retry`：针对可恢复的出站 API 错误，Telegram 发送助手（CLI/工具/操作）的重试策略（尝试次数、minDelayMs、maxDelayMs、抖动）。
-   `channels.telegram.network.autoSelectFamily`：覆盖 Node 的 autoSelectFamily（true=启用，false=禁用）。在 Node 22+ 上默认启用，WSL2 默认禁用。
-   `channels.telegram.network.dnsResultOrder`：覆盖 DNS 结果顺序（`ipv4first` 或 `verbatim`）。在 Node 22+ 上默认为 `ipv4first`。
-   `channels.telegram.proxy`：用于 Bot API 调用的代理 URL（SOCKS/HTTP）。
-   `channels.telegram.webhookUrl`：启用 webhook 模式（需要 `channels.telegram.webhookSecret`）。
-   `channels.telegram.webhookSecret`：webhook 密钥（设置 webhookUrl 时需要）。
-   `channels.telegram.webhookPath`：本地 webhook 路径（默认 `/telegram-webhook`）。
-   `channels.telegram.webhookHost`：本地 webhook 绑定主机（默认 `127.0.0.1`）。
-   `channels.telegram.webhookPort`：本地 webhook 绑定端口（默认 `8787`）。
-   `channels.telegram.actions.reactions`：门控 Telegram 工具反应。
-   `channels.telegram.actions.sendMessage`：门控 Telegram 工具消息发送。
-   `channels.telegram.actions.deleteMessage`：门控 Telegram 工具消息删除。
-   `channels.telegram.actions.sticker`：门控 Telegram 贴纸操作 — 发送和搜索（默认：false）。
-   `channels.telegram.reactionNotifications`：`off | own | all` — 控制哪些反应触发系统事件（默认：未设置时为 `own`）。
-   `channels.telegram.reactionLevel`：`off | ack | minimal | extensive` — 控制代理的反应能力（默认：未设置时为 `minimal`）。
-   [配置参考 - Telegram](../gateway/configuration-reference.md#telegram)

Telegram 特有的高信号字段：

-   启动/授权：`enabled`、`botToken`、`tokenFile`、`accounts.*`
-   访问控制：`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`、`groups`、`groups.*.topics.*`、顶层 `bindings[]`（`type: "acp"`）
-   命令/菜单：`commands.native`、`commands.nativeSkills`、`customCommands`
-   线程/回复：`replyToMode`
-   流式传输：`streaming`（预览）、`blockStreaming`
-   格式化/交付：`textChunkLimit`、`chunkMode`、`linkPreview`、`responsePrefix`
-   媒体/网络：`mediaMaxMb`、`timeoutSeconds`、`retry`、`network.autoSelectFamily`、`proxy`
-   Webhook：`webhookUrl`、`webhookSecret`、`webhookPath`、`webhookHost`
-   操作/能力：`capabilities.inlineButtons`、`actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
-   反应：`reactionNotifications`、`reactionLevel`
-   写入/历史记录：`configWrites`、`historyLimit`、`dmHistoryLimit`、`dms.*.historyLimit`

## 相关

-   [配对](./pairing.md)
-   [频道路由](./channel-routing.md)
-   [多代理路由](../concepts/multi-agent.md)
-   [故障排除](./troubleshooting.md)

[Slack](./slack.md)[Tlon](./tlon.md)