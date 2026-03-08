title: "OpenClaw 网关配置参考与设置指南"
description: "OpenClaw 网关所有配置字段的完整参考。学习如何设置频道、私信策略、模型覆盖，以及配置 WhatsApp、Telegram 和 Discord。"
keywords: ["openclaw 配置", "网关设置", "频道配置", "私信策略", "模型覆盖", "whatsapp 机器人", "telegram 机器人", "discord 机器人"]
---

  配置与操作

  
# 配置参考

`~/.openclaw/openclaw.json` 中可用的每个字段。如需面向任务的概览，请参阅[配置](./configuration.md)。配置格式为 **JSON5**（允许注释和尾随逗号）。所有字段都是可选的——省略时 OpenClaw 会使用安全的默认值。

* * *

## 频道

每个频道在其配置部分存在时自动启动（除非 `enabled: false`）。

### 私信和群组访问

所有频道都支持私信策略和群组策略：

| 私信策略 | 行为 |
| --- | --- |
| `pairing` (默认) | 未知发件人获得一次性配对码；所有者必须批准 |
| `allowlist` | 仅允许 `allowFrom` 中的发件人（或已配对的允许存储） |
| `open` | 允许所有入站私信（需要 `allowFrom: ["*"]`） |
| `disabled` | 忽略所有入站私信 |

| 群组策略 | 行为 |
| --- | --- |
| `allowlist` (默认) | 仅允许匹配配置的允许列表的群组 |
| `open` | 绕过群组允许列表（提及门控仍然适用） |
| `disabled` | 阻止所有群组/房间消息 |

> **ℹ️** `channels.defaults.groupPolicy` 设置当提供商的 `groupPolicy` 未设置时的默认值。配对码在 1 小时后过期。待处理的私信配对请求上限为 **每个频道 3 个**。如果完全缺少提供商块（`channels.` 不存在），运行时的群组策略会回退到 `allowlist`（故障关闭）并发出启动警告。

### 频道模型覆盖

使用 `channels.modelByChannel` 将特定频道 ID 固定到某个模型。值接受 `provider/model` 或配置的模型别名。当会话尚未有模型覆盖时（例如，通过 `/model` 设置），频道映射会生效。

```json
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
```

### 频道默认值和心跳

使用 `channels.defaults` 为所有提供商设置共享的群组策略和心跳行为：

```json
{
  channels: {
    defaults: {
      groupPolicy: "allowlist", // open | allowlist | disabled
      heartbeat: {
        showOk: false,
        showAlerts: true,
        useIndicator: true,
      },
    },
  },
}
```

-   `channels.defaults.groupPolicy`：当提供商级别的 `groupPolicy` 未设置时的回退群组策略。
-   `channels.defaults.heartbeat.showOk`：在心跳输出中包含健康的频道状态。
-   `channels.defaults.heartbeat.showAlerts`：在心跳输出中包含降级/错误状态。
-   `channels.defaults.heartbeat.useIndicator`：渲染紧凑的指示器风格心跳输出。

### WhatsApp

WhatsApp 通过网关的网页频道（Baileys Web）运行。当存在已链接的会话时自动启动。

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // 蓝色勾选标记（自聊模式下为 false）
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

