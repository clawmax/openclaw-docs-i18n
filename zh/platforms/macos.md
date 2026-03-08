

  平台概览

  
# macOS 应用

macOS 应用是 OpenClaw 的**菜单栏伴侣**。它负责权限管理、在本地管理/连接到网关（通过 launchd 或手动方式），并将 macOS 能力作为一个节点暴露给智能体。

## 功能概述

-   在菜单栏显示原生通知和状态。
-   拥有 TCC 提示权限（通知、辅助功能、屏幕录制、麦克风、语音识别、自动化/AppleScript）。
-   运行或连接到网关（本地或远程）。
-   暴露 macOS 专属工具（画布、摄像头、屏幕录制、`system.run`）。
-   在**远程**模式下启动本地节点主机服务（通过 launchd），在**本地**模式下停止该服务。
-   可选托管 **PeekabooBridge** 以进行 UI 自动化。
-   根据请求通过 npm/pnpm 安装全局 CLI（`openclaw`）（不推荐使用 bun 作为网关运行时）。

## 本地模式 vs 远程模式

-   **本地**（默认）：如果存在正在运行的本地网关，应用会连接到它；否则，它会通过 `openclaw gateway install` 启用 launchd 服务。
-   **远程**：应用通过 SSH/Tailscale 连接到远程网关，并且从不启动本地进程。应用会启动本地**节点主机服务**，以便远程网关可以访问此 Mac。应用不会将网关作为子进程启动。

## Launchd 控制

应用管理一个按用户划分的 LaunchAgent，标签为 `ai.openclaw.gateway`（或使用 `--profile`/`OPENCLAW_PROFILE` 时的 `ai.openclaw.`；旧版 `com.openclaw.*` 仍会被卸载）。

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

运行命名配置文件时，请将标签替换为 `ai.openclaw.`。如果 LaunchAgent 未安装，请从应用中启用它或运行 `openclaw gateway install`。

## 节点能力 (mac)

macOS 应用将自己呈现为一个节点。常用命令：

