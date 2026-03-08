

  配置与操作

  
# 网关锁

最后更新: 2025-12-11

## 目的

-   确保在同一主机上，每个基础端口仅运行一个网关实例；额外的网关必须使用隔离的配置文件和唯一的端口。
-   在崩溃或收到 SIGKILL 信号后能够存活，且不会留下陈旧的锁文件。
-   当控制端口已被占用时，快速失败并给出清晰的错误信息。

## 机制

-   网关在启动时立即使用独占的 TCP 监听器绑定 WebSocket 监听器（默认为 `ws://127.0.0.1:18789`）。
-   如果绑定因 `EADDRINUSE` 失败，启动将抛出 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`。
-   操作系统会在任何进程退出（包括崩溃和 SIGKILL）时自动释放监听器——无需单独的锁文件或清理步骤。
-   在关闭时，网关会关闭 WebSocket 服务器和底层的 HTTP 服务器，以迅速释放端口。

## 错误表现

-   如果另一个进程占用了端口，启动将抛出 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`。
-   其他绑定失败会表现为 `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:: …")`。

## 操作说明

-   如果端口被*另一个*进程占用，错误信息相同；请释放该端口或使用 `openclaw gateway --port ` 选择另一个端口。
-   macOS 应用程序在生成网关前仍会维护其自身的轻量级 PID 防护；运行时锁由 WebSocket 绑定强制执行。

[日志记录](./logging.md)[后台执行与进程工具](./background-process.md)