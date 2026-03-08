

  平台概览

  
# iOS 应用

可用性：内部预览版。iOS 应用尚未公开发布。

## 功能概述

-   通过 WebSocket（局域网或 Tailnet）连接到网关。
-   暴露节点能力：画布、屏幕截图、摄像头捕获、位置、通话模式、语音唤醒。
-   接收 `node.invoke` 命令并报告节点状态事件。

## 系统要求

-   在另一台设备（macOS、Linux 或通过 WSL2 的 Windows）上运行的网关。
-   网络路径：
    -   通过 Bonjour 在同一局域网内，**或**
    -   通过单播 DNS-SD 的 Tailnet（示例域名：`openclaw.internal.`），**或**
    -   手动指定主机/端口（备用方案）。

## 快速开始（配对 + 连接）

1.  启动网关：

```bash
openclaw gateway --port 18789
```

2.  在 iOS 应用中，打开设置并选择一个已发现的网关（或启用“手动主机”并输入主机/端口）。
3.  在网关主机上批准配对请求：

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  验证连接：

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## 发现路径

### Bonjour（局域网）

网关在 `local.` 域上广播 `_openclaw-gw._tcp`。iOS 应用会自动列出这些服务。

### Tailnet（跨网络）

如果 mDNS 被阻止，请使用单播 DNS-SD 区域（选择一个域名；例如：`openclaw.internal.`）和 Tailscale 分割 DNS。请参阅 [Bonjour](../gateway/bonjour.md) 了解 CoreDNS 示例。

### 手动主机/端口

在设置中，启用**手动主机**并输入网关主机 + 端口（默认 `18789`）。

## 画布 + A2UI

iOS 节点渲染一个 WKWebView 画布。使用 `node.invoke` 来驱动它：

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

注意：

-   网关画布主机提供 `/__openclaw__/canvas/` 和 `/__openclaw__/a2ui/` 服务。
-   它由网关 HTTP 服务器提供（与 `gateway.port` 相同，默认 `18789`）。
-   当广告了画布主机 URL 时，iOS 节点在连接时会自动导航到 A2UI。
-   使用 `canvas.navigate` 和 `{"url":""}` 返回到内置脚手架。

### 画布执行脚本 / 截图

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## 语音唤醒 + 通话模式

-   语音唤醒和通话模式在设置中可用。
-   iOS 可能会暂停后台音频；当应用未处于活动状态时，请将语音功能视为尽力而为的服务。

## 常见错误

-   `NODE_BACKGROUND_UNAVAILABLE`：将 iOS 应用切换到前台（画布/摄像头/屏幕命令需要此操作）。
-   `A2UI_HOST_NOT_CONFIGURED`：网关未广告画布主机 URL；请检查[网关配置](../gateway/configuration.md)中的 `canvasHost`。
-   配对提示从未出现：运行 `openclaw devices list` 并手动批准。
-   重新安装后连接失败：钥匙串中的配对令牌已被清除；请重新配对节点。

## 相关文档

-   [配对](../channels/pairing.md)
-   [发现](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[Android 应用](./android.md)[DigitalOcean](./digitalocean.md)