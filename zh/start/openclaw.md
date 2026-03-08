

  指南

  
# 个人助手设置

OpenClaw 是 **Pi** 智能体的 WhatsApp + Telegram + Discord + iMessage 网关。插件可添加 Mattermost。本指南是关于“个人助手”的设置：一个专用的 WhatsApp 号码，其行为就像你全天候在线的智能体。

## ⚠️ 安全第一

你将把一个智能体置于能够：

-   在你的机器上运行命令（取决于你的 Pi 工具设置）
-   在你的工作空间中读写文件
-   通过 WhatsApp/Telegram/Discord/Mattermost（插件）发送消息出去

开始时请保持保守：

-   始终设置 `channels.whatsapp.allowFrom`（切勿在你的个人 Mac 上运行对全世界开放的服务）。
-   为助手使用一个专用的 WhatsApp 号码。
-   心跳现在默认每 30 分钟一次。在你信任此设置之前，请通过设置 `agents.defaults.heartbeat.every: "0m"` 来禁用它。

## 先决条件

-   已安装并完成初始设置的 OpenClaw — 如果尚未完成，请参阅 [入门指南](./getting-started.md)
-   用于助手的第二个电话号码（SIM/eSIM/预付费）

## 双手机设置（推荐）

你需要这样设置：如果你将个人 WhatsApp 链接到 OpenClaw，那么发给你的每条消息都会成为“智能体输入”。这通常不是你想要的。

## 5 分钟快速开始

1.  配对 WhatsApp Web（显示二维码；用助手手机扫描）：

```bash
openclaw channels login
```

2.  启动网关（保持其运行）：

```bash
openclaw gateway --port 18789
```

3.  在 `~/.openclaw/openclaw.json` 中放置一个最小配置：

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

现在从你允许列表中的手机向助手号码发送消息。当初始设置完成后，我们会自动打开仪表板并打印一个干净的（非令牌化的）链接。如果提示需要认证，请将 `gateway.auth.token` 中的令牌粘贴到控制 UI 设置中。稍后重新打开：`openclaw dashboard`。

## 为智能体提供工作空间 (AGENTS)

OpenClaw 从其工作空间目录读取操作指令和“记忆”。默认情况下，OpenClaw 使用 `~/.openclaw/workspace` 作为智能体工作空间，并会在设置/首次运行智能体时自动创建它（以及初始的 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`）。`BOOTSTRAP.md` 仅在工作空间全新时创建（删除后不应再次出现）。`MEMORY.md` 是可选的（不会自动创建）；当存在时，它会被加载用于正常会话。子智能体会话仅注入 `AGENTS.md` 和 `TOOLS.md`。提示：将此文件夹视为 OpenClaw 的“记忆”，并将其设为 git 仓库（最好是私有的），以便备份你的 `AGENTS.md` + 记忆文件。如果安装了 git，全新的工作空间会自动初始化。

```bash
openclaw setup
```

完整的工作空间布局 + 备份指南：[智能体工作空间](../concepts/agent-workspace.md) 记忆工作流：[记忆](../concepts/memory.md) 可选：使用 `agents.defaults.workspace` 选择不同的工作空间（支持 `~`）。

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

如果你已经从仓库中提供了自己的工作空间文件，可以完全禁用引导文件的创建：

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## 将其转变为“助手”的配置

OpenClaw 默认提供了一个良好的助手设置，但你通常需要调整：

-   `SOUL.md` 中的角色/指令
-   思考默认值（如果需要）
-   心跳（一旦你信任它）

示例：

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 从 0 开始；稍后启用。
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## 会话与记忆

-   会话文件：`~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   会话元数据（令牌使用情况、最后路由等）：`~/.openclaw/agents//sessions/sessions.json`（旧版：`~/.openclaw/sessions/sessions.json`）
-   `/new` 或 `/reset` 为该聊天启动一个新会话（可通过 `resetTriggers` 配置）。如果单独发送，智能体会回复一个简短的问候以确认重置。
-   `/compact [指令]` 压缩会话上下文并报告剩余的上下文预算。

## 心跳（主动模式）

默认情况下，OpenClaw 每 30 分钟运行一次心跳，提示为：`如果存在 HEARTBEAT.md 请阅读它（工作空间上下文）。严格遵守它。不要推断或重复先前聊天中的旧任务。如果无需关注任何事项，请回复 HEARTBEAT_OK。` 设置 `agents.defaults.heartbeat.every: "0m"` 以禁用。

-   如果 `HEARTBEAT.md` 存在但实际上是空的（只有空行和像 `# 标题` 这样的 Markdown 标题），OpenClaw 会跳过心跳运行以节省 API 调用。
-   如果文件缺失，心跳仍会运行，模型会决定做什么。
-   如果智能体回复 `HEARTBEAT_OK`（可选地带有简短填充；参见 `agents.defaults.heartbeat.ackMaxChars`），OpenClaw 将抑制该心跳的对外发送。
-   默认情况下，允许将心跳发送到 DM 风格的 `user:` 目标。设置 `agents.defaults.heartbeat.directPolicy: "block"` 以在保持心跳运行活跃的同时，抑制直接目标发送。
-   心跳运行完整的智能体轮次 — 较短的间隔会消耗更多令牌。

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## 媒体输入与输出

入站附件（图像/音频/文档）可以通过模板呈现给你的命令：

-   `{{MediaPath}}`（本地临时文件路径）
-   `{{MediaUrl}}`（伪 URL）
-   `{{Transcript}}`（如果启用了音频转录）

来自智能体的出站附件：在其单独一行中包含 `MEDIA:<路径或URL>`（无空格）。示例：

```
这是截图。
MEDIA:https://example.com/screenshot.png
```

OpenClaw 会提取这些内容并将其作为媒体与文本一起发送。

## 操作清单

```bash
openclaw status          # 本地状态（凭证、会话、排队事件）
openclaw status --all    # 完整诊断（只读，可粘贴）
openclaw status --deep   # 添加网关健康探测（Telegram + Discord）
openclaw health --json   # 网关健康快照（WS）
```

日志位于 `/tmp/openclaw/` 下（默认：`openclaw-YYYY-MM-DD.log`）。

## 后续步骤

-   WebChat：[WebChat](../web/webchat.md)
-   网关运维：[网关运行手册](../gateway.md)
-   Cron + 唤醒：[Cron 任务](../automation/cron-jobs.md)
-   macOS 菜单栏伴侣：[OpenClaw macOS 应用](../platforms/macos.md)
-   iOS 节点应用：[iOS 应用](../platforms/ios.md)
-   Android 节点应用：[Android 应用](../platforms/android.md)
-   Windows 状态：[Windows (WSL2)](../platforms/windows.md)
-   Linux 状态：[Linux 应用](../platforms/linux.md)
-   安全性：[安全性](../gateway/security.md)

[初始设置：macOS 应用](./onboarding.md)[CLI 参考](./wizard-cli-reference.md)