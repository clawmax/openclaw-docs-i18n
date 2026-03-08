

  多智能体

  
# Presence

OpenClaw “presence” 是一个轻量级的、尽力而为的视图，用于展示：

-   **网关**本身，以及
-   **连接到网关的客户端**（mac 应用、WebChat、CLI 等）

Presence 主要用于渲染 macOS 应用的**实例**选项卡，并为操作员提供快速的可见性。

## Presence 字段（显示内容）

Presence 条目是具有以下字段的结构化对象：

-   `instanceId`（可选但强烈推荐）：稳定的客户端标识（通常是 `connect.client.instanceId`）
-   `host`：易于理解的主机名
-   `ip`：尽力而为的 IP 地址
-   `version`：客户端版本字符串
-   `deviceFamily` / `modelIdentifier`：硬件提示
-   `mode`：`ui`、`webchat`、`cli`、`backend`、`probe`、`test`、`node`、…
-   `lastInputSeconds`：“自上次用户输入以来的秒数”（如果已知）
-   `reason`：`self`、`connect`、`node-connected`、`periodic`、…
-   `ts`：最后更新时间戳（自纪元以来的毫秒数）

## 生产者（presence 的来源）

Presence 条目由多个来源产生并**合并**。

### 1) 网关自身条目

网关在启动时总是会生成一个“self”条目，这样即使在任何客户端连接之前，UI 也能显示网关主机。

### 2) WebSocket 连接

每个 WS 客户端都以一个 `connect` 请求开始。在成功握手后，网关会为该连接更新或插入一个 presence 条目。

#### 为什么一次性 CLI 命令不显示

CLI 经常为短期的、一次性的命令建立连接。为了避免刷屏实例列表，`client.mode === "cli"` **不会**被转换为 presence 条目。

### 3) system-event 信标

客户端可以通过 `system-event` 方法发送更丰富的周期性信标。mac 应用使用此方法来报告主机名、IP 和 `lastInputSeconds`。

### 4) 节点连接（角色：node）

当一个节点通过网关 WebSocket 以 `role: node` 连接时，网关会为该节点更新或插入一个 presence 条目（与其他 WS 客户端流程相同）。

## 合并 + 去重规则（为什么 instanceId 很重要）

Presence 条目存储在单个内存映射中：

-   条目由一个 **presence 键** 作为键。
-   最好的键是一个稳定的 `instanceId`（来自 `connect.client.instanceId`），它能在重启后保持不变。
-   键不区分大小写。

如果一个客户端在没有稳定 `instanceId` 的情况下重新连接，它可能会显示为**重复**行。

## TTL 和有限大小

Presence 被设计为临时性的：

-   **TTL：** 超过 5 分钟的条目会被清除
-   **最大条目数：** 200（最旧的条目优先被丢弃）

这可以保持列表的新鲜度，并避免内存无限增长。

## 远程/隧道注意事项（回环 IP）

当客户端通过 SSH 隧道 / 本地端口转发连接时，网关看到的远程地址可能是 `127.0.0.1`。为了避免覆盖客户端报告的正确 IP，回环远程地址会被忽略。

## 消费者

### macOS 实例选项卡

macOS 应用渲染 `system-presence` 的输出，并根据最后更新的时间应用一个小的状态指示器（活跃/空闲/陈旧）。

## 调试技巧

-   要查看原始列表，请对网关调用 `system-presence`。
-   如果看到重复条目：
    -   确认客户端在握手中发送了稳定的 `client.instanceId`
    -   确认周期性信标使用了相同的 `instanceId`
    -   检查连接派生的条目是否缺少 `instanceId`（出现重复是预期的）

[多智能体路由](./multi-agent.md)[消息](./messages.md)

---