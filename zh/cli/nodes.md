

  CLI 命令

  
# nodes

管理配对的节点（设备）并调用节点能力。相关：

-   节点概述：[节点](../nodes.md)
-   相机：[相机节点](../nodes/camera.md)
-   图像：[图像节点](../nodes/images.md)

常用选项：

-   `--url`, `--token`, `--timeout`, `--json`

## 常用命令

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` 打印待处理/已配对表格。已配对行包含最近连接时间（Last Connect）。使用 `--connected` 仅显示当前连接的节点。使用 `--last-connected ` 过滤出在指定时长内连接过的节点（例如 `24h`, `7d`）。

## 调用 / 运行

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

调用标志：

-   `--params `: JSON 对象字符串（默认 `{}`）。
-   `--invoke-timeout `: 节点调用超时时间（默认 `15000`）。
-   `--idempotency-key `: 可选幂等键。

### 执行风格默认值

`nodes run` 镜像模型的执行行为（默认值 + 批准）：

-   读取 `tools.exec.*`（以及 `agents.list[].tools.exec.*` 覆盖）。
-   在调用 `system.run` 之前使用执行批准（`exec.approval.request`）。
-   当 `tools.exec.node` 已设置时，可以省略 `--node`。
-   需要一个声明支持 `system.run` 的节点（macOS 伴侣应用或无头节点主机）。

标志：

-   `--cwd `: 工作目录。
-   `--env <key=val>`: 环境变量覆盖（可重复）。注意：节点主机会忽略 `PATH` 覆盖（并且 `tools.exec.pathPrepend` 不应用于节点主机）。
-   `--command-timeout `: 命令超时时间。
-   `--invoke-timeout `: 节点调用超时时间（默认 `30000`）。
-   `--needs-screen-recording`: 需要屏幕录制权限。
-   `--raw `: 运行一个 Shell 字符串（`/bin/sh -lc` 或 `cmd.exe /c`）。在 Windows 节点主机的允许列表模式下，`cmd.exe /c` shell 包装器运行需要批准（仅允许列表条目不会自动允许包装器形式）。
-   `--agent `: 代理范围的批准/允许列表（默认为配置的代理）。
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: 覆盖。

[node](./node.md)[onboard](./onboard.md)

---