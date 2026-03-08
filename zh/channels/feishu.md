

  消息平台

  
# 飞书

飞书 (Lark) 是一个团队聊天平台，被公司用于消息传递和协作。此插件通过平台的 WebSocket 事件订阅将 OpenClaw 连接到飞书/Lark 机器人，从而无需暴露公共 Webhook URL 即可接收消息。

* * *

## 所需插件

安装飞书插件：

```bash
openclaw plugins install @openclaw/feishu
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## 快速开始

有两种方式添加飞书频道：

### 方法 1：引导向导（推荐）

如果您刚刚安装 OpenClaw，请运行向导：

```bash
openclaw onboard
```

该向导将引导您完成：

1.  创建飞书应用并收集凭证
2.  在 OpenClaw 中配置应用凭证
3.  启动网关

✅ **配置完成后**，检查网关状态：

-   `openclaw gateway status`
-   `openclaw logs --follow`

### 方法 2：CLI 设置

如果您已完成初始安装，通过 CLI 添加频道：

```bash
openclaw channels add
```

选择 **Feishu**，然后输入 App ID 和 App Secret。✅ **配置完成后**，管理网关：

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## 步骤 1：创建飞书应用

### 1. 打开飞书开放平台

访问 [飞书开放平台](https://open.feishu.cn/app) 并登录。Lark（国际版）租户应使用 [https://open.larksuite.com/app](https://open.larksuite.com/app) 并在飞书配置中设置 `domain: "lark"`。

### 2. 创建应用

1.  点击 **创建企业自建应用**
2.  填写应用名称和描述
3.  选择一个应用图标

![创建企业自建应用](../images/channels-feishu-step2-create-app.png.md)

### 3. 复制凭证

在 **凭证与基础信息** 中，复制：

-   **App ID**（格式：`cli_xxx`）
-   **App Secret**

❗ **重要：** 请妥善保管 App Secret。![获取凭证](../images/channels-feishu-step3-credentials.png.md)

### 4. 配置权限

在 **权限管理** 页面，点击 **批量导入** 并粘贴：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![配置权限](../images/channels-feishu-step4-permissions.png.md)

### 5. 启用机器人能力

在 **应用功能** > **机器人**：

1.  启用机器人能力
2.  设置机器人名称

![启用机器人能力](../images/channels-feishu-step5-bot-capability.png.md)

### 6. 配置事件订阅

⚠️ **重要：** 在设置事件订阅之前，请确保：

1.  您已为飞书运行了 `openclaw channels add`
2.  网关正在运行（`openclaw gateway status`）

在 **事件订阅** 中：

1.  选择 **使用长连接接收事件** (WebSocket)
2.  添加事件：`im.message.receive_v1`

⚠️ 如果网关未运行，长连接设置可能无法保存。![配置事件订阅](../images/channels-feishu-step6-event-subscription.png.md)

### 7. 发布应用

1.  在 **版本管理与发布** 中创建版本
2.  提交审核并发布
3.  等待管理员批准（企业自建应用通常会自动批准）

* * *

## 步骤 2：配置 OpenClaw

### 使用向导配置（推荐）

```bash
openclaw channels add
```

选择 **Feishu** 并粘贴您的 App ID 和 App Secret。

### 通过配置文件配置

编辑 `~/.openclaw/openclaw.json`：

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

如果您使用 `connectionMode: "webhook"`，请设置 `verificationToken`。飞书 Webhook 服务器默认绑定到 `127.0.0.1`；仅在您确实需要不同的绑定地址时才设置 `webhookHost`。

#### 校验令牌（Webhook 模式）

使用 Webhook 模式时，请在配置中设置 `channels.feishu.verificationToken`。获取该值的方法：

1.  在飞书开放平台，打开您的应用
2.  进入 **开发配置** → **事件与回调**
3.  打开 **加密策略** 标签页
4.  复制 **校验令牌**

![校验令牌位置](../images/channels-feishu-verification-token.png.md)

### 通过环境变量配置

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark（国际版）域名

如果您的租户在 Lark（国际版）上，请将域名设置为 `lark`（或完整的域名字符串）。您可以在 `channels.feishu.domain` 或每个账户（`channels.feishu.accounts..domain`）中设置。

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### 配额优化标志

您可以使用两个可选标志来减少飞书 API 调用：

-   `typingIndicator`（默认 `true`）：当为 `false` 时，跳过“正在输入”状态调用。
-   `resolveSenderNames`（默认 `true`）：当为 `false` 时，跳过发送者资料查询调用。

可以在顶层或每个账户中设置：

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## 步骤 3：启动 + 测试

### 1. 启动网关

```bash
openclaw gateway
```

### 2. 发送测试消息

在飞书中，找到您的机器人并发送一条消息。

### 3. 批准配对

默认情况下，机器人会回复一个配对码。批准它：

```bash
openclaw pairing approve feishu <CODE>
```

批准后，即可正常聊天。

* * *

## 概述

-   **飞书机器人频道**：由网关管理的飞书机器人
-   **确定性路由**：回复始终返回飞书
-   **会话隔离**：私聊共享一个主会话；群聊是隔离的
-   **WebSocket 连接**：通过飞书 SDK 建立长连接，无需公共 URL

* * *

## 访问控制

### 私聊

-   **默认**：`dmPolicy: "pairing"`（未知用户会收到配对码）
-   **批准配对**：
    
    复制
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **允许列表模式**：设置 `channels.feishu.allowFrom` 并填入允许的 Open ID

### 群聊

**1. 群聊策略** (`channels.feishu.groupPolicy`)：

-   `"open"` = 允许群内所有人（默认）
-   `"allowlist"` = 仅允许 `groupAllowFrom` 中的群
-   `"disabled"` = 禁用群消息

**2. 提及要求** (`channels.feishu.groups.<chat_id>.requireMention`)：

-   `true` = 需要 @提及（默认）
-   `false` = 无需提及即可回复

* * *

## 群聊配置示例

### 允许所有群聊，需要 @提及（默认）

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // 默认 requireMention: true
    },
  },
}
```

