

  CLI 命令

  
# daemon

用于网关服务管理的传统别名。`openclaw daemon ...` 映射到与 `openclaw gateway ...` 服务命令相同的服务控制界面。

## 用法

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## 子命令

-   `status`: 显示服务安装状态并探测网关健康状态
-   `install`: 安装服务 (`launchd`/`systemd`/`schtasks`)
-   `uninstall`: 移除服务
-   `start`: 启动服务
-   `stop`: 停止服务
-   `restart`: 重启服务

## 常用选项

-   `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   生命周期命令 (`uninstall|start|stop|restart`): `--json`

注意：

-   `status` 在可能的情况下会解析配置的身份验证 SecretRefs 以用于探测身份验证。
-   当令牌身份验证需要令牌且 `gateway.auth.token` 由 SecretRef 管理时，`install` 会验证该 SecretRef 是否可解析，但不会将解析出的令牌持久化到服务环境元数据中。
-   如果令牌身份验证需要令牌且配置的令牌 SecretRef 未解析，安装将失败（保守处理）。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，安装将被阻止，直到显式设置模式。

## 推荐使用

请使用 [`openclaw gateway`](./gateway.md) 查看当前文档和示例。

[cron](./cron.md)[dashboard](./dashboard.md)

---