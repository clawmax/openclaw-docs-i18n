

  RPC 与 API

  
# RPC 适配器

OpenClaw 通过 JSON-RPC 集成外部 CLI。目前使用两种模式。

## 模式 A: HTTP 守护进程 (signal-cli)

-   `signal-cli` 作为守护进程运行，通过 HTTP 提供 JSON-RPC。
-   事件流采用 SSE (`/api/v1/events`)。
-   健康检查：`/api/v1/check`。
-   当 `channels.signal.autoStart=true` 时，OpenClaw 管理其生命周期。

有关设置和端点，请参阅 [Signal](../channels/signal.md)。

## 模式 B: stdio 子进程 (旧版: imsg)

> **注意：** 对于新的 iMessage 设置，请改用 [BlueBubbles](../channels/bluebubbles.md)。

-   OpenClaw 将 `imsg rpc` 作为子进程启动（旧版 iMessage 集成）。
-   JSON-RPC 通过 stdin/stdout 以行分隔方式传输（每行一个 JSON 对象）。
-   无需 TCP 端口，也无需守护进程。

使用的核心方法：

-   `watch.subscribe` → 通知 (`method: "message"`)
-   `watch.unsubscribe`
-   `send`
-   `chats.list` (探测/诊断)

有关旧版设置和寻址方式（推荐使用 `chat_id`），请参阅 [iMessage](../channels/imessage.md)。

## 适配器指南

-   网关拥有进程控制权（启动/停止与提供者生命周期绑定）。
-   保持 RPC 客户端的健壮性：设置超时，在退出时重启。
-   优先使用稳定 ID（例如 `chat_id`）而非显示字符串。

[webhooks](../cli/webhooks.md)[设备模型数据库](./device-models.md)

---