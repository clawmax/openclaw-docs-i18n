

  模板

  
# 默认 AGENTS.md

## 首次运行（推荐）

OpenClaw 为代理使用一个专用工作区目录。默认路径：`~/.openclaw/workspace`（可通过 `agents.defaults.workspace` 配置）。

1.  创建工作区（如果尚不存在）：

```bash
mkdir -p ~/.openclaw/workspace
```

2.  将默认工作区模板复制到工作区：

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3.  可选：如果你想要个人助理技能名册，请用此文件替换 AGENTS.md：

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4.  可选：通过设置 `agents.defaults.workspace` 选择不同的工作区（支持 `~`）：

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## 安全默认值

-   不要将目录或机密信息转储到聊天中。
-   除非明确要求，否则不要运行破坏性命令。
-   不要向外部消息界面发送部分/流式回复（仅发送最终回复）。

## 会话启动（必需）

-   读取 `SOUL.md`、`USER.md`、`memory.md` 以及 `memory/` 目录下的今天和昨天的文件。
-   在回复之前完成此操作。

## 灵魂（必需）

-   `SOUL.md` 定义了身份、语气和边界。保持其最新状态。
-   如果你更改了 `SOUL.md`，请告知用户。
-   每个会话你都是一个全新的实例；连续性存在于这些文件中。

## 共享空间（推荐）

-   你不是用户的声音；在群聊或公共频道中要小心。
-   不要分享私人数据、联系信息或内部笔记。

## 内存系统（推荐）

-   每日日志：`memory/YYYY-MM-DD.md`（如果需要，创建 `memory/` 目录）。
-   长期记忆：`memory.md` 用于存储持久的事实、偏好和决策。
-   在会话启动时，如果存在，读取今天、昨天以及 `memory.md` 的内容。
-   捕获内容：决策、偏好、约束、未完成事项。
-   除非明确要求，否则避免记录机密信息。

## 工具与技能

-   工具存在于技能中；当你需要时，遵循每个技能的 `SKILL.md`。
-   将环境特定的笔记保存在 `TOOLS.md` 中（技能笔记）。

## 备份提示（推荐）

如果你将此工作区视为 Clawd 的“记忆”，请将其设为 git 仓库（最好是私有的），以便备份 `AGENTS.md` 和你的记忆文件。

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# 可选：添加一个私有远程仓库并推送
```

## OpenClaw 的功能

-   运行 WhatsApp 网关和 Pi 编码代理，使助手能够通过主机 Mac 读写聊天、获取上下文并运行技能。
-   macOS 应用程序管理权限（屏幕录制、通知、麦克风）并通过其捆绑的二进制文件公开 `openclaw` CLI。
-   默认情况下，直接聊天会合并到代理的 `main` 会话中；群组保持隔离，格式为 `agent:::group:`（房间/频道：`agent:::channel:`）；心跳保持后台任务存活。

## 核心技能（在 设置 → 技能 中启用）

-   **mcporter** — 用于管理外部技能后端的工具服务器运行时/CLI。
-   **Peekaboo** — 快速的 macOS 屏幕截图，可选 AI 视觉分析。
-   **camsnap** — 从 RTSP/ONVIF 安全摄像头捕获帧、剪辑或运动警报。
-   **oracle** — 支持 OpenAI 的代理 CLI，具有会话回放和浏览器控制功能。
-   **eightctl** — 从终端控制你的睡眠。
-   **imsg** — 发送、读取、流式传输 iMessage 和 SMS。
-   **wacli** — WhatsApp CLI：同步、搜索、发送。
-   **discord** — Discord 操作：反应、贴纸、投票。使用 `user:` 或 `channel:` 目标（裸数字 ID 是模糊的）。
-   **gog** — Google 套件 CLI：Gmail、日历、云端硬盘、联系人。
-   **spotify-player** — 终端 Spotify 客户端，用于搜索/排队/控制播放。
-   **sag** — 具有 mac 风格 say UX 的 ElevenLabs 语音；默认流式传输到扬声器。
-   **Sonos CLI** — 从脚本控制 Sonos 扬声器（发现/状态/播放/音量/分组）。
-   **blucli** — 从脚本播放、分组和自动化 BluOS 播放器。
-   **OpenHue CLI** — 用于场景和自动化的 Philips Hue 灯光控制。
-   **OpenAI Whisper** — 本地语音转文本，用于快速听写和语音邮件转录。
-   **Gemini CLI** — 从终端使用 Google Gemini 模型进行快速问答。
-   **agent-tools** — 用于自动化和辅助脚本的实用工具包。

## 使用说明

-   脚本编写优先使用 `openclaw` CLI；mac 应用程序处理权限。
-   从“技能”选项卡运行安装；如果二进制文件已存在，它会隐藏按钮。
-   保持心跳启用，以便助手可以安排提醒、监控收件箱并触发摄像头捕获。
-   Canvas UI 以全屏模式运行，带有原生覆盖层。避免将关键控件放置在左上角/右上角/底部边缘；在布局中添加明确的边距，不要依赖安全区域插入。
-   对于浏览器驱动的验证，使用 `openclaw browser`（标签/状态/截图）以及 OpenClaw 管理的 Chrome 配置文件。
-   对于 DOM 检查，使用 `openclaw browser eval|query|dom|snapshot`（当你需要机器输出时，使用 `--json`/`--out`）。
-   对于交互，使用 `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run`（点击/输入需要快照引用；使用 `evaluate` 处理 CSS 选择器）。

[设备模型数据库](./device-models.md)[AGENTS.md 模板](./templates/AGENTS.md)

---