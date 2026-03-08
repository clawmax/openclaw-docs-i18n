

  消息平台

  
# Tlon

Tlon 是一个基于 Urbit 构建的去中心化消息平台。OpenClaw 可以连接到您的 Urbit 飞船，并响应私信和群组聊天消息。默认情况下，群组回复需要 @ 提及，并且可以通过允许列表进一步限制。状态：通过插件支持。支持私信、群组提及、主题回复、富文本格式和图片上传。暂不支持反应和投票。

## 需要插件

Tlon 作为插件提供，不包含在核心安装包中。通过 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/tlon
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/tlon
```

详情：[插件](../tools/plugin.md)

## 设置

1.  安装 Tlon 插件。
2.  获取您的飞船 URL 和登录码。
3.  配置 `channels.tlon`。
4.  重启网关。
5.  向机器人发送私信或在群组频道中提及它。

最小配置（单账户）：

```json
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // 推荐：您的主飞船，始终允许
    },
  },
}
```

## 私有/局域网飞船

默认情况下，OpenClaw 会阻止私有/内部主机名和 IP 范围以进行 SSRF 保护。如果您的飞船运行在私有网络（localhost、局域网 IP 或内部主机名）上，您必须明确选择加入：

```json
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

这适用于以下类型的 URL：

-   `http://localhost:8080`
-   `http://192.168.x.x:8080`
-   `http://my-ship.local:8080`

⚠️ 仅在您信任本地网络时启用此设置。此设置会禁用对您飞船 URL 请求的 SSRF 保护。

## 群组频道

默认启用自动发现。您也可以手动固定频道：

```json
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

禁用自动发现：

```json
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## 访问控制

私信允许列表（空 = 不允许私信，使用 `ownerShip` 进行审批流程）：

```json
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

群组授权（默认受限）：

```json
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## 所有者与审批系统

设置一个所有者飞船，以便在未经授权的用户尝试交互时接收审批请求：

```json
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

所有者飞船**在所有地方都自动获得授权**——私信邀请会自动接受，频道消息始终允许。您无需将所有者添加到 `dmAllowlist` 或 `defaultAuthorizedShips`。设置后，所有者会收到以下情况的私信通知：

-   来自允许列表之外的飞船的私信请求
-   在未经授权的频道中被提及
-   群组邀请请求

## 自动接受设置

自动接受私信邀请（针对 `dmAllowlist` 中的飞船）：

```json
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

自动接受群组邀请：

```json
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
    },
  },
}
```

## 投递目标（CLI/cron）

与 `openclaw message send` 或 cron 投递一起使用：

-   私信：`~sampel-palnet` 或 `dm/~sampel-palnet`
-   群组：`chat/~host-ship/channel` 或 `group:~host-ship/channel`

## 捆绑技能

Tlon 插件包含一个捆绑技能 ([`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill))，提供对 Tlon 操作的 CLI 访问：

-   **联系人**：获取/更新个人资料，列出联系人
-   **频道**：列出、创建、发布消息、获取历史记录
-   **群组**：列出、创建、管理成员
-   **私信**：发送消息，对消息做出反应
-   **反应**：向帖子和私信添加/移除表情符号反应
-   **设置**：通过斜杠命令管理插件权限

安装插件后，该技能自动可用。

## 功能支持

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 群组/频道 | ✅ 支持（默认需要提及） |
| 主题 | ✅ 支持（在主题内自动回复） |
| 富文本 | ✅ Markdown 转换为 Tlon 格式 |
| 图片 | ✅ 上传到 Tlon 存储 |
| 反应 | ✅ 通过[捆绑技能](#捆绑技能) |
| 投票 | ❌ 暂不支持 |
| 原生命令 | ✅ 支持（默认仅限所有者） |

## 故障排除

首先运行以下排查步骤：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

常见故障：

-   **私信被忽略**：发送者不在 `dmAllowlist` 中且未配置 `ownerShip` 进行审批流程。
-   **群组消息被忽略**：频道未被发现或发送者未获授权。
-   **连接错误**：检查飞船 URL 是否可达；对于本地飞船，启用 `allowPrivateNetwork`。
-   **认证错误**：验证登录码是否最新（登录码会轮换）。

## 配置参考

完整配置：[配置](../gateway/configuration.md) 提供者选项：

-   `channels.tlon.enabled`：启用/禁用频道启动。
-   `channels.tlon.ship`：机器人的 Urbit 飞船名称（例如 `~sampel-palnet`）。
-   `channels.tlon.url`：飞船 URL（例如 `https://sampel-palnet.tlon.network`）。
-   `channels.tlon.code`：飞船登录码。
-   `channels.tlon.allowPrivateNetwork`：允许 localhost/局域网 URL（绕过 SSRF）。
-   `channels.tlon.ownerShip`：用于审批系统的所有者飞船（始终授权）。
-   `channels.tlon.dmAllowlist`：允许发送私信的飞船（空 = 无）。
-   `channels.tlon.autoAcceptDmInvites`：自动接受来自允许列表飞船的私信邀请。
-   `channels.tlon.autoAcceptGroupInvites`：自动接受所有群组邀请。
-   `channels.tlon.autoDiscoverChannels`：自动发现群组频道（默认：true）。
-   `channels.tlon.groupChannels`：手动固定的频道嵌套路径。
-   `channels.tlon.defaultAuthorizedShips`：对所有频道授权的飞船。
-   `channels.tlon.authorization.channelRules`：按频道授权规则。
-   `channels.tlon.showModelSignature`：在消息后附加模型名称。

## 注意事项

-   群组回复需要提及（例如 `~your-bot-ship`）才能响应。
-   主题回复：如果传入消息在主题内，OpenClaw 会在主题内回复。
-   富文本：Markdown 格式（粗体、斜体、代码、标题、列表）会转换为 Tlon 原生格式。
-   图片：URL 会上传到 Tlon 存储并作为图片块嵌入。

[Telegram](./telegram.md)[Twitch](./twitch.md)