

  安全

  
# 形式化验证（安全模型）

本页面跟踪 OpenClaw 的**形式化安全模型**（目前为 TLA+/TLC；未来视需要增加）。

> 注意：一些旧链接可能引用之前的项目名称。

**目标（北极星）：** 在明确的假设下，提供机器可检查的论证，证明 OpenClaw 强制执行其预期的安全策略（授权、会话隔离、工具门控和配置错误安全）。**当前状态：** 一个可执行的、攻击者驱动的**安全回归测试套件**：

-   每个声明都有一个在有限状态空间上可运行的模型检查。
-   许多声明配有一个**负面模型**，用于为现实的缺陷类别生成反例轨迹。

**目前尚不是：** 一个证明“OpenClaw 在所有方面都是安全的”或完整的 TypeScript 实现是正确的。

## 模型存放位置

模型维护在单独的代码仓库中：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

## 重要注意事项

-   这些是**模型**，而非完整的 TypeScript 实现。模型与代码之间可能存在差异。
-   结果受 TLC 探索的状态空间限制；“绿色”并不意味着超出建模假设和边界的安全性。
-   一些声明依赖于明确的环境假设（例如，正确的部署、正确的配置输入）。

## 复现结果

目前，通过克隆模型仓库到本地并运行 TLC 来复现结果（见下文）。未来的迭代可能提供：

-   通过 CI 运行的模型及公共产物（反例轨迹、运行日志）
-   用于小型、有边界检查的托管“运行此模型”工作流

开始使用：

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 需要 Java 11+（TLC 运行在 JVM 上）。
# 该仓库提供了固定的 `tla2tools.jar`（TLA+ 工具）以及 `bin/tlc` 和 Make 目标。

make <target>
```

### 网关暴露与开放网关错误配置

**声明：** 在非本地回环地址上绑定且无身份验证可能导致远程攻击成为可能/增加暴露风险；令牌/密码根据模型假设阻止未经授权的攻击者。

-   绿色运行：
    -   `make gateway-exposure-v2`
    -   `make gateway-exposure-v2-protected`
-   红色（预期）：
    -   `make gateway-exposure-v2-negative`

另请参阅：模型仓库中的 `docs/gateway-exposure-matrix.md`。

### Nodes.run 流水线（最高风险能力）

**声明：** `nodes.run` 需要 (a) 节点命令允许列表加上声明的命令，以及 (b) 配置时需实时批准；批准被令牌化以防止重放（在模型中）。

-   绿色运行：
    -   `make nodes-pipeline`
    -   `make approvals-token`
-   红色（预期）：
    -   `make nodes-pipeline-negative`
    -   `make approvals-token-negative`

### 配对存储（DM 门控）

**声明：** 配对请求遵守 TTL 和待处理请求上限。

-   绿色运行：
    -   `make pairing`
    -   `make pairing-cap`
-   红色（预期）：
    -   `make pairing-negative`
    -   `make pairing-cap-negative`

### 入口门控（提及 + 控制命令绕过）

**声明：** 在需要提及的群组上下文中，未经授权的“控制命令”无法绕过提及门控。

-   绿色：
    -   `make ingress-gating`
-   红色（预期）：
    -   `make ingress-gating-negative`

### 路由/会话密钥隔离

**声明：** 来自不同对等方的私信不会合并到同一会话中，除非明确链接/配置。

-   绿色：
    -   `make routing-isolation`
-   红色（预期）：
    -   `make routing-isolation-negative`

## v1++：额外的有界模型（并发、重试、轨迹正确性）

这些是后续模型，旨在提高围绕现实世界故障模式（非原子更新、重试和消息扇出）的保真度。

### 配对存储并发 / 幂等性

**声明：** 配对存储即使在交错执行下也应强制执行 `MaxPending` 和幂等性（即，“先检查后写入”必须是原子的/加锁的；刷新不应创建重复项）。这意味着：

-   在并发请求下，不能超过通道的 `MaxPending`。
-   对相同 `(channel, sender)` 的重复请求/刷新不应创建重复的活跃待处理行。
-   绿色运行：
    -   `make pairing-race`（原子/加锁的上限检查）
    -   `make pairing-idempotency`
    -   `make pairing-refresh`
    -   `make pairing-refresh-race`
-   红色（预期）：
    -   `make pairing-race-negative`（非原子的开始/提交上限竞争）
    -   `make pairing-idempotency-negative`
    -   `make pairing-refresh-negative`
    -   `make pairing-refresh-race-negative`

### 入口轨迹关联 / 幂等性

**声明：** 摄取应在扇出过程中保持轨迹关联，并在提供者重试下保持幂等性。这意味着：

-   当一个外部事件变成多个内部消息时，每个部分都保持相同的轨迹/事件标识。
-   重试不会导致重复处理。
-   如果提供者事件 ID 缺失，去重将回退到安全的键（例如，轨迹 ID）以避免丢弃不同的事件。
-   绿色：
    -   `make ingress-trace`
    -   `make ingress-trace2`
    -   `make ingress-idempotency`
    -   `make ingress-dedupe-fallback`
-   红色（预期）：
    -   `make ingress-trace-negative`
    -   `make ingress-trace2-negative`
    -   `make ingress-idempotency-negative`
    -   `make ingress-dedupe-fallback-negative`

### 路由 dmScope 优先级 + identityLinks

**声明：** 路由必须默认保持私信会话隔离，并且仅在明确配置时才会合并会话（通道优先级 + 身份链接）。这意味着：

-   特定于通道的 dmScope 覆盖必须优先于全局默认值。
-   identityLinks 应仅在显式链接的组内合并会话，而不是在不相关的对等方之间。
-   绿色：
    -   `make routing-precedence`
    -   `make routing-identitylinks`
-   红色（预期）：
    -   `make routing-precedence-negative`
    -   `make routing-identitylinks-negative`

[Tailscale](../gateway/tailscale.md)[README](./README.md)