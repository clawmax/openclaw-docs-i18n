

  内置工具

  
# Exec 工具

在工作区中运行 shell 命令。支持通过 `process` 进行前台 + 后台执行。如果 `process` 被禁止，`exec` 将同步运行并忽略 `yieldMs`/`background`。后台会话的作用域限定于每个代理；`process` 只能看到来自同一代理的会话。

## 参数

-   `command` (必需)
-   `workdir` (默认为当前工作目录)
-   `env` (键/值覆盖)
-   `yieldMs` (默认 10000)：延迟后自动后台运行
-   `background` (布尔值)：立即后台运行
-   `timeout` (秒，默认 1800)：到期时终止
-   `pty` (布尔值)：在可用时在伪终端中运行（仅限 TTY 的 CLI、编码代理、终端 UI）
-   `host` (`sandbox | gateway | node`)：执行位置
-   `security` (`deny | allowlist | full`)：`gateway`/`node` 的强制执行模式
-   `ask` (`off | on-miss | always`)：`gateway`/`node` 的审批提示
-   `node` (字符串)：`host=node` 时的节点 ID/名称
-   `elevated` (布尔值)：请求提升模式（网关主机）；仅当 `elevated` 解析为 `full` 时才强制 `security=full`

注意：

-   `host` 默认为 `sandbox`。
-   当沙盒关闭时，`elevated` 被忽略（exec 已在主机上运行）。
-   `gateway`/`node` 的审批由 `~/.openclaw/exec-approvals.json` 控制。
-   `node` 需要一个已配对的节点（伴侣应用或无头节点主机）。
-   如果有多个可用节点，请设置 `exec.node` 或 `tools.exec.node` 来选择一个。
-   在非 Windows 主机上，exec 在设置 `SHELL` 时使用它；如果 `SHELL` 是 `fish`，则优先使用 `PATH` 中的 `bash`（或 `sh`）以避免与 fish 不兼容的脚本，如果两者都不存在，则回退到 `SHELL`。
-   在 Windows 主机上，exec 优先发现 PowerShell 7 (`pwsh`)（Program Files、ProgramW6432，然后是 PATH），然后回退到 Windows PowerShell 5.1。
-   主机执行 (`gateway`/`node`) 拒绝 `env.PATH` 和加载器覆盖 (`LD_*`/`DYLD_*`) 以防止二进制劫持或代码注入。
-   OpenClaw 在生成的命令环境（包括 PTY 和沙盒执行）中设置 `OPENCLAW_SHELL=exec`，以便 shell/配置文件规则可以检测 exec 工具上下文。
-   重要：沙盒**默认关闭**。如果沙盒关闭且显式配置/请求了 `host=sandbox`，exec 现在会失败关闭，而不是静默在网关上运行。请启用沙盒或使用带有审批的 `host=gateway`。
-   脚本预检检查（针对常见的 Python/Node shell 语法错误）仅检查有效 `workdir` 边界内的文件。如果脚本路径解析到 `workdir` 之外，则跳过该文件的预检。

## 配置

