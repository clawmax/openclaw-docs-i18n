

  内置工具

  
# Exec Approvals

Exec approvals 是**配套应用 / 节点主机防护栏**，用于让沙箱化代理在真实主机（`gateway` 或 `node`）上运行命令。可以将其视为安全联锁：只有当策略 + 允许列表 +（可选的）用户批准全部同意时，命令才被允许。Exec approvals 是**附加于**工具策略和提升门控之外的（除非提升模式设置为 `full`，这会跳过审批）。有效策略是 `tools.exec.*` 和 approvals 默认值中**更严格**的那个；如果 approvals 字段被省略，则使用 `tools.exec` 的值。如果配套应用 UI **不可用**，任何需要提示的请求将由**询问回退**（默认：拒绝）解决。

## 适用范围

Exec approvals 在执行主机上本地强制执行：

-   **网关主机** → 网关机器上的 `openclaw` 进程
-   **节点主机** → 节点运行器（macOS 配套应用或无头节点主机）

macOS 拆分：

-   **节点主机服务** 通过本地 IPC 将 `system.run` 转发给 **macOS 应用**。
-   **macOS 应用** 强制执行 approvals 并在 UI 上下文中执行命令。

## 设置与存储

Approvals 存储在执行主机上的本地 JSON 文件中：`~/.openclaw/exec-approvals.json` 示例模式：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 策略旋钮

### 安全 (exec.security)

-   **deny**: 阻止所有主机执行请求。
-   **allowlist**: 仅允许已列入允许列表的命令。
-   **full**: 允许所有命令（等同于提升模式）。

### 询问 (exec.ask)

-   **off**: 从不提示。
-   **on-miss**: 仅在允许列表不匹配时提示。
-   **always**: 对每个命令都提示。

### 询问回退 (askFallback)

如果需要提示但无法访问 UI，回退决定：

-   **deny**: 阻止。
-   **allowlist**: 仅当允许列表匹配时允许。
-   **full**: 允许。

## 允许列表（按代理）

允许列表是**按代理**的。如果存在多个代理，请在 macOS 应用中切换正在编辑的代理。模式是**不区分大小写的通配符匹配**。模式应解析为**二进制文件路径**（仅包含文件名的条目将被忽略）。旧的 `agents.default` 条目在加载时会迁移到 `agents.main`。示例：

-   `~/Projects/**/bin/peekaboo`
-   `~/.local/bin/*`
-   `/opt/homebrew/bin/rg`

每个允许列表条目跟踪：

-   **id** 用于 UI 标识的稳定 UUID（可选）
-   **最后使用** 时间戳
-   **最后使用的命令**
-   **最后解析的路径**

## 自动允许技能 CLI

当启用**自动允许技能 CLI**时，已知技能引用的可执行文件在节点（macOS 节点或无头节点主机）上被视为已列入允许列表。这通过 Gateway RPC 使用 `skills.bins` 来获取技能二进制列表。如果您想要严格的手动允许列表，请禁用此功能。

## 安全二进制文件（仅限标准输入）

