

  平台概览

  
# Android 应用

## 支持概览

-   角色：伴侣节点应用（Android 不托管网关）。
-   需要网关：是（在 macOS、Linux 或通过 WSL2 的 Windows 上运行）。
-   安装：[入门指南](../start/getting-started.md) + [配对](../channels/pairing.md)。
-   网关：[运行手册](../gateway.md) + [配置](../gateway/configuration.md)。
    -   协议：[网关协议](../gateway/protocol.md)（节点 + 控制平面）。

## 系统控制

系统控制（launchd/systemd）位于网关主机上。参见[网关](../gateway.md)。

## 连接运行手册

Android 节点应用 ⇄ (mDNS/NSD + WebSocket) ⇄ **网关** Android 直接连接到网关 WebSocket（默认 `ws://:18789`）并使用设备配对（`role: node`）。

### 先决条件

-   您可以在“主”机器上运行网关。
-   Android 设备/模拟器可以访问网关 WebSocket：
    -   同一局域网内使用 mDNS/NSD，**或**
    -   同一 Tailscale tailnet 内使用广域 Bonjour / 单播 DNS-SD（见下文），**或**
    -   手动指定网关主机/端口（备用方案）
-   您可以在网关机器上（或通过 SSH）运行 CLI (`openclaw`)。

### 1) 启动网关

```bash
openclaw gateway --port 18789 --verbose
```

在日志中确认看到类似信息：

-   `listening on ws://0.0.0.0:18789`

对于仅限 tailnet 的设置（推荐用于维也纳 ⇄ 伦敦），将网关绑定到 tailnet IP：

-   在网关主机的 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
-   重启网关 / macOS 菜单栏应用。

### 2) 验证发现（可选）

从网关机器上：

```bash
dns-sd -B _openclaw-gw._tcp local.
```

更多调试说明：[Bonjour](../gateway/bonjour.md)。

#### 通过单播 DNS-SD 进行 Tailnet（维也纳 ⇄ 伦敦）发现

Android NSD/mDNS 发现无法跨网络工作。如果您的 Android 节点和网关位于不同网络但通过 Tailscale 连接，请改用广域 Bonjour / 单播 DNS-SD：

1.  在网关主机上设置一个 DNS-SD 区域（例如 `openclaw.internal.`）并发布 `_openclaw-gw._tcp` 记录。
2.  为您的选定域名配置 Tailscale 分割 DNS，指向该 DNS 服务器。

详细信息和示例 CoreDNS 配置：[Bonjour](../gateway/bonjour.md)。

### 3) 从 Android 连接

在 Android 应用中：

-   应用通过**前台服务**（持久通知）保持其网关连接活跃。
-   打开**连接**标签页。
-   使用**设置代码**或**手动**模式。
-   如果发现被阻止，请在**高级控制**中使用手动主机/端口（以及需要时的 TLS/令牌/密码）。

首次成功配对后，Android 在启动时会自动重连：

-   手动端点（如果启用），否则
-   最后发现的网关（尽力而为）。

### 4) 批准配对（CLI）

在网关机器上：

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

配对详情：[配对](../channels/pairing.md)。

### 5) 验证节点已连接

-   通过节点状态：
    
    复制
    
    ```bash
    openclaw nodes status
    ```
    
-   通过网关：
    
    复制
    
    ```bash
    openclaw gateway call node.list --params "{}"
    ```
    

### 6) 聊天 + 历史记录

Android 聊天标签页支持会话选择（默认 `main`，以及其他现有会话）：

-   历史记录：`chat.history`
-   发送：`chat.send`
-   推送更新（尽力而为）：`chat.subscribe` → `event:"chat"`

### 7) 画布 + 屏幕 + 相机

#### 网关画布主机（推荐用于网页内容）

如果您希望节点显示代理可以在磁盘上编辑的真实 HTML/CSS/JS，请将节点指向网关画布主机。注意：节点从网关 HTTP 服务器加载画布（与 `gateway.port` 相同端口，默认 `18789`）。

1.  在网关主机上创建 `~/.openclaw/workspace/canvas/index.html`。
2.  将节点导航到该地址（局域网）：

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet（可选）：如果两台设备都在 Tailscale 上，请使用 MagicDNS 名称或 tailnet IP 代替 `.local`，例如 `http://<gateway-magicdns>:18789/__openclaw__/canvas/`。此服务器向 HTML 注入实时重载客户端，并在文件更改时重新加载。A2UI 主机位于 `http://<gateway-host>:18789/__openclaw__/a2ui/`。画布命令（仅限前台）：

-   `canvas.eval`, `canvas.snapshot`, `canvas.navigate`（使用 `{"url":""}` 或 `{"url":"/"}` 返回默认脚手架）。`canvas.snapshot` 返回 `{ format, base64 }`（默认 `format="jpeg"`）。
-   A2UI：`canvas.a2ui.push`, `canvas.a2ui.reset`（`canvas.a2ui.pushJSONL` 旧别名）

相机命令（仅限前台；权限控制）：

-   `camera.snap` (jpg)
-   `camera.clip` (mp4)

有关参数和 CLI 助手，请参阅[相机节点](../nodes/camera.md)。屏幕命令：

-   `screen.record` (mp4；仅限前台)

### 8) 语音 + 扩展的 Android 命令界面

-   语音：Android 在语音标签页中使用单麦克风开关流程，进行转录捕获和 TTS 播放（配置 ElevenLabs 时使用，系统 TTS 作为后备）。
-   语音唤醒/通话模式切换目前已从 Android UX/运行时中移除。
-   其他 Android 命令族（可用性取决于设备 + 权限）：
    -   `device.status`, `device.info`, `device.permissions`, `device.health`
    -   `notifications.list`, `notifications.actions`
    -   `photos.latest`
    -   `contacts.search`, `contacts.add`
    -   `calendar.events`, `calendar.add`
    -   `motion.activity`, `motion.pedometer`
    -   `app.update`

[Windows (WSL2)](./windows.md)[iOS 应用](./ios.md)