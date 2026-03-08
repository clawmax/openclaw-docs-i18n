

  消息平台

  
# Google Chat

状态：已就绪，支持通过 Google Chat API webhook（仅 HTTP）进行私聊和群组空间。

## 快速设置（新手）

1.  创建一个 Google Cloud 项目并启用 **Google Chat API**。
    -   前往：[Google Chat API 凭据](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   如果尚未启用，请启用该 API。
2.  创建一个 **服务账号**：
    -   点击 **创建凭据** > **服务账号**。
    -   为其命名（例如 `openclaw-chat`）。
    -   权限留空（点击 **继续**）。
    -   可访问的主体留空（点击 **完成**）。
3.  创建并下载 **JSON 密钥**：
    -   在服务账号列表中，点击您刚刚创建的账号。
    -   转到 **密钥** 标签页。
    -   点击 **添加密钥** > **创建新密钥**。
    -   选择 **JSON** 并点击 **创建**。
4.  将下载的 JSON 文件存储在您的网关主机上（例如 `~/.openclaw/googlechat-service-account.json`）。
5.  在 [Google Cloud Console Chat 配置](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) 中创建一个 Google Chat 应用：
    -   填写 **应用信息**：
        -   **应用名称**：（例如 `OpenClaw`）
        -   **头像 URL**：（例如 `https://openclaw.ai/logo.png`）
        -   **描述**：（例如 `个人 AI 助手`）
    -   启用 **交互功能**。
    -   在 **功能** 下，勾选 **加入空间和群组对话**。
    -   在 **连接设置** 下，选择 **HTTP 端点 URL**。
    -   在 **触发器** 下，选择 **对所有触发器使用通用 HTTP 端点 URL**，并将其设置为您的网关公共 URL 后接 `/googlechat`。
        -   *提示：运行 `openclaw status` 以查找您的网关公共 URL。*
    -   在 **可见性** 下，勾选 **使此 Chat 应用对 `您的域名` 中的特定人员和群组可用**。
    -   在文本框中输入您的电子邮件地址（例如 `user@example.com`）。
    -   点击底部的 **保存**。
6.  **启用应用状态**：
    -   保存后，**刷新页面**。
    -   查找 **应用状态** 部分（通常在保存后靠近顶部或底部）。
    -   将状态更改为 **上线 - 对用户可用**。
    -   再次点击 **保存**。
7.  使用服务账号路径 + webhook 受众配置 OpenClaw：
    -   环境变量：`GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
    -   或配置文件：`channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8.  设置 webhook 受众类型 + 值（与您的 Chat 应用配置匹配）。
9.  启动网关。Google Chat 将向您的 webhook 路径发送 POST 请求。

## 添加到 Google Chat

一旦网关运行且您的电子邮件已添加到可见性列表：

1.  前往 [Google Chat](https://chat.google.com/)。
2.  点击 **私信** 旁边的 **+**（加号）图标。
3.  在搜索栏（通常用于添加人员的位置）中，输入您在 Google Cloud Console 中配置的 **应用名称**。
    -   **注意**：该机器人*不会*出现在“市场”浏览列表中，因为它是私有应用。您必须通过名称搜索它。
4.  从结果中选择您的机器人。
5.  点击 **添加** 或 **聊天** 以开始 1:1 对话。
6.  发送“Hello”来触发助手！

## 公共 URL（仅 Webhook）

Google Chat webhook 需要一个公共 HTTPS 端点。出于安全考虑，**仅将 `/googlechat` 路径暴露到互联网**。将 OpenClaw 仪表板和其他敏感端点保留在您的私有网络中。

### 选项 A：Tailscale Funnel（推荐）

使用 Tailscale Serve 处理私有仪表板，使用 Funnel 处理公共 webhook 路径。这使 `/` 保持私有，同时仅暴露 `/googlechat`。

1.  **检查您的网关绑定到哪个地址：**
    
    复制
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    记下 IP 地址（例如 `127.0.0.1`、`0.0.0.0` 或您的 Tailscale IP，如 `100.x.x.x`）。
2.  **仅将仪表板暴露给 tailnet（端口 8443）：**
    
    复制
    
    ```bash
    # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0):
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # 如果仅绑定到 Tailscale IP (例如 100.106.161.80):
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **仅公开暴露 webhook 路径：**
    
    复制
    
    ```bash
    # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0):
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # 如果仅绑定到 Tailscale IP (例如 100.106.161.80):
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **授权节点访问 Funnel：** 如果提示，请访问输出中显示的授权 URL，以在您的 tailnet 策略中为此节点启用 Funnel。
5.  **验证配置：**
    
    复制
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

您的公共 webhook URL 将是：`https://<节点名称>..ts.net/googlechat` 您的私有仪表板保持仅限 tailnet 访问：`https://<节点名称>..ts.net:8443/` 在 Google Chat 应用配置中使用公共 URL（不带 `:8443`）。

> 注意：此配置在重启后仍然有效。如需稍后移除，请运行 `tailscale funnel reset` 和 `tailscale serve reset`。

### 选项 B：反向代理（Caddy）

如果您使用像 Caddy 这样的反向代理，仅代理特定路径：

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

使用此配置，任何对 `your-domain.com/` 的请求将被忽略或返回 404，而 `your-domain.com/googlechat` 则安全地路由到 OpenClaw。

### 选项 C：Cloudflare Tunnel

配置您的隧道入口规则，仅路由 webhook 路径：

-   **路径**：`/googlechat` -> `http://localhost:18789/googlechat`
-   **默认规则**：HTTP 404（未找到）

## 工作原理

1.  Google Chat 向网关发送 webhook POST 请求。每个请求包含一个 `Authorization: Bearer ` 标头。
    -   当标头存在时，OpenClaw 在读取/解析完整的 webhook 主体之前验证承载者身份验证。
    -   支持通过更严格的身份验证前主体预算来处理在主体中携带 `authorizationEventObject.systemIdToken` 的 Google Workspace 附加组件请求。
2.  OpenClaw 根据配置的 `audienceType` + `audience` 验证令牌：
    -   `audienceType: "app-url"` → 受众是您的 HTTPS webhook URL。
    -   `audienceType: "project-number"` → 受众是 Cloud 项目编号。
3.  消息按空间路由：
    -   私聊使用会话键 `agent::googlechat:dm:`。
    -   群组空间使用会话键 `agent::googlechat:group:`。
4.  私聊默认采用配对机制。未知发送者会收到配对码；通过以下命令批准：
    -   `openclaw pairing approve googlechat `
5.  群组空间默认需要 @-提及。如果提及检测需要应用的用户名，请使用 `botUser`。

## 目标标识符

使用以下标识符进行消息投递和允许列表：

-   私信：`users/`（推荐）。
-   原始电子邮件 `name@example.com` 是可变的，仅在 `channels.googlechat.dangerouslyAllowNameMatching: true` 时用于直接允许列表匹配。
-   已弃用：`users/` 被视为用户 ID，而非电子邮件允许列表。
-   群组空间：`spaces/`。

## 配置要点

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // 或 serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选；有助于提及检测
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "仅简短回答。",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

注意事项：

-   服务账号凭据也可以通过 `serviceAccount`（JSON 字符串）内联传递。
-   也支持 `serviceAccountRef`（环境变量/文件 SecretRef），包括 `channels.googlechat.accounts..serviceAccountRef` 下的每个账号引用。
-   如果未设置 `webhookPath`，默认 webhook 路径为 `/googlechat`。
-   `dangerouslyAllowNameMatching` 重新启用可变的电子邮件主体匹配以用于允许列表（应急兼容模式）。
-   当 `actions.reactions` 启用时，可以通过 `reactions` 工具和 `channels action` 使用反应功能。
-   `typingIndicator` 支持 `none`、`message`（默认）和 `reaction`（反应需要用户 OAuth）。
-   附件通过 Chat API 下载并存储在媒体管道中（大小受 `mediaMaxMb` 限制）。

密钥管理参考详情：[密钥管理](../gateway/secrets.md)。

## 故障排除

### 405 方法不允许

如果 Google Cloud Logs Explorer 显示如下错误：

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

这意味着 webhook 处理器未注册。常见原因：

1.  **通道未配置**：您的配置中缺少 `channels.googlechat` 部分。通过以下命令验证：
    
    复制
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    如果返回“Config path not found”，请添加配置（参见[配置要点](#config-highlights)）。
2.  **插件未启用**：检查插件状态：
    
    复制
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    如果显示“disabled”，请在您的配置中添加 `plugins.entries.googlechat.enabled: true`。
3.  **网关未重启**：添加配置后，重启网关：
    
    复制
    
    ```bash
    openclaw gateway restart
    ```
    

验证通道是否正在运行：

```bash
openclaw channels status
# 应显示：Google Chat default: enabled, configured, ...
```

### 其他问题

-   使用 `openclaw channels status --probe` 检查身份验证错误或缺少受众配置。
-   如果没有消息到达，请确认 Chat 应用的 webhook URL + 事件订阅。
-   如果提及门控阻止回复，请将 `botUser` 设置为应用的用户资源名称并验证 `requireMention`。
-   在发送测试消息时使用 `openclaw logs --follow` 查看请求是否到达网关。

相关文档：

-   [网关配置](../gateway/configuration.md)
-   [安全性](../gateway/security.md)
-   [反应](../tools/reactions.md)

[飞书](./feishu.md)[iMessage](./imessage.md)