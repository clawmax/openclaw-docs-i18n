

  Web 界面

  
# WebChat

状态：macOS/iOS SwiftUI 聊天界面直接与网关 WebSocket 通信。

## 功能概述

-   网关的原生聊天界面（无需嵌入式浏览器或本地静态服务器）。
-   使用与其他通道相同的会话和路由规则。
-   确定性路由：回复始终返回至 WebChat。

## 快速开始

1.  启动网关。
2.  打开 WebChat UI（macOS/iOS 应用）或控制界面的聊天标签页。
3.  确保网关认证已配置（默认必需，即使在环回地址上）。

## 工作原理（行为）

-   该界面连接到网关 WebSocket 并使用 `chat.history`、`chat.send` 和 `chat.inject`。
-   `chat.history` 为保持稳定性而设限：网关可能会截断长文本字段、省略繁重的元数据，并用 `[chat.history omitted: message too large]` 替换过大的条目。
-   `chat.inject` 直接将助手备注附加到对话记录中并广播到界面（不运行代理）。
-   已中止的运行可能会在界面中保留部分助手输出可见。
-   当存在缓冲输出时，网关会将已中止的部分助手文本持久化到对话记录历史中，并用中止元数据标记这些条目。
-   历史记录始终从网关获取（不监视本地文件）。
-   如果网关无法访问，WebChat 将处于只读模式。

## 控制界面代理工具面板

-   控制界面的 `/agents` 工具面板通过 `tools.catalog` 获取运行时目录，并将每个工具标记为 `core` 或 `plugin:`（对于可选插件工具还会加上 `optional`）。
-   如果 `tools.catalog` 不可用，该面板将回退到内置的静态列表。
-   该面板编辑配置文件和覆盖配置，但有效的运行时访问仍遵循策略优先级（`allow`/`deny`、每个代理以及提供商/通道的覆盖）。

## 远程使用

-   远程模式通过 SSH/Tailscale 隧道传输网关 WebSocket。
-   您无需运行单独的 WebChat 服务器。

## 配置参考（WebChat）

完整配置：[配置](../gateway/configuration.md) 通道选项：

-   没有专用的 `webchat.*` 块。WebChat 使用以下网关端点 + 认证设置。

相关的全局选项：

-   `gateway.port`, `gateway.bind`: WebSocket 主机/端口。
-   `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 认证（令牌/密码）。
-   `gateway.auth.mode: "trusted-proxy"`: 用于浏览器客户端的反向代理认证（参见[可信代理认证](../gateway/trusted-proxy-auth.md)）。
-   `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: 远程网关目标。
-   `session.*`: 会话存储和主密钥默认值。

[仪表板](./dashboard.md)[TUI](./tui.md)

---