```json
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

-   出站命令默认使用账号 `default`（如果存在）；否则使用第一个配置的账号 ID（按排序）。
-   可选的 `channels.whatsapp.defaultAccount` 覆盖该回退默认账号选择，当它匹配一个已配置的账号 ID 时。
-   传统的单账号 Baileys 认证目录由 `openclaw doctor` 迁移到 `whatsapp/default`。
-   每个账号的覆盖：`channels.whatsapp.accounts..sendReadReceipts`、`channels.whatsapp.accounts..dmPolicy`、`channels.whatsapp.accounts..allowFrom`。

### Telegram

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (默认: off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 100,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

-   机器人令牌：`channels.telegram.botToken` 或 `channels.telegram.tokenFile`，`TELEGRAM_BOT_TOKEN` 作为默认账号的回退。
-   可选的 `channels.telegram.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   在多账号设置中（2+ 个账号 ID），设置一个明确的默认值（`channels.telegram.defaultAccount` 或 `channels.telegram.accounts.default`）以避免回退路由；当此设置缺失或无效时，`openclaw doctor` 会发出警告。
-   `configWrites: false` 阻止 Telegram 发起的配置写入（超级群组 ID 迁移、`/config set|unset`）。
-   顶层 `bindings[]` 条目中 `type: "acp"` 为论坛主题配置持久的 ACP 绑定（在 `match.peer.id` 中使用规范的 `chatId:topic:topicId`）。字段语义在 [ACP 代理](../tools/acp-agents.md#channel-specific-settings) 中共享。
-   Telegram 流式预览使用 `sendMessage` + `editMessageText`（在直接聊天和群组聊天中均有效）。
-   重试策略：参见[重试策略](../concepts/retry.md)。

### Discord

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          ignoreOtherMentions: true,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress 在 Discord 上映射为 partial)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // 为 sessions_spawn({ thread: true }) 选择启用
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

-   令牌：`channels.discord.token`，`DISCORD_BOT_TOKEN` 作为默认账号的环境变量回退。
-   可选的 `channels.discord.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   使用 `user:`（私信）或 `channel:`（公会频道）作为投递目标；纯数字 ID 会被拒绝。
-   公会别名是小写字母，空格替换为 `-`；频道键使用别名化的名称（不带 `#`）。建议使用公会 ID。
-   默认忽略机器人撰写的消息。`allowBots: true` 启用它们；使用 `allowBots: "mentions"` 仅接受提及机器人的机器人消息（自己的消息仍被过滤）。
-   `channels.discord.guilds..ignoreOtherMentions`（以及频道覆盖）丢弃提及其他用户或角色但未提及机器人的消息（不包括 @everyone/@here）。
-   `maxLinesPerMessage`（默认 17）即使在字符数少于 2000 时也会分割过高的消息。
-   `channels.discord.threadBindings` 控制 Discord 线程绑定的路由：
    -   `enabled`：Discord 对线程绑定会话功能的覆盖（`/focus`、`/unfocus`、`/agents`、`/session idle`、`/session max-age` 以及绑定的投递/路由）
    -   `idleHours`：Discord 对不活动自动取消聚焦的覆盖，单位为小时（`0` 表示禁用）
    -   `maxAgeHours`：Discord 对硬性最大使用时间的覆盖，单位为小时（`0` 表示禁用）
    -   `spawnSubagentSessions`：`sessions_spawn({ thread: true })` 自动线程创建/绑定的选择启用开关
