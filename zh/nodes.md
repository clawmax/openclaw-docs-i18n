

  媒体与设备

  
# 节点

**节点** 是一个连接到网关 **WebSocket**（与操作员使用相同端口）的配套设备（macOS/iOS/Android/无头设备），其连接时指定 `role: "node"`，并通过 `node.invoke` 暴露一个命令接口（例如 `canvas.*`、`camera.*`、`device.*`、`notifications.*`、`system.*`）。协议详情：[网关协议](./gateway/protocol.md)。旧版传输方式：[桥接协议](./gateway/bridge-protocol.md)（TCP JSONL；当前节点已弃用/移除）。macOS 也可以运行在 **节点模式**：菜单栏应用连接到网关的 WS 服务器，并将其本地画布/摄像头命令作为节点暴露（因此 `openclaw nodes …` 可对此 Mac 生效）。注意：

-   节点是**外围设备**，而非网关。它们不运行网关服务。
-   Telegram/WhatsApp 等消息到达**网关**，而非节点。
-   故障排除手册：[/nodes/troubleshooting](./nodes/troubleshooting.md)

## 配对 + 状态

**WS 节点使用设备配对。** 节点在 `connect` 过程中呈现一个设备身份；网关会为 `role: node` 创建一个设备配对请求。通过设备 CLI（或 UI）批准。快速 CLI：

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

注意：

-   当节点的设备配对角色包含 `node` 时，`nodes status` 会将其标记为**已配对**。
-   `node.pair.*`（CLI：`openclaw nodes pending/approve/reject`）是一个独立的、由网关管理的节点配对存储；它**不**控制 WS `connect` 握手过程。

## 远程节点主机 (system.run)

当你的网关运行在一台机器上，而希望命令在另一台机器上执行时，使用**节点主机**。模型仍然与**网关**通信；当选择 `host=node` 时，网关将 `exec` 调用转发给**节点主机**。

### 什么运行在哪里

-   **网关主机**：接收消息，运行模型，路由工具调用。
-   **节点主机**：在节点机器上执行 `system.run`/`system.which`。
-   **审批**：通过 `~/.openclaw/exec-approvals.json` 在节点主机上强制执行。

### 启动节点主机（前台运行）

在节点机器上：

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "构建节点"
```

### 通过 SSH 隧道连接远程网关（环回绑定）

如果网关绑定到环回地址（`gateway.bind=loopback`，本地模式下的默认设置），远程节点主机无法直接连接。创建一个 SSH 隧道，并将节点主机指向隧道的本地端。示例（节点主机 -> 网关主机）：

```bash
# 终端 A（保持运行）：将本地 18790 端口转发到网关 127.0.0.1:18789 端口
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# 终端 B：导出网关令牌并通过隧道连接
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "构建节点"
```

注意：

-   令牌是网关配置中 `gateway.auth.token` 的值（位于网关主机的 `~/.openclaw/openclaw.json`）。
-   `openclaw node run` 读取 `OPENCLAW_GATEWAY_TOKEN` 进行身份验证。

### 启动节点主机（服务）

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "构建节点"
openclaw node restart
```

### 配对 + 命名

在网关主机上：

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

命名选项：

-   `openclaw node run` / `openclaw node install` 上的 `--display-name`（持久化存储在节点的 `~/.openclaw/node.json` 中）。
-   `openclaw nodes rename --node <id|name|ip> --name "构建节点"`（网关覆盖）。

### 允许命令列表

执行审批是**针对每个节点主机**的。从网关添加允许列表条目：

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

审批条目位于节点主机的 `~/.openclaw/exec-approvals.json`。

### 将 exec 指向节点

配置默认值（网关配置）：

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

或按会话配置：

```bash
/exec host=node security=allowlist node=<id-or-name>
```

设置后，任何带有 `host=node` 的 `exec` 调用都将在节点主机上运行（受节点允许列表/审批约束）。相关：

-   [节点主机 CLI](./cli/node.md)
-   [Exec 工具](./tools/exec.md)
-   [Exec 审批](./tools/exec-approvals.md)

## 调用命令

底层（原始 RPC）：

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

对于常见的“给代理一个 MEDIA 附件”工作流，存在更高级的辅助命令。

## 截图（画布快照）

如果节点正在显示画布（WebView），`canvas.snapshot` 返回 `{ format, base64 }`。CLI 辅助命令（写入临时文件并打印 `MEDIA:`）：

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### 画布控制

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

注意：

-   `canvas present` 接受 URL 或本地文件路径（`--target`），以及可选的 `--x/--y/--width/--height` 用于定位。
-   `canvas eval` 接受内联 JS（`--js`）或位置参数。

### A2UI（画布）

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

注意：

-   仅支持 A2UI v0.8 JSONL（v0.9/createSurface 会被拒绝）。

## 照片 + 视频（节点摄像头）

照片（`jpg`）：

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # 默认：前后摄像头（输出 2 行 MEDIA）
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

视频片段（`mp4`）：

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

注意：

-   对于 `canvas.*` 和 `camera.*`，节点必须处于**前台**（后台调用会返回 `NODE_BACKGROUND_UNAVAILABLE`）。
-   片段时长被限制（目前 `<= 60s`）以避免过大的 base64 负载。
-   Android 会在可能时提示 `CAMERA`/`RECORD_AUDIO` 权限；被拒绝的权限会以 `*_PERMISSION_REQUIRED` 失败。

## 屏幕录制（节点）

节点暴露 `screen.record`（mp4）。示例：

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

注意：

