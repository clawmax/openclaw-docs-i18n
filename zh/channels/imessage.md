

  消息平台

  
# iMessage

> **⚠️** 对于新的 iMessage 部署，请使用 [BlueBubbles](./bluebubbles.md)。`imsg` 集成是旧版功能，可能在未来的版本中被移除。

 状态：旧版外部 CLI 集成。网关生成 `imsg rpc` 进程并通过 stdio 上的 JSON-RPC 进行通信（无单独的守护进程/端口）。

## 快速设置

## 要求和权限 (macOS)

-   运行 `imsg` 的 Mac 上必须已登录 Messages。
-   运行 OpenClaw/`imsg` 的进程上下文需要“完全磁盘访问”权限（用于访问 Messages 数据库）。
-   需要通过 Messages.app 发送消息的“自动化”权限。

> **💡** 权限是按进程上下文授予的。如果网关以无头模式运行（LaunchAgent/SSH），请在同一上下文中运行一次交互式命令以触发提示：
> 
> 复制
> 
> ```
> imsg chats --limit 1
> # 或
> imsg send <handle> "test"
> ```

## 访问控制和路由

`channels.imessage.dmPolicy` 控制直接消息：

-   `pairing`（默认）
-   `allowlist`
-   `open`（要求 `allowFrom` 包含 `"*"`）
-   `disabled`

允许列表字段：`channels.imessage.allowFrom`。允许列表条目可以是句柄或聊天目标（`chat_id:*`、`chat_guid:*`、`chat_identifier:*`）。

`channels.imessage.groupPolicy` 控制群组处理：

-   `allowlist`（配置时的默认值）
-   `open`
-   `disabled`

群组发送者允许列表：`channels.imessage.groupAllowFrom`。运行时回退：如果未设置 `groupAllowFrom`，iMessage 群组发送者检查将在可用时回退到 `allowFrom`。运行时说明：如果完全缺少 `channels.imessage` 配置，运行时将回退到 `groupPolicy="allowlist"` 并记录警告（即使设置了 `channels.defaults.groupPolicy`）。群组的提及门控：

-   iMessage 没有原生的提及元数据
-   提及检测使用正则表达式模式（`agents.list[].groupChat.mentionPatterns`，回退到 `messages.groupChat.mentionPatterns`）
-   如果没有配置模式，则无法强制执行提及门控

来自授权发送者的控制命令可以绕过群组中的提及门控。

-   私信使用直接路由；群组使用群组路由。
-   使用默认的 `session.dmScope=main` 时，iMessage 私信会合并到代理主会话中。
-   群组会话是隔离的（`agent::imessage:group:<chat_id>`）。
-   回复使用原始通道/目标元数据路由回 iMessage。

类群组线程行为：一些多参与者的 iMessage 线程可能以 `is_group=false` 到达。如果该 `chat_id` 在 `channels.imessage.groups` 下被显式配置，OpenClaw 会将其视为群组流量（群组门控 + 群组会话隔离）。

## 部署模式

使用专用的 Apple ID 和 macOS 用户，以便机器人流量与您的个人 Messages 配置文件隔离。典型流程：

1.  创建/登录一个专用的 macOS 用户。
2.  在该用户中使用机器人 Apple ID 登录 Messages。
3.  在该用户中安装 `imsg`。
4.  创建 SSH 包装器，以便 OpenClaw 可以在该用户上下文中运行 `imsg`。
5.  将 `channels.imessage.accounts..cliPath` 和 `.dbPath` 指向该用户配置文件。

首次运行可能需要在机器人用户会话中进行 GUI 批准（自动化和完全磁盘访问）。

常见拓扑：

-   网关在 Linux/VM 上运行
-   iMessage + `imsg` 在您的 tailnet 中的 Mac 上运行
-   `cliPath` 包装器使用 SSH 运行 `imsg`
-   `remoteHost` 启用 SCP 附件获取

示例：

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

使用 SSH 密钥，以便 SSH 和 SCP 都是非交互式的。首先确保主机密钥受信任（例如 `ssh bot@mac-mini.tailnet-1234.ts.net`），以便 `known_hosts` 被填充。

iMessage 支持在 `channels.imessage.accounts` 下进行每个账户的配置。每个账户可以覆盖字段，例如 `cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb`、历史记录设置和附件根目录允许列表。

## 媒体、分块和发送目标

-   入站附件摄取是可选的：`channels.imessage.includeAttachments`
-   当设置了 `remoteHost` 时，可以通过 SCP 获取远程附件路径
-   附件路径必须匹配允许的根目录：
    -   `channels.imessage.attachmentRoots`（本地）
    -   `channels.imessage.remoteAttachmentRoots`（远程 SCP 模式）
    -   默认根目录模式：`/Users/*/Library/Messages/Attachments`
-   SCP 使用严格的主机密钥检查（`StrictHostKeyChecking=yes`）
-   出站媒体大小使用 `channels.imessage.mediaMaxMb`（默认 16 MB）

-   文本分块限制：`channels.imessage.textChunkLimit`（默认 4000）
-   分块模式：`channels.imessage.chunkMode`
    -   `length`（默认）
    -   `newline`（优先按段落分割）

首选显式目标：

-   `chat_id:123`（推荐用于稳定路由）
-   `chat_guid:...`
-   `chat_identifier:...`

也支持句柄目标：

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## 配置写入

iMessage 默认允许通道发起的配置写入（用于 `/config set|unset`，当 `commands.config: true` 时）。禁用：

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## 故障排除

验证二进制文件和 RPC 支持：

```bash
imsg rpc --help
openclaw channels status --probe
```

如果探测报告 RPC 不受支持，请更新 `imsg`。

检查：

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   配对批准（`openclaw pairing list imessage`）

检查：

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   `channels.imessage.groups` 允许列表行为
-   提及模式配置（`agents.list[].groupChat.mentionPatterns`）

检查：

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   来自网关主机的 SSH/SCP 密钥认证
-   主机密钥存在于网关主机的 `~/.ssh/known_hosts` 中
-   运行 Messages 的 Mac 上远程路径的可读性

在同一用户/会话上下文的交互式 GUI 终端中重新运行并批准提示：

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

确认运行 OpenClaw/`imsg` 的进程上下文已授予完全磁盘访问和自动化权限。

## 配置参考指针

-   [配置参考 - iMessage](../gateway/configuration-reference.md#imessage)
-   [网关配置](../gateway/configuration.md)
-   [配对](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)