-   顶层 `bindings[]` 条目中 `type: "acp"` 为频道和线程配置持久的 ACP 绑定（在 `match.peer.id` 中使用频道/线程 ID）。字段语义在 [ACP 代理](../tools/acp-agents.md#channel-specific-settings) 中共享。
-   `channels.discord.ui.components.accentColor` 设置 Discord 组件 v2 容器的强调色。
-   `channels.discord.voice` 启用 Discord 语音频道对话和可选的自动加入 + TTS 覆盖。
-   `channels.discord.voice.daveEncryption` 和 `channels.discord.voice.decryptionFailureTolerance` 传递给 `@discordjs/voice` DAVE 选项（默认为 `true` 和 `24`）。
-   OpenClaw 还会在多次解密失败后尝试通过离开/重新加入语音会话来恢复语音接收。
-   `channels.discord.streaming` 是规范的流模式键。传统的 `streamMode` 和布尔值 `streaming` 会自动迁移。
-   `channels.discord.autoPresence` 将运行时可用性映射到机器人状态（健康 => 在线，降级 => 闲置，耗尽 => 请勿打扰）并允许可选的状态文本覆盖。
-   `channels.discord.dangerouslyAllowNameMatching` 重新启用可变名称/标签匹配（紧急兼容模式）。

**反应通知模式：** `off`（无）、`own`（机器人的消息，默认）、`all`（所有消息）、`allowlist`（来自 `guilds..users` 的所有消息）。

### Google Chat

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

-   服务账号 JSON：内联（`serviceAccount`）或基于文件（`serviceAccountFile`）。
-   也支持服务账号 SecretRef（`serviceAccountRef`）。
-   环境变量回退：`GOOGLE_CHAT_SERVICE_ACCOUNT` 或 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
-   使用 `spaces/` 或 `users/` 作为投递目标。
-   `channels.googlechat.dangerouslyAllowNameMatching` 重新启用可变电子邮件主体匹配（紧急兼容模式）。

### Slack

```json
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      typingReaction: "hourglass_flowing_sand",
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (预览模式)
      nativeStreaming: true, // 当 streaming=partial 时使用 Slack 原生流式 API
      mediaMaxMb: 20,
    },
  },
}
```

-   **Socket 模式** 需要 `botToken` 和 `appToken`（`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` 用于默认账号环境变量回退）。
-   **HTTP 模式** 需要 `botToken` 加上 `signingSecret`（在根目录或每个账号）。
-   `configWrites: false` 阻止 Slack 发起的配置写入。
-   可选的 `channels.slack.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   `channels.slack.streaming` 是规范的流模式键。传统的 `streamMode` 和布尔值 `streaming` 会自动迁移。
-   使用 `user:`（私信）或 `channel:` 作为投递目标。

**反应通知模式：** `off`、`own`（默认）、`all`、`allowlist`（来自 `reactionAllowlist`）。**线程会话隔离：** `thread.historyScope` 是每个线程（默认）或在频道内共享。`thread.inheritParent` 将父频道转录复制到新线程。

-   `typingReaction` 在回复运行时为入站 Slack 消息添加临时反应，完成后移除。使用 Slack 表情符号短代码，例如 `"hourglass_flowing_sand"`。

| 操作组 | 默认 | 备注 |
| --- | --- | --- |
| reactions | 启用 | 反应 + 列出反应 |
| messages | 启用 | 读取/发送/编辑/删除 |
| pins | 启用 | 固定/取消固定/列表 |
| memberInfo | 启用 | 成员信息 |
| emojiList | 启用 | 自定义表情符号列表 |

### Mattermost

Mattermost 作为插件提供：`openclaw plugins install @openclaw/mattermost`。

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      commands: {
        native: true, // 选择启用
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // 可选，用于反向代理/公共部署的显式 URL
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

聊天模式：`oncall`（在 @-提及时回复，默认）、`onmessage`（每条消息）、`onchar`（以触发前缀开头的消息）。当启用 Mattermost 原生命令时：

-   `commands.callbackPath` 必须是路径（例如 `/api/channels/mattermost/command`），而不是完整 URL。
-   `commands.callbackUrl` 必须解析到 OpenClaw 网关端点，并且可以从 Mattermost 服务器访问。
-   对于私有/tailnet/内部回调主机，Mattermost 可能需要 `ServiceSettings.AllowedUntrustedInternalConnections` 包含回调主机/域名。使用主机/域名值，而不是完整 URL。
-   `channels.mattermost.configWrites`：允许或拒绝 Mattermost 发起的配置写入。
-   `channels.mattermost.requireMention`：在频道中回复前需要 `@mention`。
-   可选的 `channels.mattermost.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。

### Signal

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // 可选的账号绑定
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**反应通知模式：** `off`、`own`（默认）、`all`、`allowlist`（来自 `reactionAllowlist`）。

-   `channels.signal.account`：将频道启动固定到特定的 Signal 账号身份。
-   `channels.signal.configWrites`：允许或拒绝 Signal 发起的配置写入。
-   可选的 `channels.signal.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。

### BlueBubbles

BlueBubbles 是推荐的 iMessage 路径（插件支持，在 `channels.bluebubbles` 下配置）。

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, group controls, and advanced actions:
      // 参见 /channels/bluebubbles
    },
  },
}
```

-   此处涵盖的核心键路径：`channels.bluebubbles`、`channels.bluebubbles.dmPolicy`。
-   可选的 `channels.bluebubbles.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   完整的 BlueBubbles 频道配置记录在 [BlueBubbles](../channels/bluebubbles.md) 中。

### iMessage

OpenClaw 生成 `imsg rpc`（通过 stdio 的 JSON-RPC）。无需守护进程或端口。

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

-   可选的 `channels.imessage.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   需要 Messages 数据库的完全磁盘访问权限。
-   建议使用 `chat_id:` 目标。使用 `imsg chats --limit 20` 列出聊天。
-   `cliPath` 可以指向 SSH 包装器；设置 `remoteHost`（`host` 或 `user@host`）用于 SCP 附件获取。
-   `attachmentRoots` 和 `remoteAttachmentRoots` 限制入站附件路径（默认：`/Users/*/Library/Messages/Attachments`）。
-   SCP 使用严格的主机密钥检查，因此确保中继主机密钥已存在于 `~/.ssh/known_hosts` 中。
-   `channels.imessage.configWrites`：允许或拒绝 iMessage 发起的配置写入。

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Microsoft Teams

Microsoft Teams 由扩展支持，在 `channels.msteams` 下配置。

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, team/channel policies:
      // 参见 /channels/msteams
    },
  },
}
```

-   此处涵盖的核心键路径：`channels.msteams`、`channels.msteams.configWrites`。
-   完整的 Teams 配置（凭据、webhook、私信/群组策略、每个团队/每个频道的覆盖）记录在 [Microsoft Teams](../channels/msteams.md) 中。

### IRC

IRC 由扩展支持，在 `channels.irc` 下配置。

```json
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

