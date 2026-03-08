

  基础概念

  
# 代理工作区

工作区是代理的家。它是用于文件工具和工作区上下文的唯一工作目录。请将其视为私有并当作记忆来处理。这与存储配置、凭据和会话的 `~/.openclaw/` 是分开的。**重要提示：** 工作区是**默认的当前工作目录**，并非一个严格的沙箱。工具会相对于工作区解析相对路径，但绝对路径仍然可以访问主机上的其他地方，除非启用了沙箱隔离。如果需要隔离，请使用 [`agents.defaults.sandbox`](../gateway/sandboxing.md)（和/或每个代理的沙箱配置）。当启用沙箱隔离且 `workspaceAccess` 不是 `"rw"` 时，工具将在 `~/.openclaw/sandboxes` 下的沙箱工作区内运行，而不是您的主机工作区。

## 默认位置

-   默认：`~/.openclaw/workspace`
-   如果设置了 `OPENCLAW_PROFILE` 且不为 `"default"`，则默认位置变为 `~/.openclaw/workspace-`。
-   在 `~/.openclaw/openclaw.json` 中覆盖：

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`、`openclaw configure` 或 `openclaw setup` 将创建工作区并填充引导文件（如果它们缺失）。沙箱种子复制仅接受工作区内的常规文件；解析到源工作区之外的符号链接/硬链接别名将被忽略。如果您已自行管理工作区文件，可以禁用引导文件创建：

```json
{ agent: { skipBootstrap: true } }
```

## 额外的工作区文件夹

旧版本的安装可能创建了 `~/openclaw`。保留多个工作区目录可能会导致令人困惑的认证或状态漂移，因为一次只有一个工作区处于活动状态。**建议：** 保留一个活动工作区。如果您不再使用额外的文件夹，请将其归档或移至回收站（例如 `trash ~/openclaw`）。如果您有意保留多个工作区，请确保 `agents.defaults.workspace` 指向活动的工作区。`openclaw doctor` 在检测到额外的工作区目录时会发出警告。

## 工作区文件说明（每个文件的含义）

这些是 OpenClaw 期望在工作区内存在的标准文件：

-   `AGENTS.md`
    -   代理的操作指南以及它应如何使用记忆。
    -   在每次会话开始时加载。
    -   适合放置规则、优先级和“行为准则”细节。
-   `SOUL.md`
    -   角色设定、语气和边界。
    -   每次会话加载。
-   `USER.md`
    -   用户是谁以及如何称呼他们。
    -   每次会话加载。
-   `IDENTITY.md`
    -   代理的名称、风格和表情符号。
    -   在引导仪式期间创建/更新。
-   `TOOLS.md`
    -   关于您本地工具和约定的笔记。
    -   不控制工具的可用性；仅为指导性内容。
-   `HEARTBEAT.md`
    -   可选的心跳运行小清单。
    -   保持简短以避免令牌消耗。
-   `BOOT.md`
    -   当内部钩子启用时，在网关重启时执行的可选启动清单。
    -   保持简短；使用消息工具进行外发发送。
-   `BOOTSTRAP.md`
    -   一次性首次运行仪式。
    -   仅为一个全新的工作区创建。
    -   仪式完成后删除它。
-   `memory/YYYY-MM-DD.md`
    -   每日记忆日志（每天一个文件）。
    -   建议在会话开始时读取今天和昨天的日志。
-   `MEMORY.md`（可选）
    -   精选的长期记忆。
    -   仅在主要的私有会话中加载（不在共享/群组上下文中）。

有关工作流程和自动记忆清理，请参阅[记忆](./memory.md)。

-   `skills/`（可选）
    -   工作区特定的技能。
    -   当名称冲突时，覆盖托管/捆绑的技能。
-   `canvas/`（可选）
    -   用于节点显示的 Canvas UI 文件（例如 `canvas/index.html`）。

如果任何引导文件缺失，OpenClaw 会向会话中注入一个“文件缺失”标记并继续。大型引导文件在注入时会被截断；可以通过 `agents.defaults.bootstrapMaxChars`（默认值：20000）和 `agents.defaults.bootstrapTotalMaxChars`（默认值：150000）调整限制。`openclaw setup` 可以重新创建缺失的默认文件，而不会覆盖现有文件。

## 什么不在工作区中

这些文件位于 `~/.openclaw/` 下，**不应**提交到工作区仓库：

-   `~/.openclaw/openclaw.json`（配置）
-   `~/.openclaw/credentials/`（OAuth 令牌、API 密钥）
-   `~/.openclaw/agents//sessions/`（会话记录 + 元数据）
-   `~/.openclaw/skills/`（托管技能）

如果您需要迁移会话或配置，请单独复制它们，并将其排除在版本控制之外。

## Git 备份（推荐，私有）

将工作区视为私有记忆。将其放入一个**私有**的 git 仓库中，以便备份和恢复。在运行网关的机器上执行这些步骤（即工作区所在的机器）。

### 1) 初始化仓库

如果已安装 git，全新的工作区会自动初始化。如果此工作区还不是一个仓库，请运行：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) 添加私有远程仓库（适合初学者的选项）

选项 A：GitHub 网页界面

1.  在 GitHub 上创建一个新的**私有**仓库。
2.  不要使用 README 初始化（避免合并冲突）。
3.  复制 HTTPS 远程 URL。
4.  添加远程仓库并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

选项 B：GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

选项 C：GitLab 网页界面

1.  在 GitLab 上创建一个新的**私有**仓库。
2.  不要使用 README 初始化（避免合并冲突）。
3.  复制 HTTPS 远程 URL。
4.  添加远程仓库并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) 持续更新

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## 不要提交机密信息

即使在私有仓库中，也应避免在工作区中存储机密信息：

-   API 密钥、OAuth 令牌、密码或私有凭据。
-   `~/.openclaw/` 下的任何内容。
-   聊天记录或敏感附件的原始转储。

如果必须存储敏感引用，请使用占位符，并将真实的机密信息保存在其他地方（密码管理器、环境变量或 `~/.openclaw/`）。建议的 `.gitignore` 起始内容：

```
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## 将工作区迁移到新机器

1.  将仓库克隆到所需路径（默认为 `~/.openclaw/workspace`）。
2.  在 `~/.openclaw/openclaw.json` 中将 `agents.defaults.workspace` 设置为该路径。
3.  运行 `openclaw setup --workspace ` 以填充任何缺失的文件。
4.  如果需要会话，请单独从旧机器复制 `~/.openclaw/agents//sessions/`。

## 高级说明

-   多代理路由可以为每个代理使用不同的工作区。有关路由配置，请参阅[通道路由](../channels/channel-routing.md)。
-   如果启用了 `agents.defaults.sandbox`，非主会话可以使用 `agents.defaults.sandbox.workspaceRoot` 下的每个会话沙箱工作区。

[上下文](./context.md)[OAuth](./oauth.md)