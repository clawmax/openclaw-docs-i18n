

  网关

  
# 网关运行手册

使用本页面进行网关服务的 Day-1 启动和 Day-2 运维。

## 5 分钟本地启动

### 步骤 1：启动网关

```bash
openclaw gateway --port 18789
# debug/trace 镜像输出到标准输出
openclaw gateway --port 18789 --verbose
# 强制终止选定端口上的监听器，然后启动
openclaw gateway --force
```

### 步骤 2：验证服务健康状态

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

健康基线：`Runtime: running` 且 `RPC probe: ok`。

### 步骤 3：验证通道就绪状态

```bash
openclaw channels status --probe
```

 

> **ℹ️** 网关配置重载会监视活动配置文件路径（从配置文件/状态默认值解析，或当设置了 `OPENCLAW_CONFIG_PATH` 时使用该值）。默认模式为 `gateway.reload.mode="hybrid"`。

## 运行时模型

-   一个常驻进程，用于路由、控制平面和通道连接。
-   单一复用端口用于：
    -   WebSocket 控制/RPC
    -   HTTP API（OpenAI 兼容、响应、工具调用）
    -   控制界面和钩子
-   默认绑定模式：`loopback`。
-   默认需要认证（`gateway.auth.token` / `gateway.auth.password`，或 `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`）。

### 端口和绑定优先级

| 设置 | 解析顺序 |
| --- | --- |
| 网关端口 | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| 绑定模式 | CLI/覆盖 → `gateway.bind` → `loopback` |

### 热重载模式

| `gateway.reload.mode` | 行为 |
| --- | --- |
| `off` | 不重载配置 |
| `hot` | 仅应用热安全的变更 |
| `restart` | 在需要重启的变更时重启 |
| `hybrid` (默认) | 安全时热应用，需要时重启 |

## 运维命令集

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## 远程访问

首选：Tailscale/VPN。备选：SSH 隧道。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

然后客户端连接到本地的 `ws://127.0.0.1:18789`。

> **⚠️** 如果网关配置了认证，即使通过 SSH 隧道，客户端也必须发送认证信息（`token`/`password`）。

 参见：[远程网关](./gateway/remote.md)、[认证](./gateway/authentication.md)、[Tailscale](./gateway/tailscale.md)。

## 监管与服务生命周期

对于类生产环境的可靠性，请使用受监管的运行方式。

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## 单主机多网关

大多数设置应运行**一个**网关。仅在需要严格隔离/冗余（例如救援配置文件）时使用多个。每个实例的检查清单：

-   唯一的 `gateway.port`
-   唯一的 `OPENCLAW_CONFIG_PATH`
-   唯一的 `OPENCLAW_STATE_DIR`
-   唯一的 `agents.defaults.workspace`

示例：

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

参见：[多网关](./gateway/multiple-gateways.md)。

### 开发配置文件快速路径

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

默认值包括隔离的状态/配置和基础网关端口 `19001`。

## 协议快速参考（运维视角）

-   第一个客户端帧必须是 `connect`。
-   网关返回 `hello-ok` 快照（`presence`、`health`、`stateVersion`、`uptimeMs`、限制/策略）。
-   请求：`req(method, params)` → `res(ok/payload|error)`。
-   常见事件：`connect.challenge`、`agent`、`chat`、`presence`、`tick`、`health`、`heartbeat`、`shutdown`。

智能体运行分为两个阶段：

1.  立即接受确认（`status:"accepted"`）
2.  最终完成响应（`status:"ok"|"error"`），中间穿插流式 `agent` 事件。

参见完整协议文档：[网关协议](./gateway/protocol.md)。

## 运维检查

### 存活状态

-   打开 WebSocket 并发送 `connect`。
-   期望收到包含快照的 `hello-ok` 响应。

### 就绪状态

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### 间隙恢复

事件不会重放。在序列出现间隙时，继续之前请刷新状态（`health`、`system-presence`）。

## 常见故障特征

| 特征 | 可能的问题 |
| --- | --- |
| `refusing to bind gateway ... without auth` | 非环回绑定但未设置 token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | 端口冲突 |
| `Gateway start blocked: set gateway.mode=local` | 配置设置为远程模式 |
| `unauthorized` 在连接期间 | 客户端与网关之间的认证不匹配 |

完整的诊断阶梯，请使用[网关故障排除](./gateway/troubleshooting.md)。

## 安全保证

-   网关协议客户端在网关不可用时快速失败（无隐式直接通道回退）。
-   无效/非连接的首帧会被拒绝并关闭连接。
-   优雅关闭会在套接字关闭前发出 `shutdown` 事件。

* * *

相关：

-   [故障排除](./gateway/troubleshooting.md)
-   [后台进程](./gateway/background-process.md)
-   [配置](./gateway/configuration.md)
-   [健康检查](./gateway/health.md)
-   [诊断工具](./gateway/doctor.md)
-   [认证](./gateway/authentication.md)

[配置](./gateway/configuration.md)