-   此处涵盖的核心键路径：`channels.irc`、`channels.irc.dmPolicy`、`channels.irc.configWrites`、`channels.irc.nickserv.*`。
-   可选的 `channels.irc.defaultAccount` 覆盖默认账号选择，当它匹配一个已配置的账号 ID 时。
-   完整的 IRC 频道配置（主机/端口/TLS/频道/允许列表/提及门控）记录在 [IRC](../channels/irc.md) 中。

### 多账号（所有频道）

每个频道运行多个账号（每个都有自己的 `accountId`）：

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   `default` 在省略 `accountId` 时使用（CLI + 路由）。
-   环境变量令牌仅适用于**默认**账号。
-   基础频道设置适用于所有账号，除非每个账号有覆盖。
-   使用 `bindings[].match.accountId` 将每个账号路由到不同的代理。
-   如果您在仍使用单账号顶层频道配置时通过 `openclaw channels add`（或频道引导）添加非默认账号，OpenClaw 会先将账号作用域的顶层单账号值移动到 `channels..accounts.default`，以便原始账号继续工作。
-   现有的仅频道绑定（无 `accountId`）继续匹配默认账号；账号作用域的绑定仍然是可选的。
-   `openclaw doctor --fix` 也会修复混合形状，当命名账号存在但 `default` 缺失时，将账号作用域的顶层单账号值移动到 `accounts.default`。

### 其他扩展频道

许多扩展频道配置为 `channels.`，并在其专用频道页面中记录（例如飞书、Matrix、LINE、Nostr、Zalo、Nextcloud Talk、Synology Chat 和 Twitch）。参见完整频道索引：[频道](../channels.md)。

### 群组聊天提及门控

群组消息默认**需要提及**（元数据提及或正则表达式模式）。适用于 WhatsApp、Telegram、Discord、Google Chat 和 iMessage 群组聊天。**提及类型：**

-   **元数据提及**：原生平台 @-提及。在 WhatsApp 自聊模式下忽略。
-   **文本模式**：`agents.list[].groupChat.mentionPatterns` 中的正则表达式模式。始终检查。
-   仅当检测可能时（原生提及或至少一个模式）才强制执行提及门控。