-   `tools.exec.notifyOnExit` (默认: true)：为 true 时，后台 exec 会话在退出时排队系统事件并请求心跳。
-   `tools.exec.approvalRunningNoticeMs` (默认: 10000)：当审批门控的 exec 运行时间超过此时长时，发出单个“运行中”通知（0 表示禁用）。
-   `tools.exec.host` (默认: `sandbox`)
-   `tools.exec.security` (默认: 沙盒为 `deny`，网关 + 节点未设置时为 `allowlist`)
-   `tools.exec.ask` (默认: `on-miss`)
-   `tools.exec.node` (默认: 未设置)
-   `tools.exec.pathPrepend`：为 exec 运行（仅限网关 + 沙盒）预置到 `PATH` 的目录列表。
-   `tools.exec.safeBins`：仅限标准输入的安全二进制文件，无需显式允许列表条目即可运行。有关行为详情，请参阅[安全二进制文件](./exec-approvals.md#safe-bins-stdin-only)。
-   `tools.exec.safeBinTrustedDirs`：为 `safeBins` 路径检查额外显式信任的目录。`PATH` 条目从不自动受信任。内置默认值为 `/bin` 和 `/usr/bin`。
-   `tools.exec.safeBinProfiles`：每个安全二进制文件的可选自定义 argv 策略（`minPositional`、`maxPositional`、`allowedValueFlags`、`deniedFlags`）。

示例：

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH 处理

-   `host=gateway`：将你的登录 shell `PATH` 合并到 exec 环境中。主机执行拒绝 `env.PATH` 覆盖。守护进程本身仍以最小 `PATH` 运行：
    -   macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
    -   Linux：`/usr/local/bin`、`/usr/bin`、`/bin`
-   `host=sandbox`：在容器内运行 `sh -lc`（登录 shell），因此 `/etc/profile` 可能会重置 `PATH`。OpenClaw 在配置文件来源之后通过内部环境变量（无 shell 插值）预置 `env.PATH`；`tools.exec.pathPrepend` 在此处也适用。
-   `host=node`：仅将你传递的非阻塞环境覆盖发送到节点。主机执行拒绝 `env.PATH` 覆盖，并且节点主机会忽略它们。如果需要在节点上添加额外的 PATH 条目，请配置节点主机服务环境（systemd/launchd）或将工具安装在标准位置。

按代理的节点绑定（在配置中使用代理列表索引）：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

控制 UI：节点选项卡包含一个小的“Exec 节点绑定”面板，用于相同的设置。

## 会话覆盖 (/exec)

使用 `/exec` 为 `host`、`security`、`ask` 和 `node` 设置**每个会话**的默认值。发送不带参数的 `/exec` 以显示当前值。示例：

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## 授权模型

`/exec` 仅对**授权发送者**（通道允许列表/配对加上 `commands.useAccessGroups`）生效。它仅更新**会话状态**，不写入配置。要硬性禁用 exec，请通过工具策略拒绝它（`tools.deny: ["exec"]` 或按代理）。除非你显式设置 `security=full` 和 `ask=off`，否则主机审批仍然适用。

## Exec 审批（伴侣应用 / 节点主机）

沙盒代理可以要求在 `exec` 在网关或节点主机上运行之前进行每次请求的审批。有关策略、允许列表和 UI 流程，请参阅[Exec 审批](./exec-approvals.md)。当需要审批时，exec 工具会立即返回 `status: "approval-pending"` 和一个审批 ID。一旦批准（或拒绝/超时），网关会发出系统事件（`Exec finished` / `Exec denied`）。如果命令在 `tools.exec.approvalRunningNoticeMs` 后仍在运行，则会发出单个 `Exec running` 通知。

## 允许列表 + 安全二进制文件

手动允许列表强制执行仅匹配**已解析的二进制文件路径**（无基本名称匹配）。当 `security=allowlist` 时，仅当管道中的每个段都已允许或是安全二进制文件时，shell 命令才会自动允许。链接（`;`、`&&`、`||`）和重定向在允许列表模式下会被拒绝，除非每个顶级段都满足允许列表（包括安全二进制文件）。重定向仍然不受支持。`autoAllowSkills` 是 exec 审批中一个单独的便捷路径。它与手动路径允许列表条目不同。为了严格的显式信任，请保持 `autoAllowSkills` 禁用。使用这两个控件处理不同的任务：

-   `tools.exec.safeBins`：小的、仅限标准输入的流过滤器。
-   `tools.exec.safeBinTrustedDirs`：为安全二进制文件可执行路径额外显式信任的目录。
-   `tools.exec.safeBinProfiles`：自定义安全二进制文件的显式 argv 策略。
-   allowlist：对可执行路径的显式信任。

不要将 `safeBins` 视为通用允许列表，也不要添加解释器/运行时二进制文件（例如 `python3`、`node`、`ruby`、`bash`）。如果你需要这些，请使用显式允许列表条目并保持启用审批提示。`openclaw security audit` 会在缺少显式配置文件的解释器/运行时 `safeBins` 条目时发出警告，`openclaw doctor --fix` 可以搭建缺失的自定义 `safeBinProfiles` 条目。有关完整的策略详情和示例，请参阅[Exec 审批](./exec-approvals.md#safe-bins-stdin-only)和[安全二进制文件与允许列表](./exec-approvals.md#safe-bins-versus-allowlist)。

## 示例

前台：

```json
{ "tool": "exec", "command": "ls -la" }
```

后台 + 轮询：

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

发送按键（tmux 风格）：

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

提交（仅发送回车）：

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

粘贴（默认带括号）：

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (实验性)

`apply_patch` 是 `exec` 的一个子工具，用于结构化的多文件编辑。显式启用它：

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

注意：

-   仅适用于 OpenAI/OpenAI Codex 模型。
-   工具策略仍然适用；`allow: ["exec"]` 隐式允许 `apply_patch`。
-   配置位于 `tools.exec.applyPatch` 下。
-   `tools.exec.applyPatch.workspaceOnly` 默认为 `true`（仅限工作区）。仅当你故意希望 `apply_patch` 在工作区目录外写入/删除时，才将其设置为 `false`。

[提升模式](./elevated.md)[Exec 审批](./exec-approvals.md)

---