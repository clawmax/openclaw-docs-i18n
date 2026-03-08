

  消息平台

  
# Matrix

Matrix 是一个开放、去中心化的消息协议。OpenClaw 作为 Matrix **用户**连接到任何家庭服务器，因此你需要为机器人准备一个 Matrix 账户。登录后，你可以直接私信机器人或邀请它加入房间（Matrix 的“群组”）。Beeper 也是一个有效的客户端选项，但它需要启用 E2EE。状态：通过插件支持 (@vector-im/matrix-bot-sdk)。支持私信、房间、线程、媒体、反应、投票（发送 + 投票开始作为文本）、位置和 E2EE（需要加密支持）。

## 需要插件

Matrix 作为插件提供，不包含在核心安装包中。通过 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/matrix
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/matrix
```

如果你在配置/引导过程中选择 Matrix 并且检测到 git 检出，OpenClaw 将自动提供本地安装路径。详情：[插件](../tools/plugin.md)

## 设置

1.  安装 Matrix 插件：
    -   从 npm：`openclaw plugins install @openclaw/matrix`
    -   从本地检出：`openclaw plugins install ./extensions/matrix`
2.  在家庭服务器上创建 Matrix 账户：
    -   浏览托管选项：[https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
    -   或自行托管。
3.  获取机器人账户的访问令牌：
    
    -   在你的家庭服务器上使用 `curl` 调用 Matrix 登录 API：
    
    复制
    
    ```bash
    curl --request POST \
      --url https://matrix.example.org/_matrix/client/v3/login \
      --header 'Content-Type: application/json' \
      --data '{
      "type": "m.login.password",
      "identifier": {
        "type": "m.id.user",
        "user": "your-user-name"
      },
      "password": "your-password"
    }'
    ```
    
    -   将 `matrix.example.org` 替换为你的家庭服务器 URL。
    -   或者设置 `channels.matrix.userId` + `channels.matrix.password`：OpenClaw 调用相同的登录端点，将访问令牌存储在 `~/.openclaw/credentials/matrix/credentials.json` 中，并在下次启动时重用。
4.  配置凭据：
    -   环境变量：`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (或 `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
    -   或配置文件：`channels.matrix.*`
    -   如果两者都设置，配置文件优先。
    -   使用访问令牌时：用户 ID 通过 `/whoami` 自动获取。
    -   设置时，`channels.matrix.userId` 应为完整的 Matrix ID（例如：`@bot:example.org`）。
5.  重启网关（或完成引导）。
6.  从任何 Matrix 客户端（Element、Beeper 等；参见 [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)）与机器人开始私信或邀请它加入房间。Beeper 需要 E2EE，因此设置 `channels.matrix.encryption: true` 并验证设备。

最小配置（访问令牌，用户 ID 自动获取）：

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

E2EE 配置（启用端到端加密）：

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## 加密 (E2EE)

端到端加密通过 Rust 加密 SDK **支持**。使用 `channels.matrix.encryption: true` 启用：

-   如果加密模块加载成功，加密房间将自动解密。
-   发送到加密房间的出站媒体将被加密。
-   首次连接时，OpenClaw 会请求你的其他会话进行设备验证。
-   在另一个 Matrix 客户端（Element 等）中验证设备以启用密钥共享。
-   如果加密模块无法加载，E2EE 将被禁用，加密房间将无法解密；OpenClaw 会记录警告。
-   如果看到缺少加密模块的错误（例如，`@matrix-org/matrix-sdk-crypto-nodejs-*`），请允许 `@matrix-org/matrix-sdk-crypto-nodejs` 的构建脚本，并运行 `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` 或使用 `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` 获取二进制文件。

加密状态按账户 + 访问令牌存储在 `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/`（SQLite 数据库）。同步状态存储在旁边的 `bot-storage.json` 中。如果访问令牌（设备）发生变化，将创建一个新的存储，并且必须重新验证机器人才能访问加密房间。**设备验证：** 启用 E2EE 后，机器人将在启动时请求你的其他会话进行验证。打开 Element（或其他客户端）并批准验证请求以建立信任。验证后，机器人可以解密加密房间中的消息。

## 多账户

多账户支持：使用 `channels.matrix.accounts` 配置每个账户的凭据和可选的 `name`。参见 [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 了解共享模式。每个账户作为独立的 Matrix 用户在任何家庭服务器上运行。每个账户的配置继承自顶层 `channels.matrix` 设置，并可以覆盖任何选项（DM 策略、群组、加密等）。

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "主助手",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "警报机器人",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

注意：

-   账户启动是串行化的，以避免并发模块导入时的竞争条件。
-   环境变量（`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` 等）仅适用于**默认**账户。
-   基础频道设置（DM 策略、群组策略、提及门控等）适用于所有账户，除非在单个账户中被覆盖。
-   使用 `bindings[].match.accountId` 将每个账户路由到不同的代理。
-   加密状态按账户 + 访问令牌存储（每个账户有独立的密钥存储）。

## 路由模型

-   回复始终返回 Matrix。
-   私信共享代理的主会话；房间映射到群组会话。

## 访问控制（私信）

-   默认：`channels.matrix.dm.policy = "pairing"`。未知发送者会收到配对码。
-   通过以下方式批准：
    -   `openclaw pairing list matrix`
    -   `openclaw pairing approve matrix `
-   公开私信：`channels.matrix.dm.policy="open"` 加上 `channels.matrix.dm.allowFrom=["*"]`。
-   `channels.matrix.dm.allowFrom` 接受完整的 Matrix 用户 ID（例如：`@user:server`）。当目录搜索找到单个精确匹配时，向导会将显示名称解析为用户 ID。
-   不要使用显示名称或纯本地部分（例如：`"Alice"` 或 `"alice"`）。它们是模糊的，并且在允许列表匹配时会被忽略。使用完整的 `@user:server` ID。

## 房间（群组）

-   默认：`channels.matrix.groupPolicy = "allowlist"`（提及门控）。使用 `channels.defaults.groupPolicy` 在未设置时覆盖默认值。
-   运行时注意：如果 `channels.matrix` 完全缺失，运行时对于房间检查会回退到 `groupPolicy="allowlist"`（即使设置了 `channels.defaults.groupPolicy`）。
-   使用 `channels.matrix.groups` 允许列表房间（房间 ID 或别名；当目录搜索找到单个精确匹配时，名称会解析为 ID）：

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

-   `requireMention: false` 在该房间中启用自动回复。
-   `groups."*"` 可以设置跨房间提及门控的默认值。
-   `groupAllowFrom` 限制哪些发送者可以在房间中触发机器人（完整的 Matrix 用户 ID）。
-   每个房间的 `users` 允许列表可以进一步限制特定房间内的发送者（使用完整的 Matrix 用户 ID）。
-   配置向导会提示输入房间允许列表（房间 ID、别名或名称），并且仅在精确、唯一的匹配时解析名称。
-   启动时，OpenClaw 将允许列表中的房间/用户名称解析为 ID 并记录映射关系；未解析的条目在允许列表匹配时被忽略。
-   邀请默认自动加入；通过 `channels.matrix.autoJoin` 和 `channels.matrix.autoJoinAllowlist` 控制。
-   要**禁止所有房间**，设置 `channels.matrix.groupPolicy: "disabled"`（或保持允许列表为空）。
-   旧版键：`channels.matrix.rooms`（与 `groups` 形状相同）。

## 线程

-   支持回复线程。
-   `channels.matrix.threadReplies` 控制回复是否保持在线程中：
    -   `off`, `inbound` (默认), `always`
-   `channels.matrix.replyToMode` 控制不在线程中回复时的回复元数据：
    -   `off` (默认), `first`, `all`

## 功能

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 房间 | ✅ 支持 |
| 线程 | ✅ 支持 |
| 媒体 | ✅ 支持 |
| E2EE | ✅ 支持（需要加密模块） |
| 反应 | ✅ 支持（通过工具发送/读取） |
| 投票 | ✅ 发送支持；入站投票开始转换为文本（响应/结束忽略） |
| 位置 | ✅ 支持（geo URI；海拔忽略） |
| 原生命令 | ✅ 支持 |

## 故障排除

首先运行此阶梯：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后根据需要确认私信配对状态：

```bash
openclaw pairing list matrix
```

常见故障：

-   已登录但房间消息被忽略：房间被 `groupPolicy` 或房间允许列表阻止。
-   私信被忽略：当 `channels.matrix.dm.policy="pairing"` 时，发送者待批准。
-   加密房间失败：加密支持或加密设置不匹配。

故障排除流程：[/channels/troubleshooting](./troubleshooting.md)。

## 配置参考 (Matrix)

完整配置：[配置](../gateway/configuration.md) 提供者选项：

-   `channels.matrix.enabled`：启用/禁用频道启动。
-   `channels.matrix.homeserver`：家庭服务器 URL。
-   `channels.matrix.userId`：Matrix 用户 ID（使用访问令牌时可选）。
-   `channels.matrix.accessToken`：访问令牌。
-   `channels.matrix.password`：登录密码（令牌存储）。
-   `channels.matrix.deviceName`：设备显示名称。
-   `channels.matrix.encryption`：启用 E2EE（默认：false）。
-   `channels.matrix.initialSyncLimit`：初始同步限制。
-   `channels.matrix.threadReplies`：`off | inbound | always`（默认：inbound）。
-   `channels.matrix.textChunkLimit`：出站文本分块大小（字符）。
-   `channels.matrix.chunkMode`：`length`（默认）或 `newline` 以在长度分块前按空行（段落边界）分割。
-   `channels.matrix.dm.policy`：`pairing | allowlist | open | disabled`（默认：pairing）。
-   `channels.matrix.dm.allowFrom`：私信允许列表（完整的 Matrix 用户 ID）。`open` 需要 `"*"`。向导在可能时将名称解析为 ID。
-   `channels.matrix.groupPolicy`：`allowlist | open | disabled`（默认：allowlist）。
-   `channels.matrix.groupAllowFrom`：群组消息的允许列表发送者（完整的 Matrix 用户 ID）。
-   `channels.matrix.allowlistOnly`：强制私信和房间的允许列表规则。
-   `channels.matrix.groups`：群组允许列表 + 每个房间的设置映射。
-   `channels.matrix.rooms`：旧版群组允许列表/配置。
-   `channels.matrix.replyToMode`：线程/标签的回复模式。
-   `channels.matrix.mediaMaxMb`：入站/出站媒体上限（MB）。
-   `channels.matrix.autoJoin`：邀请处理（`always | allowlist | off`，默认：always）。
-   `channels.matrix.autoJoinAllowlist`：自动加入允许的房间 ID/别名。
-   `channels.matrix.accounts`：多账户配置，按键为账户 ID（每个账户继承顶层设置）。
-   `channels.matrix.actions`：每个操作的工具门控（反应/消息/置顶/成员信息/频道信息）。

[LINE](./line.md)[Mattermost](./mattermost.md)