```json
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` 设置全局默认值。频道可以使用 `channels..historyLimit`（或每个账号）覆盖。设置为 `0` 以禁用。

#### 私信历史限制

```json
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

解析顺序：每个私信覆盖 → 提供商默认值 → 无限制（全部保留）。支持：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

#### 自聊模式

在 `allowFrom` 中包含您自己的号码以启用自聊模式（忽略原生 @-提及，仅响应文本模式）：

```json
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### 命令（聊天命令处理）

```json
{
  commands: {
    native: "auto", // 支持时注册原生命令
    text: true, // 在聊天消息中解析 /commands
    bash: false, // 允许 ! (别名: /bash)
    bashForegroundMs: 2000,
    config: false, // 允许 /config
    debug: false, // 允许 /debug
    restart: false, // 允许 /restart + 网关重启工具
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   文本命令必须是**独立**消息，以 `/` 开头。
-   `native: "auto"` 为 Discord/Telegram 开启原生命令，Slack 保持关闭。
-   每个频道覆盖：`channels.discord.commands.native`（布尔值或 `"auto"`）。`false` 清除先前注册的命令。
-   `channels.telegram.customCommands` 添加额外的 Telegram 机器人菜单条目。
-   `bash: true` 为主机 shell 启用 `! `。需要 `tools.elevated.enabled` 且发送者在 `tools.elevated.allowFrom.` 中。
-   `config: true` 启用 `/config`（读取/写入 `openclaw.json`）。对于网关 `chat.send` 客户端，持久的 `/config set|unset` 写入还需要 `operator.admin`；只读的 `/config show` 仍对普通写入作用域的运营商客户端可用。
-   `channels..configWrites` 按频道控制配置变更（默认：true）。
-   `allowFrom` 是按提供商的。设置后，它是**唯一**的授权来源（忽略频道允许列表/配对和 `useAccessGroups`）。
-   `useAccessGroups: false` 允许命令在 `allowFrom` 未设置时绕过访问组策略。

* * *

## 代理默认值

### agents.defaults.workspace

默认值：`~/.openclaw/workspace`。

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### agents.defaults.repoRoot

可选的仓库根目录，显示在系统提示的 Runtime 行中。如果未设置，OpenClaw 会从工作空间向上遍历自动检测。

```json
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### agents.defaults.skipBootstrap

禁用自动创建工作空间引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`）。

```json
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### agents.defaults.bootstrapMaxChars

工作空间引导文件在截断前的最大字符数。默认值：`20000`。

```json
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### agents.defaults.bootstrapTotalMaxChars

所有工作空间引导文件注入的最大总字符数。默认值：`150000`。

```json
{
  agents: { defaults: { bootstrapTotalMaxChars: 150000 } },
}
```

### agents.defaults.bootstrapPromptTruncationWarning

控制引导上下文被截断时代理可见的警告文本。默认值：`"once"`。

-   `"off"`：从不将警告文本注入系统提示。
-   `"once"`：每个唯一的截断签名注入一次警告（推荐）。
-   `"always"`：每次存在截断时都注入警告。

```json
{
  agents: { defaults: { bootstrapPromptTruncationWarning: "once" } }, // off | once | always
}
```

### agents.defaults.imageMaxDimensionPx

在提供商调用之前，转录/工具图像块中最长边的最大像素尺寸。默认值：`1200`。较低的值通常减少视觉令牌使用和截图密集型运行的请求负载大小。较高的值保留更多视觉细节。

```json
{
  agents: { defaults: { imageMaxDimensionPx: 1200 } },
}
```

### agents.defaults.userTimezone

系统提示上下文的时区（非消息时间戳）。回退到主机时区。

```json
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### agents.defaults.timeFormat

系统提示中的时间格式。默认值：`auto`（操作系统偏好）。

```json
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### agents.defaults.model

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax