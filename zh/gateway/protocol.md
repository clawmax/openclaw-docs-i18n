

  协议与 API

  
# 网关协议

网关 WebSocket 协议是 OpenClaw 的**统一控制平面与节点传输层**。所有客户端（CLI、Web UI、macOS 应用、iOS/Android 节点、无头节点）都通过 WebSocket 连接，并在握手时声明自己的**角色**和**作用域**。

## 传输层

-   WebSocket 协议，使用 JSON 负载的文本帧。
-   第一帧**必须**是 `connect` 请求。

## 握手过程（connect）

网关 → 客户端（连接前质询）：

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

客户端 → 网关：

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

网关 → 客户端：

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

当签发设备令牌时，`hello-ok` 还会包含：

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### 节点示例

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## 帧格式

-   **请求**：`{type:"req", id, method, params}`
-   **响应**：`{type:"res", id, ok, payload|error}`
-   **事件**：`{type:"event", event, payload, seq?, stateVersion?}`

有副作用的方法需要携带**幂等键**（参见 schema 定义）。

## 角色与作用域

### 角色

-   `operator` = 控制平面客户端（CLI/UI/自动化脚本）。
-   `node` = 能力宿主（摄像头/屏幕/画布/system.run）。

### 作用域（operator）

常见作用域：

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

方法作用域只是第一道关卡。通过 `chat.send` 调用的某些斜杠命令会在其基础上执行更严格的命令级检查。例如，持久的 `/config set` 和 `/config unset` 写入操作需要 `operator.admin` 权限。

### 能力/命令/权限（node）

节点在连接时声明能力：

-   `caps`：高级能力类别。
-   `commands`：可调用命令的白名单。
-   `permissions`：细粒度开关（如 `screen.record`、`camera.capture`）。

网关将这些视为**声明**，并执行服务端白名单验证。

## 在线状态

-   `system-presence` 返回按设备身份索引的状态条目。
-   状态条目包含 `deviceId`、`roles` 和 `scopes`，这样 UI 可以在同一设备同时以**操作员**和**节点**身份连接时显示为单行。

### 节点辅助方法

-   节点可调用 `skills.bins` 获取当前技能可执行文件列表，用于自动允许检查。

### 操作员辅助方法

-   操作员可调用 `tools.catalog`（需要 `operator.read`）获取智能体的运行时工具目录。响应包含分组的工具和来源元数据：
    -   `source`：`core` 或 `plugin`
    -   `pluginId`：当 `source="plugin"` 时的插件所有者
    -   `optional`：插件工具是否为可选

## 执行审批

-   当执行请求需要审批时，网关会广播 `exec.approval.requested` 事件。
-   操作员客户端通过调用 `exec.approval.resolve` 来处理（需要 `operator.approvals` 作用域）。
-   对于 `host=node`，`exec.approval.request` 必须包含 `systemRunPlan`（规范的 `argv`/`cwd`/`rawCommand`/会话元数据）。缺少 `systemRunPlan` 的请求会被拒绝。

## 版本控制

-   `PROTOCOL_VERSION` 定义在 `src/gateway/protocol/schema.ts`。
-   客户端发送 `minProtocol` + `maxProtocol`；版本不匹配时服务器会拒绝连接。
-   Schema 和模型由 TypeBox 定义生成：
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## 认证

-   如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token`），则 `connect.params.auth.token` 必须匹配，否则连接会被关闭。
-   配对成功后，网关会签发一个**设备令牌**，作用域限定为连接的角色和权限。它会在 `hello-ok.auth.deviceToken` 中返回，客户端应持久化保存以供后续连接使用。
-   设备令牌可通过 `device.token.rotate` 和 `device.token.revoke` 进行轮换或撤销（需要 `operator.pairing` 作用域）。

## 设备身份与配对

-   节点应包含稳定的设备身份（`device.id`），由密钥对指纹派生。
-   网关为每个设备 + 角色签发令牌。
-   新设备 ID 需要配对审批，除非启用了本地自动批准。
-   **本地**连接包括本地回环地址和网关主机自身的 Tailnet 地址（这样同一主机的 Tailnet 绑定仍可自动批准）。
-   所有 WebSocket 客户端必须在 `connect` 时包含 `device` 身份（包括操作员和节点）。控制 UI 仅在启用 `gateway.controlUi.dangerouslyDisableDeviceAuth` 用于应急场景时才可省略。
-   所有连接必须对服务器提供的 `connect.challenge` nonce 进行签名。

### 设备认证迁移诊断

对于仍使用质询前签名行为的旧版客户端，`connect` 现在会在 `error.details.code` 下返回 `DEVICE_AUTH_*` 详细代码，并附带稳定的 `error.details.reason`。常见迁移失败：

| 错误消息 | details.code | details.reason | 含义 |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | 客户端省略了 `device.nonce`（或发送了空值）。 |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | 客户端使用了过时或错误的 nonce 进行签名。 |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | 签名负载与 v2 格式不匹配。 |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | 签名时间戳超出允许的偏差范围。 |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id` 与公钥指纹不匹配。 |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | 公钥格式/规范化失败。 |

迁移目标：

-   始终等待 `connect.challenge` 事件。
-   对包含服务器 nonce 的 v2 格式负载进行签名。
-   在 `connect.params.device.nonce` 中发送相同的 nonce。
-   推荐使用 `v3` 格式的签名负载，它在设备/客户端/角色/作用域/令牌/nonce 字段基础上还绑定了 `platform` 和 `deviceFamily`。
-   为保持兼容性仍接受旧版 `v2` 签名，但配对设备元数据固定仍会在重连时控制命令策略。

## TLS 与证书固定

-   WebSocket 连接支持 TLS。
-   客户端可选择固定网关证书指纹（参见 `gateway.tls` 配置以及 `gateway.remote.tlsFingerprint` 或 CLI 的 `--tls-fingerprint` 参数）。

## 协议范围

此协议暴露**完整的网关 API**（状态、频道、模型、聊天、智能体、会话、节点、审批等）。具体接口由 `src/gateway/protocol/schema.ts` 中的 TypeBox schema 定义。

[沙盒 vs 工具策略 vs 提升权限](./sandbox-vs-tool-policy-vs-elevated.md)[桥接协议](./bridge-protocol.md)