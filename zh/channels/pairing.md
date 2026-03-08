

  配置

  
# 配对

“配对”是 OpenClaw 明确的**所有者批准**步骤。它在两个场景中使用：

1.  **私信配对**（谁可以与机器人对话）
2.  **节点配对**（哪些设备/节点可以加入网关网络）

安全背景：[安全](../gateway/security.md)

## 1) 私信配对（入站聊天访问）

当频道配置了私信策略 `pairing` 时，未知发送者会收到一个短代码，并且他们的消息在您批准前**不会被处理**。默认私信策略文档位于：[安全](../gateway/security.md) 配对代码：

-   8 个字符，大写，不含易混淆字符（`0O1I`）。
-   **1 小时后过期**。机器人仅在新请求创建时发送配对消息（大致每个发送者每小时一次）。
-   默认情况下，待处理的私信配对请求上限为**每个频道 3 个**；额外的请求将被忽略，直到其中一个过期或被批准。

### 批准发送者

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

支持的频道：`telegram`、`whatsapp`、`signal`、`imessage`、`discord`、`slack`、`feishu`。

### 状态存储位置

存储在 `~/.openclaw/credentials/` 目录下：

-   待处理请求：`-pairing.json`
-   已批准的允许列表存储：
    -   默认账户：`-allowFrom.json`
    -   非默认账户：`--allowFrom.json`

账户作用域行为：

-   非默认账户仅读写其作用域内的允许列表文件。
-   默认账户使用频道作用域的非作用域允许列表文件。

请将这些文件视为敏感文件（它们控制着对您助手的访问权限）。

## 2) 节点设备配对（iOS/Android/macOS/无头节点）

节点以 `role: node` 的**设备**身份连接到网关。网关会创建一个设备配对请求，必须批准该请求。

### 通过 Telegram 配对（推荐用于 iOS）

如果您使用 `device-pair` 插件，您可以完全通过 Telegram 完成首次设备配对：

1.  在 Telegram 中，向您的机器人发送消息：`/pair`
2.  机器人会回复两条消息：一条指令消息和一条单独的**设置代码**消息（便于在 Telegram 中复制/粘贴）。
3.  在您的手机上，打开 OpenClaw iOS 应用 → 设置 → 网关。
4.  粘贴设置代码并连接。
5.  回到 Telegram：`/pair approve`

设置代码是一个 base64 编码的 JSON 载荷，包含：

-   `url`：网关 WebSocket URL（`ws://...` 或 `wss://...`）
-   `token`：一个短期有效的配对令牌

在设置代码有效期内，请像对待密码一样对待它。

### 批准节点设备

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### 节点配对状态存储

存储在 `~/.openclaw/devices/` 目录下：

-   `pending.json`（短期有效；待处理请求会过期）
-   `paired.json`（已配对的设备 + 令牌）

### 注意事项

-   遗留的 `node.pair.*` API（CLI：`openclaw nodes pending/approve`）是一个独立的网关自有配对存储。WebSocket 节点仍然需要进行设备配对。

## 相关文档

-   安全模型 + 提示注入：[安全](../gateway/security.md)
-   安全更新（运行 doctor）：[更新](../install/updating.md)
-   频道配置：
    -   Telegram：[Telegram](./telegram.md)
    -   WhatsApp：[WhatsApp](./whatsapp.md)
    -   Signal：[Signal](./signal.md)
    -   BlueBubbles (iMessage)：[BlueBubbles](./bluebubbles.md)
    -   iMessage (旧版)：[iMessage](./imessage.md)
    -   Discord：[Discord](./discord.md)
    -   Slack：[Slack](./slack.md)

[Zalo 个人版](./zalouser.md)[群组消息](./group-messages.md)