

  CLI 命令

  
# system

网关的系统级辅助工具：排队系统事件、控制心跳以及查看状态。

## 常用命令

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## system event

在**主**会话中排队一个系统事件。下一次心跳会将其作为 `System:` 行注入到提示中。使用 `--mode now` 立即触发心跳；`next-heartbeat` 则等待下一次计划的心跳。标志：

-   `--text <文本>`: 必需的系统事件文本。
-   `--mode <模式>`: `now` 或 `next-heartbeat`（默认）。
-   `--json`: 机器可读的输出。

## system heartbeat last|enable|disable

心跳控制：

-   `last`: 显示最后一次心跳事件。
-   `enable`: 重新开启心跳（如果心跳被禁用，请使用此命令）。
-   `disable`: 暂停心跳。

标志：

-   `--json`: 机器可读的输出。

## system presence

列出网关当前已知的系统状态条目（节点、实例及类似的状态行）。标志：

-   `--json`: 机器可读的输出。

## 注意事项

-   需要一个可通过当前配置（本地或远程）访问的正在运行的网关。
-   系统事件是临时的，不会在重启后持久化。

[status](./status.md)[tui](./tui.md)