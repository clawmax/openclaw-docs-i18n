

  网络与发现

  
# 网关自有配对

在网关自有配对中，**网关**是决定哪些节点允许加入的权威来源。用户界面（macOS应用、未来的客户端）只是用于批准或拒绝待处理请求的前端。**重要提示：** WS节点在 `connect` 过程中使用**设备配对**（角色 `node`）。`node.pair.*` 是一个独立的配对存储，**不**控制WS握手。只有显式调用 `node.pair.*` 的客户端才使用此流程。

## 概念

-   **待处理请求**：一个请求加入的节点；需要批准。
-   **已配对节点**：已批准并获得认证令牌的节点。
-   **传输层**：网关WS端点转发请求但不决定成员资格。（遗留的TCP桥接支持已弃用/移除。）

## 配对工作原理

1.  节点连接到网关WS并请求配对。
2.  网关存储一个**待处理请求**并发出 `node.pair.requested` 事件。
3.  您批准或拒绝该请求（通过CLI或UI）。
4.  批准后，网关颁发一个**新令牌**（令牌在重新配对时会轮换）。
5.  节点使用该令牌重新连接，现在成为“已配对”状态。

待处理请求在**5分钟**后自动过期。

## CLI工作流（适用于无头环境）

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` 显示已配对/已连接的节点及其能力。

## API接口（网关协议）

事件：

-   `node.pair.requested` — 当创建新的待处理请求时发出。
-   `node.pair.resolved` — 当请求被批准/拒绝/过期时发出。

方法：

-   `node.pair.request` — 创建或重用待处理请求。
-   `node.pair.list` — 列出待处理和已配对的节点。
-   `node.pair.approve` — 批准待处理请求（颁发令牌）。
-   `node.pair.reject` — 拒绝待处理请求。
-   `node.pair.verify` — 验证 `{ nodeId, token }`。

注意：

-   `node.pair.request` 对每个节点是幂等的：重复调用返回相同的待处理请求。
-   批准**总是**生成一个新令牌；`node.pair.request` 从不返回令牌。
-   请求可能包含 `silent: true` 作为自动批准流程的提示。

## 自动批准（macOS应用）

macOS应用可以选择性地在以下情况下尝试**静默批准**：

-   请求被标记为 `silent`，并且
-   应用可以使用同一用户验证到网关主机的SSH连接。

如果静默批准失败，则回退到正常的“批准/拒绝”提示。

## 存储（本地、私有）

配对状态存储在网关状态目录下（默认为 `~/.openclaw`）：

-   `~/.openclaw/nodes/paired.json`
-   `~/.openclaw/nodes/pending.json`

如果您覆盖了 `OPENCLAW_STATE_DIR`，`nodes/` 文件夹会随之移动。安全注意事项：

-   令牌是机密；应将 `paired.json` 视为敏感文件。
-   轮换令牌需要重新批准（或删除节点条目）。

## 传输层行为

-   传输层是**无状态**的；它不存储成员资格信息。
-   如果网关离线或配对被禁用，节点无法配对。
-   如果网关处于远程模式，配对仍然针对远程网关的存储进行。

[网络模型](./network-model.md)[发现与传输层](./discovery.md)