-   画布：`canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
-   摄像头：`camera.snap`, `camera.clip`
-   屏幕：`screen.record`
-   系统：`system.run`, `system.notify`

节点会报告一个 `permissions` 映射，以便智能体决定允许哪些操作。节点服务 + 应用 IPC：

-   当无头节点主机服务运行时（远程模式），它会作为节点连接到网关 WebSocket。
-   `system.run` 通过本地 Unix 套接字在 macOS 应用（UI/TCC 上下文）中执行；提示和输出保留在应用内。

示意图 (SCI)：

```
网关 -> 节点服务 (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac 应用 (UI + TCC + system.run)
```

## 执行审批 (system.run)

`system.run` 由 macOS 应用中的**执行审批**控制（设置 → 执行审批）。安全设置、询问规则和允许列表都本地存储在 Mac 上的：

```
~/.openclaw/exec-approvals.json
```

示例：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

注意：

-   `allowlist` 条目是用于解析后二进制路径的 glob 模式。
-   包含 shell 控制或扩展语法（`&&`、`||`、`;`、`|`、```、`$`、`<`、`>`、`(`、`)`）的原始 shell 命令文本将被视为允许列表未命中，需要显式批准（或将 shell 二进制文件加入允许列表）。
-   在提示中选择“始终允许”会将该命令添加到允许列表中。
-   `system.run` 的环境变量覆盖会被过滤（移除 `PATH`、`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`、`SHELLOPTS`、`PS4`），然后与应用的环境合并。
-   对于 shell 包装器（`bash|sh|zsh ... -c/-lc`），请求作用域的环境变量覆盖会减少到一个小的显式允许列表（`TERM`、`LANG`、`LC_*`、`COLORTERM`、`NO_COLOR`、`FORCE_COLOR`）。
-   对于允许列表模式中的“始终允许”决定，已知的调度包装器（`env`、`nice`、`nohup`、`stdbuf`、`timeout`）会保留内部可执行文件路径，而不是包装器路径。如果解包不安全，则不会自动保留任何允许列表条目。

## 深度链接

应用注册了 `openclaw://` URL 方案用于本地操作。

### openclaw://agent

触发网关 `agent` 请求。

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

查询参数：

-   `message`（必需）
-   `sessionKey`（可选）
-   `thinking`（可选）
-   `deliver` / `to` / `channel`（可选）
-   `timeoutSeconds`（可选）
-   `key`（可选的无值守模式密钥）

安全性：

-   没有 `key` 时，应用会提示确认。
-   没有 `key` 时，应用会强制执行短消息限制以显示确认提示，并忽略 `deliver` / `to` / `channel` 参数。
-   使用有效的 `key` 时，运行处于无值守状态（适用于个人自动化）。

## 入门流程（典型）

1.  安装并启动 **OpenClaw.app**。
2.  完成权限检查清单（TCC 提示）。
3.  确保**本地**模式处于活动状态且网关正在运行。
4.  如果需要终端访问，请安装 CLI。

## 状态目录位置 (macOS)

避免将 OpenClaw 状态目录放在 iCloud 或其他云同步文件夹中。同步支持的路径可能会增加延迟，并偶尔导致会话和凭据的文件锁/同步竞争。建议使用本地非同步状态路径，例如：

```
OPENCLAW_STATE_DIR=~/.openclaw
```

如果 `openclaw doctor` 检测到状态位于以下路径：

-   `~/Library/Mobile Documents/com~apple~CloudDocs/...`
-   `~/Library/CloudStorage/...`

它会发出警告并建议移回本地路径。

## 构建与开发工作流（原生）

-   `cd apps/macos && swift build`
-   `swift run OpenClaw`（或使用 Xcode）
-   打包应用：`scripts/package-mac-app.sh`

## 调试网关连接性 (macOS CLI)

使用调试 CLI 来执行与 macOS 应用相同的网关 WebSocket 握手和发现逻辑，而无需启动应用。

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

连接选项：

-   `--url <ws://host:port>`：覆盖配置
-   `--mode <local|remote>`：从配置解析（默认：配置或本地）
-   `--probe`：强制进行新的健康探测
-   `--timeout <ms>`：请求超时（默认：`15000`）
-   `--json`：用于差异比较的结构化输出

发现选项：

-   `--include-local`：包含会被过滤为“本地”的网关
-   `--timeout <ms>`：整体发现窗口（默认：`2000`）
-   `--json`：用于差异比较的结构化输出

提示：与 `openclaw gateway discover --json` 进行比较，以查看 macOS 应用的发现管道（NWBrowser + tailnet DNS‑SD 回退）是否与 Node CLI 基于 `dns-sd` 的发现不同。

## 远程连接管道 (SSH 隧道)

当 macOS 应用在**远程**模式下运行时，它会打开一个 SSH 隧道，以便本地 UI 组件可以像访问本地主机一样与远程网关通信。

### 控制隧道（网关 WebSocket 端口）

-   **目的：** 健康检查、状态、Web 聊天、配置和其他控制平面调用。
-   **本地端口：** 网关端口（默认 `18789`），始终保持稳定。
-   **远程端口：** 远程主机上的相同网关端口。
-   **行为：** 没有随机本地端口；应用重用现有的健康隧道，或在需要时重新启动它。
-   **SSH 形态：** `ssh -N -L <local>:127.0.0.1:<remote>`，带有 BatchMode + ExitOnForwardFailure + keepalive 选项。
-   **IP 报告：** SSH 隧道使用环回地址，因此网关看到的节点 IP 将是 `127.0.0.1`。如果您希望显示真实的客户端 IP，请使用**直接 (ws/wss)** 传输方式（参见 [macOS 远程访问](/platforms/mac/remote)）。

有关设置步骤，请参阅 [macOS 远程访问](/platforms/mac/remote)。有关协议详情，请参阅 [网关协议](/gateway/protocol)。

## 相关文档

-   [网关运行手册](/gateway)
-   [网关 (macOS)](/platforms/mac/bundled-gateway)
-   [macOS 权限](/platforms/mac/permissions)
-   [画布](/platforms/mac/canvas)

[平台](/platforms)[Linux 应用](/platforms/linux)