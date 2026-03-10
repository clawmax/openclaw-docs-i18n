

  CLI 命令

  
# devices

当你需要处理设备接入请求、查看已配对设备或管理设备专属令牌时，这个命令组就是你的工具箱。无论是审批新设备的配对请求，还是轮换、撤销已有设备的令牌，都可以在这里完成。

## 命令

### openclaw devices list

查看当前有哪些设备正在等待配对，以及已经配对成功的设备列表。

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

移除单个已配对的设备。一旦移除，该设备将无法再访问网关。

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes [--pending]

批量清除设备记录。默认清除已配对设备，加上 `--pending` 则清除待处理的配对请求。由于这是批量操作，必须显式传入 `--yes` 确认执行。

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve [requestId] [--latest]

批准一个待处理的设备配对请求。如果不指定 `requestId`，OpenClaw 会自动批准最新的那个请求。

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

拒绝一个待处理的设备配对请求。

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; [--scope &lt;scope...&gt;]

为指定设备的某个角色轮换令牌。你也可以顺便更新该令牌的作用域。

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

撤销指定设备的某个角色令牌。

```bash
openclaw devices revoke --device <deviceId> --role node
```

## 通用选项

- `--url `：网关 WebSocket 地址，默认使用配置中的 `gateway.remote.url`
- `--token `：网关访问令牌（如需要）
- `--password `：网关密码（用于密码认证）
- `--timeout `：RPC 调用的超时时间
- `--json`：以 JSON 格式输出，适合脚本解析

注意：当你指定了 `--url`，CLI 就不会再从配置文件或环境变量中读取凭据了。这种情况下必须显式传入 `--token` 或 `--password`，否则会报错。

## 注意事项

- 令牌轮换会返回新的令牌，这是敏感信息，请妥善保管
- 这些命令需要 `operator.pairing` 或更高权限的 `operator.admin` 作用域
- `devices clear` 是批量删除操作，所以特意加了 `--yes` 的确认门槛
- 如果本地回环地址上没有配对作用域（且未指定 `--url`），`list` 和 `approve` 命令会自动使用本地配对回退机制

[仪表盘](./dashboard.md)[目录](./directory.md)