

  消息平台

  
# Nextcloud Talk

状态：通过插件支持（webhook 机器人）。支持私聊、房间、消息反应和 Markdown 格式消息。

## 所需插件

Nextcloud Talk 作为插件提供，不包含在核心安装包中。通过 CLI（npm 仓库）安装：

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

如果在配置/引导过程中选择了 Nextcloud Talk 并且检测到 git 检出，OpenClaw 将自动提供本地安装路径。详情请见：[插件](../tools/plugin.md)

## 快速设置（新手）

1.  安装 Nextcloud Talk 插件。
2.  在你的 Nextcloud 服务器上，创建一个机器人：
    
    复制
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  在目标房间设置中启用机器人。
4.  配置 OpenClaw：
    -   配置文件：`channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   或环境变量：`NEXTCLOUD_TALK_BOT_SECRET`（仅限默认账户）
5.  重启网关（或完成引导）。

最小配置：

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## 注意事项

-   机器人无法发起私聊。用户必须先向机器人发送消息。
-   Webhook URL 必须能被网关访问；如果位于代理后面，请设置 `webhookPublicUrl`。
-   机器人 API 不支持媒体文件上传；媒体文件以 URL 形式发送。
-   Webhook 负载不区分私聊和房间；设置 `apiUser` + `apiPassword` 以启用房间类型查询（否则私聊会被当作房间处理）。

## 访问控制（私聊）

-   默认：`channels.nextcloud-talk.dmPolicy = "pairing"`。未知发送者会收到一个配对码。
-   通过以下方式批准：
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   公开私聊：`channels.nextcloud-talk.dmPolicy="open"` 加上 `channels.nextcloud-talk.allowFrom=["*"]`。
-   `allowFrom` 仅匹配 Nextcloud 用户 ID；显示名称会被忽略。

## 房间（群组）

-   默认：`channels.nextcloud-talk.groupPolicy = "allowlist"`（提及门控）。
-   使用 `channels.nextcloud-talk.rooms` 设置房间允许列表：

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   若要禁止所有房间，请保持允许列表为空或设置 `channels.nextcloud-talk.groupPolicy="disabled"`。

## 功能支持

| 功能 | 状态 |
| --- | --- |
| 私聊 | 支持 |
| 房间 | 支持 |
| 话题 | 不支持 |
| 媒体 | 仅限 URL |
| 消息反应 | 支持 |
| 原生命令 | 不支持 |

## 配置参考（Nextcloud Talk）

完整配置：[配置](../gateway/configuration.md) 提供者选项：

-   `channels.nextcloud-talk.enabled`：启用/禁用通道启动。
-   `channels.nextcloud-talk.baseUrl`：Nextcloud 实例 URL。
-   `channels.nextcloud-talk.botSecret`：机器人共享密钥。
-   `channels.nextcloud-talk.botSecretFile`：密钥文件路径。
-   `channels.nextcloud-talk.apiUser`：用于房间查询（私聊检测）的 API 用户。
-   `channels.nextcloud-talk.apiPassword`：用于房间查询的 API/应用密码。
-   `channels.nextcloud-talk.apiPasswordFile`：API 密码文件路径。
-   `channels.nextcloud-talk.webhookPort`：webhook 监听端口（默认：8788）。
-   `channels.nextcloud-talk.webhookHost`：webhook 主机（默认：0.0.0.0）。
-   `channels.nextcloud-talk.webhookPath`：webhook 路径（默认：/nextcloud-talk-webhook）。
-   `channels.nextcloud-talk.webhookPublicUrl`：外部可访问的 webhook URL。
-   `channels.nextcloud-talk.dmPolicy`：`pairing | allowlist | open | disabled`。
-   `channels.nextcloud-talk.allowFrom`：私聊允许列表（用户 ID）。`open` 需要 `"*"`。
-   `channels.nextcloud-talk.groupPolicy`：`allowlist | open | disabled`。
-   `channels.nextcloud-talk.groupAllowFrom`：群组允许列表（用户 ID）。
-   `channels.nextcloud-talk.rooms`：每个房间的设置和允许列表。
-   `channels.nextcloud-talk.historyLimit`：群组历史消息限制（0 表示禁用）。
-   `channels.nextcloud-talk.dmHistoryLimit`：私聊历史消息限制（0 表示禁用）。
-   `channels.nextcloud-talk.dms`：每个私聊的覆盖设置（historyLimit）。
-   `channels.nextcloud-talk.textChunkLimit`：出站文本分块大小（字符数）。
-   `channels.nextcloud-talk.chunkMode`：`length`（默认）或 `newline` 以在按长度分块前按空行（段落边界）分割。
-   `channels.nextcloud-talk.blockStreaming`：为此通道禁用块流式传输。
-   `channels.nextcloud-talk.blockStreamingCoalesce`：块流式传输合并调优。
-   `channels.nextcloud-talk.mediaMaxMb`：入站媒体文件大小上限（MB）。

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)

---