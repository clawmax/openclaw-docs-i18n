

  消息平台

  
# IRC

当您希望 OpenClaw 加入经典频道 (`#room`) 和直接消息时，请使用 IRC。IRC 作为扩展插件提供，但其配置在主配置文件的 `channels.irc` 下进行。

## 快速开始

1.  在 `~/.openclaw/openclaw.json` 中启用 IRC 配置。
2.  至少设置：

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  启动/重启网关：

```bash
openclaw gateway run
```

## 安全默认值

-   `channels.irc.dmPolicy` 默认为 `"pairing"`。
-   `channels.irc.groupPolicy` 默认为 `"allowlist"`。
-   当 `groupPolicy="allowlist"` 时，设置 `channels.irc.groups` 来定义允许的频道。
-   使用 TLS (`channels.irc.tls=true`)，除非您有意接受明文传输。

## 访问控制

IRC 频道有两个独立的“门控”：

1.  **频道访问** (`groupPolicy` + `groups`)：机器人是否接受来自某个频道的任何消息。
2.  **发送者访问** (`groupAllowFrom` / 每个频道的 `groups["#channel"].allowFrom`)：谁被允许在该频道内触发机器人。

配置键：

-   DM 允许列表 (DM 发送者访问)：`channels.irc.allowFrom`
-   群组发送者允许列表 (频道发送者访问)：`channels.irc.groupAllowFrom`
-   每个频道的控制 (频道 + 发送者 + 提及规则)：`channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` 允许未配置的频道 (**默认情况下仍受提及门控限制**)

允许列表条目应使用稳定的发送者身份 (`nick!user@host`)。仅昵称匹配是可变的，并且仅在 `channels.irc.dangerouslyAllowNameMatching: true` 时启用。

### 常见陷阱：allowFrom 用于 DM，而非频道

如果您看到如下日志：

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

…这意味着发送者未被允许发送**群组/频道**消息。通过以下任一方式修复：

-   设置 `channels.irc.groupAllowFrom` (对所有频道全局生效)，或
-   设置每个频道的发送者允许列表：`channels.irc.groups["#channel"].allowFrom`

示例 (允许 `#tuirc-dev` 中的任何人向机器人发送消息)：

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 回复触发 (提及)

即使频道被允许 (通过 `groupPolicy` + `groups`) 且发送者被允许，OpenClaw 在群组上下文中默认采用**提及门控**。这意味着您可能会看到类似 `drop channel … (missing-mention)` 的日志，除非消息包含与机器人匹配的提及模式。要使机器人在 IRC 频道中回复**而无需提及**，请为该频道禁用提及门控：

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

或者，允许**所有** IRC 频道 (无需每个频道的允许列表) 并且仍然无需提及即可回复：

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## 安全说明 (推荐用于公共频道)

如果您在公共频道中允许 `allowFrom: ["*"]`，任何人都可以提示机器人。为降低风险，请限制该频道的工具。

### 频道内所有人使用相同的工具

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 不同发送者使用不同工具 (所有者获得更多权限)

使用 `toolsBySender` 对 `"*"` 应用更严格的策略，对您的昵称应用更宽松的策略：

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

注意：

-   `toolsBySender` 的键应使用 `id:` 作为 IRC 发送者身份值：`id:eigen` 或 `id:eigen!~eigen@174.127.248.171` 以进行更强匹配。
-   仍接受未加前缀的旧键，并仅作为 `id:` 进行匹配。
-   第一个匹配的发送者策略生效；`"*"` 是通配符回退。

有关群组访问与提及门控 (以及它们如何交互) 的更多信息，请参阅：[/channels/groups](./groups.md)。

## NickServ

要在连接后向 NickServ 验证身份：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

连接时可选一次性注册：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

昵称注册后，请禁用 `register` 以避免重复的 REGISTER 尝试。

## 环境变量

默认账户支持：

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS` (逗号分隔)
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## 故障排除

-   如果机器人已连接但从不回复频道消息，请验证 `channels.irc.groups` **以及**提及门控是否正在丢弃消息 (`missing-mention`)。如果您希望它无需 ping 即可回复，请为该频道设置 `requireMention:false`。
-   如果登录失败，请验证昵称可用性和服务器密码。
-   如果在自定义网络上 TLS 失败，请验证主机/端口和证书设置。

[iMessage](./imessage.md)[LINE](./line.md)