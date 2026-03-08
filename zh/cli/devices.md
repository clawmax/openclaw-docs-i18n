

  CLI 命令

  
# devices

管理设备配对请求和设备作用域令牌。

## 命令

### openclaw devices list

列出待处理的配对请求和已配对的设备。

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

移除一个已配对的设备条目。

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

批量清除已配对的设备。

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

批准一个待处理的设备配对请求。如果省略 `requestId`，OpenClaw 会自动批准最近的一个待处理请求。

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

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

为特定角色轮换设备令牌（可选更新作用域）。

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

撤销特定角色的设备令牌。

```bash
openclaw devices revoke --device <deviceId> --role node
```

## 通用选项

-   `--url `: 网关 WebSocket URL（默认使用配置中的 `gateway.remote.url`）。
-   `--token `: 网关令牌（如果需要）。
-   `--password `: 网关密码（密码认证）。
-   `--timeout `: RPC 超时时间。
-   `--json`: JSON 输出（推荐用于脚本）。

注意：当你设置 `--url` 时，CLI 不会回退到配置文件或环境变量中的凭据。请显式传递 `--token` 或 `--password`。缺少显式凭据会导致错误。

## 说明

-   令牌轮换会返回一个新令牌（敏感信息）。请将其视为机密。
-   这些命令需要 `operator.pairing`（或 `operator.admin`）作用域。
-   `devices clear` 命令特意通过 `--yes` 进行保护。
-   如果在本地回环地址上无法使用配对作用域（且未显式传递 `--url`），list/approve 命令可以使用本地配对回退机制。

[仪表盘](./dashboard.md)[目录](./directory.md)

---