### 允许所有群聊，无需 @提及

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### 仅允许特定群聊

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // 飞书群聊 ID (chat_id) 格式如：oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### 限制群聊中哪些发送者可以发送消息（发送者允许列表）

除了允许群聊本身之外，**该群聊中的所有消息**都受发送者 open_id 的限制：只有列在 `groups.<chat_id>.allowFrom` 中的用户的消息才会被处理；来自其他成员的消息将被忽略（这是发送者级别的完全限制，不仅限于控制命令如 /reset 或 /new）。

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // 飞书用户 ID (open_id) 格式如：ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## 获取群聊/用户 ID

### 群聊 ID (chat_id)

群聊 ID 格式如 `oc_xxx`。**方法 1（推荐）**

1.  启动网关并在群聊中 @提及机器人
2.  运行 `openclaw logs --follow` 并查找 `chat_id`

**方法 2** 使用飞书 API 调试器列出群聊。

### 用户 ID (open_id)

用户 ID 格式如 `ou_xxx`。**方法 1（推荐）**

1.  启动网关并向机器人发送私聊消息
2.  运行 `openclaw logs --follow` 并查找 `open_id`

**方法 2** 检查配对请求中的用户 Open ID：

```bash
openclaw pairing list feishu
```

* * *

## 常用命令

| 命令 | 描述 |
| --- | --- |
| `/status` | 显示机器人状态 |
| `/reset` | 重置会话 |
| `/model` | 显示/切换模型 |

> 注意：飞书尚不支持原生命令菜单，因此命令必须以文本形式发送。

## 网关管理命令

| 命令 | 描述 |
| --- | --- |
| `openclaw gateway status` | 显示网关状态 |
| `openclaw gateway install` | 安装/启动网关服务 |
| `openclaw gateway stop` | 停止网关服务 |
| `openclaw gateway restart` | 重启网关服务 |
| `openclaw logs --follow` | 跟踪网关日志 |

* * *

## 故障排除

### 机器人在群聊中不响应

1.  确保机器人已添加到群聊
2.  确保您 @提及了机器人（默认行为）
3.  检查 `groupPolicy` 未设置为 `"disabled"`
4.  检查日志：`openclaw logs --follow`

### 机器人未收到消息

1.  确保应用已发布并获批准
2.  确保事件订阅包含 `im.message.receive_v1`
3.  确保 **长连接** 已启用
4.  确保应用权限完整
5.  确保网关正在运行：`openclaw gateway status`
6.  检查日志：`openclaw logs --follow`

