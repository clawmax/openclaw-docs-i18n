

  配置与操作

  
# 后台执行与进程工具

OpenClaw 通过 `exec` 工具运行 shell 命令，并将长时间运行的任务保存在内存中。`process` 工具管理这些后台会话。

## exec 工具

关键参数：

-   `command` (必需)
-   `yieldMs` (默认 10000)：在此延迟后自动转为后台运行
-   `background` (布尔值)：立即在后台运行
-   `timeout` (秒，默认 1800)：在此超时后终止进程
-   `elevated` (布尔值)：如果启用了提升模式/允许，则在主机上运行
-   需要真实的 TTY？设置 `pty: true`。
-   `workdir`, `env`

行为：

-   前台运行直接返回输出。
-   当转为后台运行时（显式设置或超时），工具返回 `status: "running"` + `sessionId` 和一小段尾部输出。
-   输出保存在内存中，直到会话被轮询或清除。
-   如果 `process` 工具被禁用，`exec` 将同步运行并忽略 `yieldMs`/`background`。
-   派生的 exec 命令会收到 `OPENCLAW_SHELL=exec` 环境变量，用于上下文感知的 shell/配置文件规则。

## 子进程桥接

当在 exec/process 工具之外派生长时间运行的子进程时（例如，CLI 重启或网关助手），请附加子进程桥接助手，以便转发终止信号并在退出/错误时分离监听器。这可以避免在 systemd 上产生孤儿进程，并确保跨平台的关闭行为一致。环境变量覆盖：

-   `PI_BASH_YIELD_MS`：默认 yield (毫秒)
-   `PI_BASH_MAX_OUTPUT_CHARS`：内存中输出上限 (字符数)
-   `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`：每个流的待处理 stdout/stderr 上限 (字符数)
-   `PI_BASH_JOB_TTL_MS`：已完成会话的 TTL (毫秒，限制在 1分钟–3小时)

配置（推荐）：

-   `tools.exec.backgroundMs` (默认 10000)
-   `tools.exec.timeoutSec` (默认 1800)
-   `tools.exec.cleanupMs` (默认 1800000)
-   `tools.exec.notifyOnExit` (默认 true)：当后台 exec 退出时，将系统事件加入队列并请求心跳。
-   `tools.exec.notifyOnExitEmptySuccess` (默认 false)：如果为 true，对于成功完成但未产生任何输出的后台运行，也会将完成事件加入队列。

## process 工具

操作：

-   `list`：正在运行 + 已完成的会话
-   `poll`：获取会话的新输出（同时报告退出状态）
-   `log`：读取聚合输出（支持 `offset` + `limit`）
-   `write`：发送 stdin (`data`, 可选的 `eof`)
-   `kill`：终止后台会话
-   `clear`：从内存中移除已完成的会话
-   `remove`：如果正在运行则终止，否则如果已完成则清除

注意：

-   只有转为后台运行的会话才会被列出/持久化在内存中。
-   进程重启时会话将丢失（无磁盘持久化）。
-   会话日志只有在您运行 `process poll/log` 并且工具结果被记录时，才会保存到聊天历史中。
-   `process` 的作用域限定在每个代理；它只能看到由该代理启动的会话。
-   `process list` 包含一个派生的 `name`（命令动词 + 目标），用于快速扫描。
-   `process log` 使用基于行的 `offset`/`limit`。
-   当 `offset` 和 `limit` 都被省略时，它返回最后 200 行并包含分页提示。
-   当提供了 `offset` 而 `limit` 被省略时，它返回从 `offset` 到末尾的内容（不限制为 200 行）。

## 示例

运行一个长任务并稍后轮询：

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

立即在后台启动：

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

发送 stdin：

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```

[网关锁](./gateway-lock.md)[多网关](./multiple-gateways.md)