-   `screen.record` 要求节点应用处于前台。
-   Android 会在录制前显示系统屏幕捕获提示。
-   屏幕录制时长限制为 `<= 60s`。
-   `--no-audio` 禁用麦克风捕获（iOS/Android 支持；macOS 使用系统捕获音频）。
-   当有多个屏幕可用时，使用 `--screen ` 选择显示器。

## 定位（节点）

当设置中启用定位时，节点暴露 `location.get`。CLI 辅助命令：

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

注意：

-   定位**默认关闭**。
-   “始终允许”需要系统权限；后台获取是尽力而为的。
-   响应包含纬度/经度、精度（米）和时间戳。

## 短信（Android 节点）

当用户授予**短信**权限且设备支持电话功能时，Android 节点可以暴露 `sms.send`。底层调用：

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

注意：

-   在 Android 设备上接受权限提示后，该功能才会被广告。
-   没有电话功能的纯 Wi-Fi 设备不会广告 `sms.send`。

## Android 设备与个人数据命令

当启用相应功能时，Android 节点可以广告额外的命令族。可用族：

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

调用示例：

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

注意：

-   运动命令受可用传感器能力限制。
-   `app.update` 受节点运行时的权限和策略限制。

## 系统命令（节点主机 / mac 节点）

macOS 节点暴露 `system.run`、`system.notify` 和 `system.execApprovals.get/set`。无头节点主机暴露 `system.run`、`system.which` 和 `system.execApprovals.get/set`。示例：

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

注意：

-   `system.run` 在负载中返回 stdout/stderr/退出码。
-   `system.notify` 尊重 macOS 应用上的通知权限状态。
-   无法识别的节点 `platform` / `deviceFamily` 元数据使用保守的默认允许列表，该列表排除 `system.run` 和 `system.which`。如果你有意为未知平台需要这些命令，请通过 `gateway.nodes.allowCommands` 显式添加它们。
-   `system.run` 支持 `--cwd`、`--env KEY=VAL`、`--command-timeout` 和 `--needs-screen-recording`。
-   对于 shell 包装器（`bash|sh|zsh ... -c/-lc`），请求作用域的 `--env` 值被缩减为显式允许列表（`TERM`、`LANG`、`LC_*`、`COLORTERM`、`NO_COLOR`、`FORCE_COLOR`）。
-   对于允许列表模式中的“始终允许”决策，已知的调度包装器（`env`、`nice`、`nohup`、`stdbuf`、`timeout`）会持久化内部可执行文件路径，而不是包装器路径。如果解包不安全，则不会自动持久化任何允许列表条目。
-   在允许列表模式的 Windows 节点主机上，通过 `cmd.exe /c` 运行的 shell 包装器需要审批（仅允许列表条目不会自动允许包装器形式）。
-   `system.notify` 支持 `--priority <passive|active|timeSensitive>` 和 `--delivery <system|overlay|auto>`。
-   节点主机忽略 `PATH` 覆盖，并剥离危险的启动/Shell 键（`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`、`SHELLOPTS`、`PS4`）。如果你需要额外的 PATH 条目，请配置节点主机服务环境（或将工具安装在标准位置），而不是通过 `--env` 传递 `PATH`。
-   在 macOS 节点模式下，`system.run` 受 macOS 应用中的执行审批控制（设置 → 执行审批）。询问/允许列表/完全允许的行为与无头节点主机相同；被拒绝的提示返回 `SYSTEM_RUN_DENIED`。
-   在无头节点主机上，`system.run` 受执行审批控制（`~/.openclaw/exec-approvals.json`）。

## Exec 节点绑定

当有多个节点可用时，你可以将 exec 绑定到特定节点。这将设置 `exec host=node` 的默认节点（并且可以按代理覆盖）。全局默认：

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

按代理覆盖：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

取消设置以允许任何节点：

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## 权限映射

节点可能在 `node.list` / `node.describe` 中包含一个 `permissions` 映射，以权限名称（例如 `screenRecording`、`accessibility`）为键，布尔值（`true` = 已授予）为值。

## 无头节点主机（跨平台）

OpenClaw 可以运行一个**无头节点主机**（无 UI），它连接到网关 WebSocket 并暴露 `system.run` / `system.which`。这在 Linux/Windows 上或用于在服务器旁运行最小化节点时很有用。启动它：

```bash
openclaw node run --host <gateway-host> --port 18789
```

注意：

-   仍然需要配对（网关将显示设备配对提示）。
-   节点主机将其节点 ID、令牌、显示名称和网关连接信息存储在 `~/.openclaw/node.json` 中。
-   执行审批通过 `~/.openclaw/exec-approvals.json` 在本地强制执行（参见[执行审批](./tools/exec-approvals.md)）。
-   在 macOS 上，无头节点主机默认在本地执行 `system.run`。设置 `OPENCLAW_NODE_EXEC_HOST=app` 可通过配套应用执行主机路由 `system.run`；添加 `OPENCLAW_NODE_EXEC_FALLBACK=0` 以要求应用主机，并在其不可用时失败关闭。
-   当网关 WS 使用 TLS 时，添加 `--tls` / `--tls-fingerprint`。

## Mac 节点模式

-   macOS 菜单栏应用作为节点连接到网关 WS 服务器（因此 `openclaw nodes …` 可对此 Mac 生效）。
-   在远程模式下，应用为网关端口打开 SSH 隧道并连接到 `localhost`。

[认证监控](./automation/auth-monitoring.md)[节点故障排除](./nodes/troubleshooting.md)