`tools.exec.safeBins` 定义了一个小的**仅限标准输入**的二进制文件列表（例如 `jq`），这些文件可以在允许列表模式下运行，**无需**显式的允许列表条目。安全二进制文件拒绝位置文件参数和类似路径的标记，因此它们只能对传入的流进行操作。验证仅从 argv 形状确定性地进行（不检查主机文件系统是否存在），这可以防止因允许/拒绝差异而产生的文件存在性探测行为。对于默认的安全二进制文件，面向文件的选项被拒绝（例如 `sort -o`、`sort --output`、`sort --files0-from`、`sort --compress-program`、`wc --files0-from`、`jq -f/--from-file`、`grep -f/--file`）。安全二进制文件还对破坏仅限标准输入行为的选项强制执行显式的每二进制文件标志策略（例如 `sort -o/--output/--compress-program` 和 grep 的递归标志）。安全二进制文件还强制在仅限标准输入的段中，在执行时将 argv 标记视为**字面文本**（不进行通配符扩展，也不进行 `$VARS` 扩展），因此像 `*` 或 `$HOME/...` 这样的模式不能用于偷运文件读取。安全二进制文件还必须从受信任的二进制目录解析（系统默认目录加上网关进程启动时的 `PATH`）。这阻止了请求范围的 PATH 劫持尝试。Shell 链和重定向在允许列表模式下不会自动允许。当每个顶级段都满足允许列表条件（包括安全二进制文件或技能自动允许）时，允许 Shell 链（`&&`、`||`、`;`）。重定向在允许列表模式下仍然不受支持。命令替换（`$()` / 反引号）在允许列表解析期间被拒绝，包括在双引号内；如果需要字面的 `$()` 文本，请使用单引号。在 macOS 配套应用 approvals 中，包含 shell 控制或扩展语法（`&&`、`||`、`;`、`|`、```、`$`、`<`、`>`、`(`、`)`）的原始 shell 文本被视为允许列表未命中，除非 shell 二进制文件本身已列入允许列表。默认安全二进制文件：`jq`、`cut`、`uniq`、`head`、`tail`、`tr`、`wc`。`grep` 和 `sort` 不在默认列表中。如果您选择加入，请为它们的非标准输入工作流保留显式的允许列表条目。对于安全二进制文件模式下的 `grep`，请使用 `-e`/`--regexp` 提供模式；位置模式形式被拒绝，因此文件操作数不能作为模糊的位置参数偷运。

## 控制 UI 编辑

使用 **Control UI → Nodes → Exec approvals** 卡片来编辑默认值、每个代理的覆盖项和允许列表。选择一个范围（Defaults 或一个代理），调整策略，添加/删除允许列表模式，然后**保存**。UI 显示每个模式的**最后使用**元数据，以便您保持列表整洁。目标选择器选择**网关**（本地 approvals）或一个**节点**。节点必须通告 `system.execApprovals.get/set`（macOS 应用或无头节点主机）。如果节点尚未通告 exec approvals，请直接编辑其本地的 `~/.openclaw/exec-approvals.json`。CLI：`openclaw approvals` 支持网关或节点编辑（参见 [Approvals CLI](/cli/approvals)）。

## 审批流程

当需要提示时，网关向操作员客户端广播 `exec.approval.requested`。Control UI 和 macOS 应用通过 `exec.approval.resolve` 解决它，然后网关将批准的请求转发给节点主机。当需要 approvals 时，exec 工具会立即返回一个 approval id。使用该 id 来关联后续的系统事件（`Exec finished` / `Exec denied`）。如果在超时前没有做出决定，请求将被视为审批超时，并作为拒绝原因显示。确认对话框包括：

-   命令 + 参数
-   当前工作目录
-   代理 id
-   解析后的可执行文件路径
-   主机 + 策略元数据

操作：

-   **允许一次** → 立即运行
-   **始终允许** → 添加到允许列表 + 运行
-   **拒绝** → 阻止

## 将审批转发到聊天频道

您可以将 exec 审批提示转发到任何聊天频道（包括插件频道），并使用 `/approve` 批准它们。这使用正常的外发交付管道。配置：

```json
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // 子字符串或正则表达式
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

在聊天中回复：

```bash
/approve  allow-once
/approve  allow-always
/approve  deny
```

### macOS IPC 流程

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

安全说明：

-   Unix 套接字模式 `0600`，令牌存储在 `exec-approvals.json` 中。
-   相同 UID 对等检查。
-   质询/响应（随机数 + HMAC 令牌 + 请求哈希）+ 短 TTL。

## 系统事件

Exec 生命周期作为系统消息显示：

-   `Exec running`（仅当命令超过运行通知阈值时）
-   `Exec finished`
-   `Exec denied`

这些消息在节点报告事件后发布到代理的会话中。网关主机 exec approvals 在命令完成时（以及可选地在运行时间超过阈值时）发出相同的生命周期事件。需要审批的 exec 在这些消息中重用 approval id 作为 `runId`，以便于关联。

## 影响

-   **full** 功能强大；尽可能首选允许列表。
-   **ask** 让您保持知情，同时仍允许快速批准。
-   每个代理的允许列表可防止一个代理的审批泄露到其他代理。
-   Approvals 仅适用于来自**授权发送者**的主机执行请求。未经授权的发送者无法发出 `/exec`。
-   `/exec security=full` 是授权操作员的会话级便利功能，设计上会跳过 approvals。要硬性阻止主机执行，请将 approvals 安全设置为 `deny` 或通过工具策略拒绝 `exec` 工具。

相关：

-   [Exec 工具](/tools/exec)
-   [提升模式](/tools/elevated)
-   [技能](/tools/skills)

[Exec 工具](/tools/exec)[Firecrawl](/tools/firecrawl)