### App Secret 泄露

1.  在飞书开放平台重置 App Secret
2.  在您的配置中更新 App Secret
3.  重启网关

### 消息发送失败

1.  确保应用拥有 `im:message:send_as_bot` 权限
2.  确保应用已发布
3.  检查日志获取详细错误信息

* * *

## 高级配置

### 多账户

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` 控制当出站 API 未明确指定 `accountId` 时使用哪个飞书账户。

### 消息限制

-   `textChunkLimit`：出站文本分块大小（默认：2000 字符）
-   `mediaMaxMb`：媒体上传/下载限制（默认：30MB）

### 流式传输

飞书通过交互式卡片支持流式回复。启用后，机器人在生成文本时会更新卡片。

```json
{
  channels: {
    feishu: {
      streaming: true, // 启用流式卡片输出（默认 true）
      blockStreaming: true, // 启用块级流式传输（默认 true）
    },
  },
}
```

设置 `streaming: false` 以等待完整回复后再发送。

### 多智能体路由

使用 `bindings` 将飞书私聊或群聊路由到不同的智能体。

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

路由字段：

-   `match.channel`: `"feishu"`
-   `match.peer.kind`: `"direct"` 或 `"group"`
-   `match.peer.id`: 用户 Open ID (`ou_xxx`) 或群聊 ID (`oc_xxx`)

查看 [获取群聊/用户 ID](#get-groupuser-ids) 获取查找技巧。

* * *

## 配置参考

完整配置：[网关配置](../gateway/configuration.md) 关键选项：

| 设置 | 描述 | 默认值 |
| --- | --- | --- |
| `channels.feishu.enabled` | 启用/禁用频道 | `true` |
| `channels.feishu.domain` | API 域名 (`feishu` 或 `lark`) | `feishu` |
| `channels.feishu.connectionMode` | 事件传输模式 | `websocket` |
| `channels.feishu.defaultAccount` | 出站路由的默认账户 ID | `default` |
| `channels.feishu.verificationToken` | Webhook 模式必需 | \- |
| `channels.feishu.webhookPath` | Webhook 路由路径 | `/feishu/events` |
| `channels.feishu.webhookHost` | Webhook 绑定主机 | `127.0.0.1` |
| `channels.feishu.webhookPort` | Webhook 绑定端口 | `3000` |
| `channels.feishu.accounts..appId` | App ID | \- |
| `channels.feishu.accounts..appSecret` | App Secret | \- |
| `channels.feishu.accounts..domain` | 每个账户的 API 域名覆盖 | `feishu` |
| `channels.feishu.dmPolicy` | 私聊策略 | `pairing` |
| `channels.feishu.allowFrom` | 私聊允许列表 (open_id 列表) | \- |
| `channels.feishu.groupPolicy` | 群聊策略 | `open` |
| `channels.feishu.groupAllowFrom` | 群聊允许列表 | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | 需要 @提及 | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | 启用群聊 | `true` |
| `channels.feishu.textChunkLimit` | 消息分块大小 | `2000` |
| `channels.feishu.mediaMaxMb` | 媒体大小限制 | `30` |
| `channels.feishu.streaming` | 启用流式卡片输出 | `true` |
| `channels.feishu.blockStreaming` | 启用块级流式传输 | `true` |

* * *

## dmPolicy 参考

| 值 | 行为 |
| --- | --- |
| `"pairing"` | **默认。** 未知用户会收到配对码；必须批准 |
| `"allowlist"` | 仅允许 `allowFrom` 中的用户聊天 |
| `"open"` | 允许所有用户（需要在 allowFrom 中包含 `"*"`） |
| `"disabled"` | 禁用私聊 |

* * *

## 支持的消息类型

### 接收

-   ✅ 文本
-   ✅ 富文本（帖子）
-   ✅ 图片
-   ✅ 文件
-   ✅ 音频
-   ✅ 视频
-   ✅ 表情

### 发送

-   ✅ 文本
-   ✅ 图片
-   ✅ 文件
-   ✅ 音频
-   ⚠️ 富文本（部分支持）

[Discord](./discord.md)[Google Chat](./googlechat.md)