

  消息传递平台

  
# Microsoft Teams

> “进入此地者，当放弃一切希望。”

更新日期：2026-01-21 状态：支持文本 + 私聊附件；频道/群组文件发送需要 `sharePointSiteId` + Graph 权限（参见[在群聊中发送文件](#sending-files-in-group-chats)）。投票通过 Adaptive Cards 发送。

## 所需插件

Microsoft Teams 作为插件提供，不包含在核心安装包中。**破坏性变更 (2026.1.15)：** MS Teams 已从核心中移出。如果您使用它，必须安装插件。解释：保持核心安装更轻量，并让 MS Teams 依赖项可以独立更新。通过 CLI (npm 注册表) 安装：

```bash
openclaw plugins install @openclaw/msteams
```

本地检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/msteams
```

如果您在配置/引导过程中选择了 Teams 并且检测到 git 检出，OpenClaw 将自动提供本地安装路径。详情：[插件](../tools/plugin.md)

## 快速设置（初学者）

1.  安装 Microsoft Teams 插件。
2.  创建一个 **Azure Bot**（应用 ID + 客户端密钥 + 租户 ID）。
3.  使用这些凭据配置 OpenClaw。
4.  通过公共 URL 或隧道暴露 `/api/messages`（默认端口 3978）。
5.  安装 Teams 应用包并启动网关。

最小配置：

```json
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

注意：群聊默认被阻止 (`channels.msteams.groupPolicy: "allowlist"`)。要允许群组回复，请设置 `channels.msteams.groupAllowFrom`（或使用 `groupPolicy: "open"` 允许任何成员，但需提及门控）。

## 目标

-   通过 Teams 私聊、群聊或频道与 OpenClaw 对话。
-   保持路由确定性：回复始终返回到消息来源的频道。
-   默认采用安全的频道行为（除非另行配置，否则需要提及）。

## 配置写入

默认情况下，Microsoft Teams 允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。通过以下方式禁用：

```json
{
  channels: { msteams: { configWrites: false } },
}
```

## 访问控制（私聊 + 群组）

**私聊访问**

-   默认：`channels.msteams.dmPolicy = "pairing"`。未知发件人在批准前将被忽略。
-   `channels.msteams.allowFrom` 应使用稳定的 AAD 对象 ID。
-   UPN/显示名称是可变的；直接匹配默认禁用，仅在 `channels.msteams.dangerouslyAllowNameMatching: true` 时启用。
-   当凭据允许时，向导可以通过 Microsoft Graph 将名称解析为 ID。

**群组访问**

-   默认：`channels.msteams.groupPolicy = "allowlist"`（除非您添加 `groupAllowFrom`，否则被阻止）。当未设置时，使用 `channels.defaults.groupPolicy` 覆盖默认值。
-   `channels.msteams.groupAllowFrom` 控制哪些发件人可以在群聊/频道中触发（回退到 `channels.msteams.allowFrom`）。
-   设置 `groupPolicy: "open"` 以允许任何成员（默认仍受提及门控）。
-   要**禁止所有频道**，请设置 `channels.msteams.groupPolicy: "disabled"`。

示例：

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + 频道允许列表**

-   通过将团队和频道列在 `channels.msteams.teams` 下来限定群组/频道回复的范围。
-   键可以是团队 ID 或名称；频道键可以是对话 ID 或名称。
-   当 `groupPolicy="allowlist"` 且存在团队允许列表时，仅接受列出的团队/频道（提及门控）。
-   配置向导接受 `Team/Channel` 条目并为您存储。
-   启动时，OpenClaw 将团队/频道和用户允许列表名称解析为 ID（当 Graph 权限允许时）并记录映射关系；未解析的条目保持键入时的原样。

示例：

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## 工作原理

1.  安装 Microsoft Teams 插件。
2.  创建一个 **Azure Bot**（应用 ID + 密钥 + 租户 ID）。
3.  构建一个引用该机器人并包含以下 RSC 权限的 **Teams 应用包**。
4.  将 Teams 应用上传/安装到团队中（或用于私聊的个人范围）。
5.  在 `~/.openclaw/openclaw.json`（或环境变量）中配置 `msteams` 并启动网关。
6.  网关默认监听 `/api/messages` 上的 Bot Framework webhook 流量。

## Azure Bot 设置（先决条件）

在配置 OpenClaw 之前，您需要创建一个 Azure Bot 资源。

### 步骤 1：创建 Azure Bot

1.  前往 [创建 Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2.  填写 **基础** 选项卡：
    
    | 字段 | 值 |
    | --- | --- |
    | **机器人句柄** | 您的机器人名称，例如 `openclaw-msteams`（必须唯一） |
    | **订阅** | 选择您的 Azure 订阅 |
    | **资源组** | 新建或使用现有 |
    | **定价层** | **免费** 用于开发/测试 |
    | **应用类型** | **单租户**（推荐 - 参见下面的说明） |
    | **创建类型** | **创建新的 Microsoft 应用 ID** |
    

> **弃用通知：** 2025-07-31 之后已弃用创建新的多租户机器人。对于新机器人，请使用**单租户**。

3.  点击 **查看 + 创建** → **创建**（等待约 1-2 分钟）

### 步骤 2：获取凭据

1.  前往您的 Azure Bot 资源 → **配置**
2.  复制 **Microsoft 应用 ID** → 这是您的 `appId`
3.  点击 **管理密码** → 转到应用注册
4.  在 **证书和密码** → **新建客户端密码** → 复制 **值** → 这是您的 `appPassword`
5.  前往 **概述** → 复制 **目录（租户）ID** → 这是您的 `tenantId`

### 步骤 3：配置消息传递端点

1.  在 Azure Bot → **配置**
2.  将 **消息传递端点** 设置为您的 webhook URL：
    -   生产环境：`https://your-domain.com/api/messages`
    -   本地开发：使用隧道（参见下面的[本地开发（隧道）](#local-development-tunneling)）

### 步骤 4：启用 Teams 频道

1.  在 Azure Bot → **频道**
2.  点击 **Microsoft Teams** → 配置 → 保存
3.  接受服务条款

## 本地开发（隧道）

Teams 无法访问 `localhost`。使用隧道进行本地开发：**选项 A：ngrok**

```bash
ngrok http 3978
# 复制 https URL，例如 https://abc123.ngrok.io
# 将消息传递端点设置为：https://abc123.ngrok.io/api/messages
```

**选项 B：Tailscale Funnel**

```bash
tailscale funnel 3978
# 使用您的 Tailscale funnel URL 作为消息传递端点
```

## Teams 开发者门户（替代方案）

您可以不使用手动创建清单 ZIP，而是使用 [Teams 开发者门户](https://dev.teams.microsoft.com/apps)：

1.  点击 **\+ 新建应用**
2.  填写基本信息（名称、描述、开发者信息）
3.  转到 **应用功能** → **机器人**
4.  选择 **手动输入机器人 ID** 并粘贴您的 Azure Bot 应用 ID
5.  检查范围：**个人**、**团队**、**群聊**
6.  点击 **分发** → **下载应用包**
7.  在 Teams 中：**应用** → **管理您的应用** → **上传自定义应用** → 选择 ZIP 文件

这通常比手动编辑 JSON 清单更容易。

## 测试机器人

**选项 A：Azure Web Chat（首先验证 webhook）**

1.  在 Azure 门户 → 您的 Azure Bot 资源 → **在 Web Chat 中测试**
2.  发送一条消息 - 您应该会看到回复
3.  这确认了您的 webhook 端点在 Teams 设置之前正常工作

**选项 B：Teams（应用安装后）**

1.  安装 Teams 应用（旁加载或组织目录）
2.  在 Teams 中找到机器人并发送私聊消息
3.  检查网关日志以查看传入活动

## 设置（最小化，仅文本）

1.  **安装 Microsoft Teams 插件**
    -   从 npm：`openclaw plugins install @openclaw/msteams`
    -   从本地检出：`openclaw plugins install ./extensions/msteams`
2.  **机器人注册**
    -   创建一个 Azure Bot（见上文）并记录：
        -   应用 ID
        -   客户端密钥（应用密码）
        -   租户 ID（单租户）
3.  **Teams 应用清单**
    -   包含一个 `bot` 条目，其中 `botId = `。
    -   范围：`personal`、`team`、`groupChat`。
    -   `supportsFiles: true`（个人范围文件处理所需）。
    -   添加 RSC 权限（见下文）。
    -   创建图标：`outline.png` (32x32) 和 `color.png` (192x192)。
    -   将所有三个文件打包成 ZIP：`manifest.json`、`outline.png`、`color.png`。
4.  **配置 OpenClaw**
    
    复制
    
    ```json
    {
      "msteams": {
        "enabled": true,
        "appId": "<APP_ID>",
        "appPassword": "<APP_PASSWORD>",
        "tenantId": "<TENANT_ID>",
        "webhook": { "port": 3978, "path": "/api/messages" }
      }
    }
    ```
    
    您也可以使用环境变量代替配置键：
    -   `MSTEAMS_APP_ID`
    -   `MSTEAMS_APP_PASSWORD`
    -   `MSTEAMS_TENANT_ID`
5.  **机器人端点**
    -   将 Azure Bot 消息传递端点设置为：
        -   `https://:3978/api/messages`（或您选择的路径/端口）。
6.  **运行网关**
    -   当插件已安装且 `msteams` 配置存在凭据时，Teams 频道会自动启动。

## 历史上下文

-   `channels.msteams.historyLimit` 控制有多少最近的频道/群组消息被包装到提示中。
-   回退到 `messages.groupChat.historyLimit`。设置为 `0` 以禁用（默认 50）。
-   私聊历史可以通过 `channels.msteams.dmHistoryLimit`（用户轮次）限制。每个用户的覆盖：`channels.msteams.dms["<user_id>"].historyLimit`。

## 当前 Teams RSC 权限（清单）

这些是我们的 Teams 应用清单中**现有的资源特定权限**。它们仅适用于安装应用的团队/聊天内部。**对于频道（团队范围）：**

-   `ChannelMessage.Read.Group` (Application) - 无需 @提及即可接收所有频道消息
-   `ChannelMessage.Send.Group` (Application)
-   `Member.Read.Group` (Application)
-   `Owner.Read.Group` (Application)
-   `ChannelSettings.Read.Group` (Application)
-   `TeamMember.Read.Group` (Application)
-   `TeamSettings.Read.Group` (Application)

**对于群聊：**

-   `ChatMessage.Read.Chat` (Application) - 无需 @提及即可接收所有群聊消息

## Teams 清单示例（已编辑）

最小化、有效的示例，包含必填字段。替换 ID 和 URL。

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### 清单注意事项（必填字段）

-   `bots[].botId` **必须** 与 Azure Bot 应用 ID 匹配。
-   `webApplicationInfo.id` **必须** 与 Azure Bot 应用 ID 匹配。
-   `bots[].scopes` 必须包含您计划使用的界面（`personal`、`team`、`groupChat`）。
-   `bots[].supportsFiles: true` 是个人范围文件处理所必需的。
-   如果您需要频道流量，`authorization.permissions.resourceSpecific` 必须包含频道读取/发送权限。

### 更新现有应用

要更新已安装的 Teams 应用（例如，添加 RSC 权限）：

1.  使用新设置更新您的 `manifest.json`
2.  **递增 `version` 字段**（例如，`1.0.0` → `1.1.0`）
3.  **重新打包**清单和图标（`manifest.json`、`outline.png`、`color.png`）
4.  上传新的 zip 文件：
    -   **选项 A (Teams 管理中心)：** Teams 管理中心 → Teams 应用 → 管理应用 → 找到您的应用 → 上传新版本
    -   **选项 B (旁加载)：** 在 Teams 中 → 应用 → 管理您的应用 → 上传自定义应用
5.  **对于团队频道：** 在每个团队中重新安装应用以使新权限生效
6.  **完全退出并重新启动 Teams**（不仅仅是关闭窗口）以清除缓存的应用元数据

## 能力：仅 RSC 与 Graph

### 仅使用 Teams RSC（应用已安装，无 Graph API 权限）

有效：

-   读取频道消息**文本**内容。
-   发送频道消息**文本**内容。
-   接收**个人（私聊）** 文件附件。

无效：

-   频道/群组**图像或文件内容**（有效负载仅包含 HTML 存根）。
-   下载存储在 SharePoint/OneDrive 中的附件。
-   读取消息历史记录（超出实时 webhook 事件）。

### 使用 Teams RSC + Microsoft Graph 应用程序权限

增加：

-   下载托管内容（粘贴到消息中的图像）。
-   下载存储在 SharePoint/OneDrive 中的文件附件。
-   通过 Graph 读取频道/聊天消息历史记录。

### RSC 与 Graph API

| 能力 | RSC 权限 | Graph API |
| --- | --- | --- |
| **实时消息** | 是（通过 webhook） | 否（仅轮询） |
| **历史消息** | 否 | 是（可以查询历史记录） |
| **设置复杂性** | 仅应用清单 | 需要管理员同意 + 令牌流程 |
| **离线工作** | 否（必须运行） | 是（随时查询） |

**结论：** RSC 用于实时监听；Graph API 用于历史访问。要在离线时补上错过的消息，您需要具有 `ChannelMessage.Read.All` 的 Graph API（需要管理员同意）。

## Graph 启用的媒体 + 历史记录（频道所需）

如果您需要**频道**中的图像/文件，或想要获取**消息历史记录**，必须启用 Microsoft Graph 权限并授予管理员同意。

1.  在 Entra ID (Azure AD) **应用注册**中，添加 Microsoft Graph **应用程序权限**：
    -   `ChannelMessage.Read.All`（频道附件 + 历史记录）
    -   `Chat.Read.All` 或 `ChatMessage.Read.All`（群聊）
2.  **授予管理员同意**给租户。
3.  提升 Teams 应用**清单版本**，重新上传，并**在 Teams 中重新安装应用**。
4.  **完全退出并重新启动 Teams** 以清除缓存的应用元数据。

**用户提及的额外权限：** 对于对话中的用户，用户 @提及开箱即用。但是，如果您想要动态搜索并提及**不在当前对话中**的用户，请添加 `User.Read.All` (Application) 权限并授予管理员同意。

## 已知限制

### Webhook 超时

Teams 通过 HTTP webhook 传递消息。如果处理时间过长（例如，LLM 响应缓慢），您可能会看到：

-   网关超时
-   Teams 重试消息（导致重复）
-   回复丢失

OpenClaw 通过快速返回并主动发送回复来处理此问题，但非常慢的响应仍可能导致问题。

### 格式

Teams 的 Markdown 比 Slack 或 Discord 更有限：

-   基本格式有效：**粗体**、*斜体*、`代码`、链接
-   复杂的 Markdown（表格、嵌套列表）可能无法正确渲染
-   支持 Adaptive Cards 用于投票和任意卡片发送（见下文）

## 配置

关键设置（共享频道模式请参见 `/gateway/configuration`）：

-   `channels.msteams.enabled`：启用/禁用频道。
-   `channels.msteams.appId`、`channels.msteams.appPassword`、`channels.msteams.tenantId`：机器人凭据。
-   `channels.msteams.webhook.port`（默认 `3978`）
-   `channels.msteams.webhook.path`（默认 `/api/messages`）
-   `channels.msteams.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）
-   `channels.msteams.allowFrom`：私聊允许列表（推荐使用 AAD 对象 ID）。当 Graph 访问可用时，向导在设置期间将名称解析为 ID。
-   `channels.msteams.dangerouslyAllowNameMatching`：紧急切换开关，用于重新启用可变的 UPN/显示名称匹配。
-   `channels.msteams.textChunkLimit`：出站文本分块大小。
-   `channels.msteams.chunkMode`：`length`（默认）或 `newline` 以在长度分块之前按空行（段落边界）拆分。
-   `channels.msteams.mediaAllowHosts`：入站附件主机允许列表（默认为 Microsoft/Teams 域）。
-   `channels.msteams.mediaAuthAllowHosts`：用于在媒体重试时附加 Authorization 头的主机允许列表（默认为 Graph + Bot Framework 主机）。
-   `channels.msteams.requireMention`：在频道/群组中需要 @提及（默认 true）。
-   `channels.msteams.replyStyle`：`thread | top-level`（参见[回复样式：线程与帖子](#reply-style-threads-vs-posts)）。
-   `channels.msteams.teams..replyStyle`：每个团队的覆盖。
-   `channels.msteams.teams..requireMention`：每个团队的覆盖。
-   `channels.msteams.teams..tools`：当缺少频道覆盖时使用的默认每个团队工具策略覆盖（`allow`/`deny`/`alsoAllow`）。
-   `channels.msteams.teams..toolsBySender`：默认每个团队每个发件人工具策略覆盖（支持 `"*"` 通配符）。
-   `channels.msteams.teams..channels..replyStyle`：每个频道的覆盖。
-   `channels.msteams.teams..channels..requireMention`：每个频道的覆盖。
-   `channels.msteams.teams..channels..tools`：每个频道工具策略覆盖（`allow`/`deny`/`alsoAllow`）。
-   `channels.msteams.teams..channels..toolsBySender`：每个频道每个发件人工具策略覆盖（支持 `"*"` 通配符）。
-   `toolsBySender` 键应使用明确的前缀：`id:`、`e164:`、`username:`、`name:`（遗留的无前缀键仍仅映射到 `id:`）。
-   `channels.msteams.sharePointSiteId`：用于群聊/频道中文件上传的 SharePoint 站点 ID（参见[在群聊中发送文件](#sending-files-in-group-chats)）。

## 路由和会话

-   会话键遵循标准代理格式（参见 [/concepts/session](../concepts/session.md)）：
    -   直接消息共享主会话（`agent::`）。
    -   频道/群组消息使用对话 ID：
        -   `agent::msteams:channel:`
        -   `agent::msteams:group:`

## 回复样式：线程与帖子

Teams 最近引入了两种频道 UI 样式，基于相同的基础数据模型：

| 样式 | 描述 | 推荐的 `replyStyle` |
| --- | --- | --- |
| **帖子**（经典） | 消息显示为卡片，下方有线程回复 | `thread`（默认） |
| **线程**（类似 Slack） | 消息线性流动，更像 Slack | `top-level` |

**问题：** Teams API 不公开频道使用的 UI 样式。如果您使用了错误的 `replyStyle`：

-   在 Threads 样式的频道中使用 `thread` → 回复会尴尬地嵌套显示
-   在 Posts 样式的频道中使用 `top-level` → 回复显示为单独的顶级帖子，而不是在帖子线程内

**解决方案：** 根据频道的设置方式，为每个频道配置 `replyStyle`：

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## 附件和图像

**当前限制：**

-   **私聊：** 通过 Teams 机器人文件 API，图像和文件附件有效。
-   **频道/群组：** 附件存储在 M365 存储（SharePoint/OneDrive）中。webhook 有效负载仅包含 HTML 存根，而不是实际的文件字节。**需要 Graph API 权限**才能下载频道附件。

没有 Graph 权限时，带有图像的频道消息将仅作为纯文本接收（图像内容对机器人不可访问）。默认情况下，OpenClaw 仅从 Microsoft/Teams 主机名下载媒体。使用 `channels.msteams.mediaAllowHosts` 覆盖（使用 `["*"]` 允许任何主机）。Authorization 头仅附加给 `channels.msteams.mediaAuthAllowHosts` 中的主机（默认为 Graph + Bot Framework 主机）。保持此列表严格（避免多租户后缀）。

## 在群聊中发送文件

机器人可以使用 FileConsentCard 流程（内置）在私聊中发送文件。但是，**在群聊/频道中发送文件**需要额外的设置：

| 上下文 | 文件发送方式 | 所需设置 |
| --- | --- | --- |
| **私聊** | FileConsentCard → 用户接受 → 机器人上传 | 开箱即用 |
| **群聊/频道** | 上传到 SharePoint → 共享链接 | 需要 `sharePointSiteId` + Graph 权限 |
| **图像（任何上下文）** | Base64 编码内联 | 开箱即用 |

### 为什么群聊需要 SharePoint

机器人没有个人 OneDrive 驱动器（`/me/drive` Graph API 端点不适用于应用程序身份）。要在群聊/频道中发送文件，机器人需上传到 **SharePoint 站点** 并创建共享链接。

### 设置

1.  **在 Entra ID (Azure AD) → 应用注册中添加 Graph API 权限：**
    -   `Sites.ReadWrite.All` (Application) - 上传文件到 SharePoint
    -   `Chat.Read.All` (Application) - 可选，启用每个用户的共享链接
2.  **为租户授予管理员同意**。
3.  **获取您的 SharePoint 站点 ID：**
    
    复制
    
    ```bash
    # 通过 Graph Explorer 或带有有效令牌的 curl：
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"
    
    # 示例：对于位于 "contoso.sharepoint.com/sites/BotFiles" 的站点
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"
    
    # 响应包含："id": "contoso.sharepoint.com,guid1,guid2"
    ```
    
4.  **配置 OpenClaw：**
    
    复制
    
    ```json
    {
      channels: {
        msteams: {
          // ... 其他配置 ...
          sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
        },
      },
    }
    ```
    

### 共享行为

| 权限 | 共享行为 |
| --- | --- |
| 仅 `Sites.ReadWrite.All` | 组织范围的共享链接（组织内的任何人都可以访问） |
| `Sites.ReadWrite.All` + `Chat.Read.All` | 每个用户的共享链接（仅聊天成员可以访问） |

每个用户的共享更安全，因为只有聊天参与者可以访问文件。如果缺少 `Chat.Read.All` 权限，机器人将回退到组织范围的共享。

### 回退行为

| 场景 | 结果 |
| --- | --- |
| 群聊 + 文件 + 配置了 `sharePointSiteId` | 上传到 SharePoint，发送共享链接 |
| 群聊 + 文件 + 无 `sharePointSiteId` | 尝试 OneDrive 上传（可能失败），仅发送文本 |
| 个人聊天 + 文件 | FileConsentCard 流程（无需 SharePoint 即可工作） |
| 任何上下文 + 图像 | Base64 编码内联（无需 SharePoint 即可工作） |

### 文件存储位置

上传的文件存储在配置的 SharePoint 站点的默认文档库中的 `/OpenClawShared/` 文件夹内。

## 投票（Adaptive Cards）

OpenClaw 将 Teams 投票作为 Adaptive Cards 发送（没有原生的 Teams 投票 API）。

-   CLI：`openclaw message poll --channel msteams --target conversation: ...`
-   投票由网关记录在 `~/.openclaw/msteams-polls.json` 中。
-   网关必须保持在线以记录投票。
-   投票目前不会自动发布结果摘要（如果需要，请检查存储文件）。

## Adaptive Cards（任意）

使用 `message` 工具或 CLI 向 Teams 用户或对话发送任何 Adaptive Card JSON。`card` 参数接受一个 Adaptive Card JSON 对象。当提供 `card` 时，消息文本是可选的。**代理工具：**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello!" }]
  }
}
```

**CLI：**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

有关卡片模式和示例，请参见 [Adaptive Cards 文档](https://adaptivecards.io/)。有关目标格式详情，请参见下面的[目标格式](#target-formats)。

## 目标格式

MSTeams 目标使用前缀来区分用户和对话：

| 目标类型 | 格式 | 示例 |
| --- | --- | --- |
| 用户（按 ID） | `user:<aad-object-id>` | `user:40a1a0ed-4ff2-4164-a219-55518990c197` |
| 用户（按名称） | `user:<display-name>` | `user:John Smith`（需要 Graph API） |
| 群组/频道 | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2` |
| 群组/频道（原始） | `<conversation-id>` | `19:abc123...@thread.tacv2`（如果包含 `@thread`） |

**CLI 示例：**

```bash
# 按 ID 发送给用户
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# 按显示名称发送给用户（触发 Graph API 查找）
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# 发送到群聊或频道
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# 发送 Adaptive Card 到对话
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**代理工具示例：**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello" }]
  }
}
```

注意：没有 `user:` 前缀时，名称默认为群组/团队解析。当按显示名称定位人员时，请始终使用 `user:`。

## 主动消息传递

-   主动消息仅在**用户交互之后**才可能，因为我们在那时存储了对话引用。
-   有关 `dmPolicy` 和允许列表门控，请参见 `/gateway/configuration`。

## 团队和频道 ID（常见陷阱）

Teams URL 中的 `groupId` 查询参数**不是**用于配置的团队 ID。请从 URL 路径中提取 ID：**团队 URL：**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    团队 ID（URL 解码此部分）
```

**频道 URL：**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      频道 ID（URL 解码此部分）
```

**用于配置：**

-   团队 ID = `/team/` 之后的路径段（URL 解码，例如 `19:Bk4j...@thread.tacv2`）
-   频道 ID = `/channel/` 之后的路径段（URL 解码）
-   **忽略** `groupId` 查询参数

## 私有频道

机器人在私有频道中的支持有限：

| 功能 | 标准频道 | 私有频道 |
| --- | --- | --- |
| 机器人安装 | 是 | 有限 |
| 实时消息（webhook） | 是 | 可能无效 |
| RSC 权限 | 是 | 可能行为不同 |
| @提及 | 是 | 如果机器人可访问 |
| Graph API 历史记录 | 是 | 是（有权限） |

**如果私有频道无效的解决方法：**

1.  使用标准频道进行机器人交互
2.  使用私聊 - 用户始终可以直接向机器人发送消息
3.  使用 Graph API 进行历史访问（需要 `ChannelMessage.Read.All`）

## 故障排除

### 常见问题

-   **频道中图像不显示：** Graph 权限或管理员同意缺失。重新安装 Teams 应用并完全退出/重新打开 Teams。
-   **频道中无响应：