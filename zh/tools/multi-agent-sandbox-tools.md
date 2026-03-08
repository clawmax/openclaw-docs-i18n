

  智能体协调

  
# 多智能体沙箱与工具

## 概述

多智能体设置中的每个智能体现在可以拥有自己的：

-   **沙箱配置** (`agents.list[].sandbox` 覆盖 `agents.defaults.sandbox`)
-   **工具限制** (`tools.allow` / `tools.deny`，以及 `agents.list[].tools`)

这允许您运行具有不同安全配置文件的多智能体：

-   具有完全访问权限的个人助手
-   具有受限工具的家庭/工作智能体
-   沙箱中的面向公众的智能体

`setupCommand` 属于 `sandbox.docker` 下（全局或每个智能体），并在容器创建时运行一次。身份验证是按智能体的：每个智能体从其自己的 `agentDir` 身份验证存储中读取，位置在：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

凭证**不**在智能体之间共享。切勿在智能体之间重用 `agentDir`。如果您想共享凭证，请将 `auth-profiles.json` 复制到另一个智能体的 `agentDir` 中。关于沙箱在运行时的行为，请参阅[沙箱](../gateway/sandboxing.md)。关于调试“为什么这个被阻止了？”，请参阅[沙箱 vs 工具策略 vs 提升权限](../gateway/sandbox-vs-tool-policy-vs-elevated.md)和 `openclaw sandbox explain`。

* * *

## 配置示例

### 示例 1：个人 + 受限家庭智能体

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "个人助手",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "家庭机器人",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**结果：**

-   `main` 智能体：在主机上运行，完全工具访问权限
-   `family` 智能体：在 Docker 中运行（每个智能体一个容器），仅 `read` 工具

* * *

### 示例 2：具有共享沙箱的工作智能体

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### 示例 2b：全局编码配置文件 + 仅消息智能体

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**结果：**

-   默认智能体获得编码工具
-   `support` 智能体仅限消息功能（+ Slack 工具）

* * *

### 示例 3：每个智能体的不同沙箱模式

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // 全局默认
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // 覆盖：main 从不沙箱化
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // 覆盖：public 总是沙箱化
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## 配置优先级

当同时存在全局 (`agents.defaults.*`) 和智能体特定 (`agents.list[].*`) 配置时：

### 沙箱配置

智能体特定设置覆盖全局设置：

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意：**

-   `agents.list[].sandbox.{docker,browser,prune}.*` 为该智能体覆盖 `agents.defaults.sandbox.{docker,browser,prune}.*`（当沙箱范围解析为 `"shared"` 时忽略）。

### 工具限制

过滤顺序为：

1.  **工具配置文件** (`tools.profile` 或 `agents.list[].tools.profile`)
2.  **提供商工具配置文件** (`tools.byProvider[provider].profile` 或 `agents.list[].tools.byProvider[provider].profile`)
3.  **全局工具策略** (`tools.allow` / `tools.deny`)
4.  **提供商工具策略** (`tools.byProvider[provider].allow/deny`)
5.  **智能体特定工具策略** (`agents.list[].tools.allow/deny`)
6.  **智能体提供商策略** (`agents.list[].tools.byProvider[provider].allow/deny`)
7.  **沙箱工具策略** (`tools.sandbox.tools` 或 `agents.list[].tools.sandbox.tools`)
8.  **子智能体工具策略** (`tools.subagents.tools`，如果适用)

每个级别都可以进一步限制工具，但不能恢复先前级别已拒绝的工具。如果设置了 `agents.list[].tools.sandbox.tools`，它将为该智能体替换 `tools.sandbox.tools`。如果设置了 `agents.list[].tools.profile`，它将为该智能体覆盖 `tools.profile`。提供商工具键接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。

### 工具组（简写）

工具策略（全局、智能体、沙箱）支持 `group:*` 条目，这些条目会扩展为多个具体工具：

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供商插件）

### 提升权限模式

`tools.elevated` 是全局基线（基于发送者的允许列表）。`agents.list[].tools.elevated` 可以进一步限制特定智能体的提升权限（两者都必须允许）。缓解模式：

-   为不受信任的智能体拒绝 `exec` (`agents.list[].tools.deny: ["exec"]`)
-   避免将路由到受限智能体的发送者加入允许列表
-   如果您只想要沙箱执行，则全局禁用提升权限 (`tools.elevated.enabled: false`)
-   为敏感配置文件按智能体禁用提升权限 (`agents.list[].tools.elevated.enabled: false`)

* * *

## 从单智能体迁移

**之前（单智能体）：**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**之后（具有不同配置文件的多智能体）：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

遗留的 `agent.*` 配置由 `openclaw doctor` 迁移；建议将来使用 `agents.defaults` + `agents.list`。

* * *

## 工具限制示例

### 只读智能体

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### 安全执行智能体（无文件修改）

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### 仅通信智能体

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## 常见陷阱："non-main"

`agents.defaults.sandbox.mode: "non-main"` 基于 `session.mainKey`（默认为 `"main"`），而不是智能体 id。群组/频道会话总是有自己的键，因此它们被视为非主会话并将被沙箱化。如果您希望一个智能体从不沙箱化，请设置 `agents.list[].sandbox.mode: "off"`。

* * *

## 测试

配置多智能体沙箱和工具后：

1.  **检查智能体解析：**
    
    复制
    
    ```bash
    openclaw agents list --bindings
    ```
    
2.  **验证沙箱容器：**
    
    复制
    
    ```bash
    docker ps --filter "name=openclaw-sbx-"
    ```
    
3.  **测试工具限制：**
    -   发送一条需要受限工具的消息
    -   验证智能体无法使用被拒绝的工具
4.  **监控日志：**
    
    复制
    
    ```bash
    tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
    ```
    

* * *

## 故障排除

### 尽管模式为 "all"，智能体未被沙箱化

-   检查是否存在全局 `agents.defaults.sandbox.mode` 覆盖了它
-   智能体特定配置优先，因此设置 `agents.list[].sandbox.mode: "all"`

### 尽管有拒绝列表，工具仍然可用

-   检查工具过滤顺序：全局 → 智能体 → 沙箱 → 子智能体
-   每个级别只能进一步限制，不能恢复
-   使用日志验证：`[tools] filtering tools for agent:${agentId}`

### 容器未按智能体隔离

-   在智能体特定的沙箱配置中设置 `scope: "agent"`
-   默认为 `"session"`，这会为每个会话创建一个容器

* * *

## 另请参阅

-   [多智能体路由](../concepts/multi-agent.md)
-   [沙箱配置](../gateway/configuration.md#agentsdefaults-sandbox)
-   [会话管理](../concepts/session.md)

[ACP 智能体](./acp-agents.md)[创建技能](./creating-skills.md)