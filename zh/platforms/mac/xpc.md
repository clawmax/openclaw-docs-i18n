

  macOS 伴侣应用

  
# macOS IPC

**当前模型：** 一个本地 Unix 套接字将**节点主机服务**连接到 **macOS 应用**，用于执行批准和 `system.run`。存在一个 `openclaw-mac` 调试 CLI 用于发现/连接检查；代理操作仍通过网关 WebSocket 和 `node.invoke` 进行。UI 自动化使用 PeekabooBridge。

## 目标

-   单一的 GUI 应用实例，负责所有面向 TCC 的工作（通知、屏幕录制、麦克风、语音、AppleScript）。
-   一个小的自动化接口：网关 + 节点命令，外加 PeekabooBridge 用于 UI 自动化。
-   可预测的权限：始终使用相同的签名包标识符，由 launchd 启动，因此 TCC 授权得以保留。

## 工作原理

### 网关 + 节点传输

-   应用运行网关（本地模式）并作为节点连接到它。
-   代理操作通过 `node.invoke` 执行（例如 `system.run`、`system.notify`、`canvas.*`）。

### 节点服务 + 应用 IPC

-   一个无头节点主机服务连接到网关 WebSocket。
-   `system.run` 请求通过本地 Unix 套接字转发到 macOS 应用。
-   应用在 UI 上下文中执行命令，如果需要则提示用户，并返回输出。

示意图 (SCI)：

```
代理 -> 网关 -> 节点服务 (WS)
                      |  IPC (UDS + 令牌 + HMAC + TTL)
                      v
                  Mac 应用 (UI + TCC + system.run)
```

### PeekabooBridge (UI 自动化)

-   UI 自动化使用一个单独的名为 `bridge.sock` 的 UNIX 套接字和 PeekabooBridge JSON 协议。
-   主机偏好顺序（客户端）：Peekaboo.app → Claude.app → OpenClaw.app → 本地执行。
-   安全性：桥接主机需要匹配允许的 TeamID；仅限 DEBUG 模式的同 UID 逃生舱由 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` 保护（Peekaboo 惯例）。
-   详见：[PeekabooBridge 使用指南](./peekaboo.md)。

## 操作流程

-   重启/重建：`SIGN_IDENTITY="Apple Development: <开发者姓名> ()" scripts/restart-mac.sh`
    -   终止现有实例
    -   Swift 构建 + 打包
    -   写入/引导/启动 LaunchAgent
-   单实例：如果另一个具有相同包标识符的实例正在运行，则应用提前退出。

## 加固说明

-   优先要求所有特权接口匹配 TeamID。
-   PeekabooBridge：`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（仅限 DEBUG）可能允许同 UID 调用者用于本地开发。
-   所有通信保持仅在本地；不暴露网络套接字。
-   TCC 提示仅源自 GUI 应用包；在重建过程中保持签名的包标识符稳定。
-   IPC 加固：套接字模式 `0600`、令牌、对等 UID 检查、HMAC 质询/响应、短 TTL。

[macOS 上的网关](./bundled-gateway.md)[技能](./skills.md)