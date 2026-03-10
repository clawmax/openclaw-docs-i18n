

  网关

  
# 网关运行手册

本页面是网关（Gateway）服务的运维手册，涵盖从首次启动（Day-1）到日常运维（Day-2）的全部流程。

## 5 分钟本地启动

### 步骤 1：启动网关

```bash
openclaw gateway --port 18789
# debug/trace 信息会同步输出到终端
openclaw gateway --port 18789 --verbose
# 强制杀掉占用端口的进程，然后启动
openclaw gateway --force
```

### 步骤 2：验证服务健康状态

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

健康的基准是：`Runtime: running` 且 `RPC probe: ok`。

### 步骤 3：验证频道就绪状态

```bash
openclaw channels status --probe
```

 

> **ℹ️** 网关配置热重载会监视当前激活的配置文件路径（从 profile/state 默认值解析，或通过 `OPENCLAW_CONFIG_PATH` 环境变量指定）。默认的重载模式是 `gateway.reload.mode="hybrid"`。

## 运行时模型

网关采用单进程架构，职责如下：

- 一个常驻进程，负责消息路由、控制平面和所有频道（channel）的连接管理。
- 单一复用端口，同时承载：
  - WebSocket 控制 / RPC 通信
  - HTTP API（兼容 OpenAI、Responses、工具调用等）
  - 控制 UI（Control UI）和 Webhook
- 默认绑定模式：`loopback`（仅监听本地回环地址）。
- 默认需要认证：通过 `gateway.auth.token` 或 `gateway.auth.password` 配置，也可以用环境变量 `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`。

### 端口和绑定优先级

| 设置 | 解析顺序 |
| --- | --- |
| 网关端口 | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| 绑定模式 | CLI 参数/覆盖 → `gateway.bind` → `loopback` |

### 热重载模式

| `gateway.reload.mode` | 行为 |
| --- | --- |
| `off` | 不进行配置重载 |
| `hot` | 仅应用"热安全"的配置变更（无需重启） |
| `restart` | 遇到需要重启的变更时自动重启 |
| `hybrid`（默认） | 安全时热应用，必要时重启 |

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

**推荐做法**：使用 Tailscale 或 VPN 进行远程访问。**备选方案**：SSH 隧道。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

然后在本地让客户端连接 `ws://127.0.0.1:18789`。

> **⚠️** 即使通过 SSH 隧道访问，如果网关配置了认证，客户端仍然必须发送正确的认证信息（`token` 或 `password`）。

更多细节参考：[远程网关（Remote Gateway）](./gateway/remote.md)、[认证（Authentication）](./gateway/authentication.md)、[Tailscale](./gateway/tailscale.md)。

## 进程监管与服务生命周期

在生产环境中，建议使用进程监管工具来保证网关的可靠性。

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## 单主机多网关

大多数场景下，运行 **一个网关** 就够了。只有在需要严格隔离或冗余（比如救援配置文件）时，才考虑多实例。每个实例需要独立配置：

- 唯一的 `gateway.port`
- 唯一的 `OPENCLAW_CONFIG_PATH`
- 唯一的 `OPENCLAW_STATE_DIR`
- 唯一的 `agents.defaults.workspace`

示例：

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

详见：[多网关（Multiple Gateways）](./gateway/multiple-gateways.md)。

### 开发配置快速启动

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

开发模式会自动使用隔离的状态/配置目录，网关端口默认为 `19001`。

## 协议快速参考（运维视角）

网关协议的基本流程：

- 客户端连接后，第一条帧必须是 `connect`。
- 网关返回 `hello-ok` 快照，包含 `presence`、`health`、`stateVersion`、`uptimeMs` 以及限制/策略信息。
- 请求模式：`req(method, params)` → `res(ok/payload|error)`。
- 常见事件：`connect.challenge`、`agent`、`chat`、`presence`、`tick`、`health`、`heartbeat`、`shutdown`。

智能体（agent）的运行分为两个阶段：

1. 立即返回接受确认（`status:"accepted"`）
2. 最终返回完成响应（`status:"ok"|"error"`），中间会穿插流式 `agent` 事件。

完整协议文档见：[网关协议（Gateway Protocol）](./gateway/protocol.md)。

## 运维检查

### 存活检查（Liveness）

- 打开 WebSocket 连接，发送 `connect`。
- 期望收到包含快照的 `hello-ok` 响应。

### 就绪检查（Readiness）

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### 间隙恢复

事件不会重放。如果检测到序列号间隙，在继续处理前应先刷新状态（`health`、`system-presence`）。

## 常见故障特征

| 特征 | 可能的问题 |
| --- | --- |
| `refusing to bind gateway ... without auth` | 尝试绑定非 loopback 地址但没有配置 token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | 端口被占用 |
| `Gateway start blocked: set gateway.mode=local` | 配置被设为远程模式 |
| `unauthorized` 在连接期间 | 客户端与网关的认证信息不匹配 |

完整诊断流程请参考 [网关故障排除（Gateway Troubleshooting）](./gateway/troubleshooting.md)。

## 安全保证

- 网关协议客户端在网关不可用时会快速失败（fail-fast），不会隐式回退到直接通道模式。
- 无效的首帧或非 `connect` 帧会被直接拒绝并关闭连接。
- 优雅关闭时，网关会在关闭 socket 之前发出 `shutdown` 事件。

* * *

相关主题：

- [故障排除（Troubleshooting）](./gateway/troubleshooting.md)
- [后台进程（Background Process）](./gateway/background-process.md)
- [配置（Configuration）](./gateway/configuration.md)
- [健康检查（Health）](./gateway/health.md)
- [诊断工具（Doctor）](./gateway/doctor.md)
- [认证（Authentication）](./gateway/authentication.md)

[配置（Configuration）](./gateway/configuration.md)