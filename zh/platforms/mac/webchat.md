

  macOS 配套应用

  
# WebChat

macOS 菜单栏应用将 WebChat UI 作为原生 SwiftUI 视图嵌入。它连接到网关，并默认使用所选代理的**主会话**（并提供会话切换器以访问其他会话）。

-   **本地模式**：直接连接到本地网关 WebSocket。
-   **远程模式**：通过 SSH 转发网关控制端口，并将该隧道用作数据平面。

## 启动与调试

-   手动：Lobster 菜单 → “打开聊天”。
-   测试时自动打开：
    
    复制
    
    ```bash
    dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
    ```
    
-   日志：`./scripts/clawlog.sh`（子系统 `ai.openclaw`，类别 `WebChatSwiftUI`）。

## 工作原理

-   数据平面：网关 WS 方法 `chat.history`, `chat.send`, `chat.abort`, `chat.inject` 以及事件 `chat`, `agent`, `presence`, `tick`, `health`。
-   会话：默认为主会话（`main`，或作用域为全局时的 `global`）。UI 可以在不同会话间切换。
-   新手指南使用专用会话，以将首次运行设置分开。

## 安全层面

-   远程模式仅通过 SSH 转发网关 WebSocket 控制端口。

## 已知限制

-   UI 针对聊天会话进行了优化（并非完整的浏览器沙盒）。

[语音叠加层](./voice-overlay.md)[画布](./canvas.md)

---