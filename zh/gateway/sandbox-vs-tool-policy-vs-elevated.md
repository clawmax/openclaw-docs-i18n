

  安全与沙盒化

  
# 沙盒 vs 工具策略 vs 提升模式

OpenClaw 有三类相关（但不同）的控制：

1.  **沙盒** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) 决定**工具在何处运行**（Docker 还是宿主机）。
2.  **工具策略** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) 决定**哪些工具可用/被允许**。
3.  **提升模式** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) 是一个**仅限 exec 的逃生舱**，用于在沙盒化时在宿主机上运行。

## 快速调试

使用检查器查看 OpenClaw *实际*在做什么：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

它会打印：

-   生效的沙盒模式/范围/工作区访问权限
-   会话当前是否处于沙盒中（主会话 vs 非主会话）
-   生效的沙盒工具允许/拒绝列表（以及它来自代理/全局/默认配置）
-   提升模式的关卡和修复键路径

## 沙盒：工具在何处运行

沙盒化由 `agents.defaults.sandbox.mode` 控制：

-   `"off"`：所有内容都在宿主机上运行。
-   `"non-main"`：只有非主会话被沙盒化（群组/频道的常见“意外”情况）。
-   `"all"`：所有内容都被沙盒化。

完整矩阵（范围、工作区挂载、镜像）请参阅[沙盒化](./sandboxing.md)。

### 绑定挂载（安全检查要点）

-   `docker.binds` *穿透*沙盒文件系统：无论你挂载什么，都会以你设置的权限（`:ro` 或 `:rw`）在容器内可见。
-   如果省略权限，默认是读写；对于源代码/密钥，建议使用 `:ro`。
-   `scope: "shared"` 会忽略每个代理的绑定（仅应用全局绑定）。
-   绑定 `/var/run/docker.sock` 实际上会将宿主机控制权交给沙盒；请仅在有意时这样做。
-   工作区访问权限 (`workspaceAccess: "ro"`/`"rw"`) 独立于绑定权限。

## 工具策略：哪些工具存在/可调用

有两层很重要：

-   **工具配置文件**：`tools.profile` 和 `agents.list[].tools.profile`（基础允许列表）
-   **提供商工具配置文件**：`tools.byProvider[provider].profile` 和 `agents.list[].tools.byProvider[provider].profile`
-   **全局/每代理工具策略**：`tools.allow`/`tools.deny` 和 `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **提供商工具策略**：`tools.byProvider[provider].allow/deny` 和 `agents.list[].tools.byProvider[provider].allow/deny`
-   **沙盒工具策略**（仅在沙盒化时适用）：`tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` 和 `agents.list[].tools.sandbox.tools.*`

经验法则：

-   `deny` 总是优先。
-   如果 `allow` 非空，则其他所有内容都被视为被阻止。
-   工具策略是硬性限制：`/exec` 无法覆盖被拒绝的 `exec` 工具。
-   `/exec` 仅更改授权发送者的会话默认值；它不授予工具访问权限。提供商工具键接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。

### 工具组（简写）

工具策略（全局、代理、沙盒）支持 `group:*` 条目，这些条目会扩展为多个工具：

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

可用组：

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供商插件）

## 提升模式：仅限 exec 的“在宿主机上运行”

提升模式**不**授予额外工具；它只影响 `exec`。

-   如果你处于沙盒中，`/elevated on`（或带有 `elevated: true` 的 `exec`）将在宿主机上运行（可能仍需审批）。
-   使用 `/elevated full` 可以跳过该会话的 exec 审批。
-   如果你已经在直接运行，提升模式实际上是无操作的（但仍受关卡限制）。
-   提升模式**不**是技能范围的，也**不**覆盖工具允许/拒绝列表。
-   `/exec` 与提升模式是分开的。它仅调整授权发送者的每会话 exec 默认值。

关卡：

-   启用：`tools.elevated.enabled`（以及可选的 `agents.list[].tools.elevated.enabled`）
-   发送者允许列表：`tools.elevated.allowFrom.`（以及可选的 `agents.list[].tools.elevated.allowFrom.`）

请参阅[提升模式](../tools/elevated.md)。

## 常见的“沙盒隔离”修复方法

### “工具 X 被沙盒工具策略阻止”

修复键（任选其一）：

-   禁用沙盒：`agents.defaults.sandbox.mode=off`（或每代理 `agents.list[].sandbox.mode=off`）
-   在沙盒内允许该工具：
    -   将其从 `tools.sandbox.tools.deny` 中移除（或每代理 `agents.list[].tools.sandbox.tools.deny`）
    -   或将其添加到 `tools.sandbox.tools.allow` 中（或每代理允许列表）

### “我以为这是主会话，为什么它被沙盒化了？”

在 `"non-main"` 模式下，群组/频道密钥*不是*主会话。使用主会话密钥（由 `sandbox explain` 显示）或将模式切换为 `"off"`。

[沙盒化](./sandboxing.md)[网关协议](./protocol.md)