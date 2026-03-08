

  技能

  
# ClawHub

ClawHub 是 **OpenClaw 的公共技能注册中心**。它是一项免费服务：所有技能都是公开、开放的，对所有人可见，以便共享和重用。一个技能只是一个包含 `SKILL.md` 文件（以及支持的文本文件）的文件夹。您可以在 Web 应用中浏览技能，或使用 CLI 搜索、安装、更新和发布技能。网站：[clawhub.ai](https://clawhub.ai)

## ClawHub 是什么

-   OpenClaw 技能的公共注册中心。
-   技能包和元数据的版本化存储。
-   用于搜索、标签和使用信号发现的发现界面。

## 工作原理

1.  用户发布一个技能包（文件 + 元数据）。
2.  ClawHub 存储该包，解析元数据，并分配一个版本。
3.  注册中心为技能建立索引以便搜索和发现。
4.  用户在 OpenClaw 中浏览、下载和安装技能。

## 您可以做什么

-   发布新技能和现有技能的新版本。
-   通过名称、标签或搜索发现技能。
-   下载技能包并检查其文件。
-   报告滥用或不安全的技能。
-   如果您是版主，可以隐藏、取消隐藏、删除或封禁。

## 适用人群（适合初学者）

如果您想为 OpenClaw 智能体添加新功能，ClawHub 是查找和安装技能的最简单方法。您无需了解后端工作原理。您可以：

-   通过自然语言搜索技能。
-   将技能安装到您的工作区。
-   稍后使用一个命令更新技能。
-   通过发布技能来备份您自己的技能。

## 快速开始（非技术性）

1.  安装 CLI（参见下一节）。
2.  搜索您需要的内容：
    -   `clawhub search "calendar"`
3.  安装一个技能：
    -   `clawhub install <skill-slug>`
4.  启动一个新的 OpenClaw 会话，以便它获取新技能。

## 安装 CLI

选择一种方式：

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## 如何与 OpenClaw 集成

默认情况下，CLI 将技能安装到当前工作目录下的 `./skills` 文件夹中。如果配置了 OpenClaw 工作区，`clawhub` 将回退到该工作区，除非您使用 `--workdir`（或 `CLAWHUB_WORKDIR`）覆盖。OpenClaw 从 `/skills` 加载工作区技能，并将在**下一个**会话中获取它们。如果您已经使用 `~/.openclaw/skills` 或捆绑技能，工作区技能将优先。有关技能如何加载、共享和管理的更多详细信息，请参阅 [技能](./skills.md)。

## 技能系统概述

技能是一个版本化的文件包，用于指导 OpenClaw 如何执行特定任务。每次发布都会创建一个新版本，注册中心会保留版本历史记录，以便用户审核更改。一个典型的技能包括：

-   一个包含主要描述和用法的 `SKILL.md` 文件。
-   技能使用的可选配置、脚本或支持文件。
-   元数据，如标签、摘要和安装要求。

ClawHub 使用元数据来支持发现并安全地暴露技能能力。注册中心还跟踪使用信号（例如星标和下载量）以改进排名和可见性。

## 服务提供的功能

-   **公开浏览** 技能及其 `SKILL.md` 内容。
-   **搜索** 由嵌入（向量搜索）驱动，而不仅仅是关键词。
-   **版本控制**，支持语义化版本、变更日志和标签（包括 `latest`）。
-   **下载** 每个版本的 zip 包。
-   **星标和评论** 用于社区反馈。
-   **审核** 钩子用于审批和审计。
-   **CLI 友好的 API** 用于自动化和脚本编写。

## 安全与审核

ClawHub 默认是开放的。任何人都可以上传技能，但发布技能的 GitHub 账户必须至少有一周的历史。这有助于减缓滥用行为，同时不阻碍合法贡献者。报告和审核：

-   任何登录用户都可以报告技能。
-   报告原因必须提供并会被记录。
-   每个用户最多可以同时有 20 个活跃报告。
-   拥有超过 3 个独立报告的技能默认会自动隐藏。
-   版主可以查看隐藏的技能、取消隐藏、删除它们或封禁用户。
-   滥用报告功能可能导致账户被封禁。

有兴趣成为版主吗？请在 OpenClaw Discord 中询问并联系版主或维护者。

## CLI 命令和参数

全局选项（适用于所有命令）：

-   `--workdir `: 工作目录（默认：当前目录；回退到 OpenClaw 工作区）。
-   `--dir `: 技能目录，相对于 workdir（默认：`skills`）。
-   `--site `: 站点基础 URL（浏览器登录）。
-   `--registry `: 注册中心 API 基础 URL。
-   `--no-input`: 禁用提示（非交互式）。
-   `-V, --cli-version`: 打印 CLI 版本。

认证：

-   `clawhub login`（浏览器流程）或 `clawhub login --token `
-   `clawhub logout`
-   `clawhub whoami`

选项：

-   `--token `: 粘贴 API 令牌。
-   `--label `: 为浏览器登录令牌存储的标签（默认：`CLI token`）。
-   `--no-browser`: 不打开浏览器（需要 `--token`）。

搜索：

-   `clawhub search "query"`
-   `--limit `: 最大结果数。

安装：

-   `clawhub install `
-   `--version `: 安装特定版本。
-   `--force`: 如果文件夹已存在则覆盖。

更新：

-   `clawhub update `
-   `clawhub update --all`
-   `--version `: 更新到特定版本（仅限单个 slug）。
-   `--force`: 当本地文件与任何已发布版本不匹配时覆盖。

列表：

-   `clawhub list`（读取 `.clawhub/lock.json`）

发布：

-   `clawhub publish `
-   `--slug `: 技能 slug。
-   `--name `: 显示名称。
-   `--version `: 语义化版本。
-   `--changelog `: 变更日志文本（可以为空）。
-   `--tags `: 逗号分隔的标签（默认：`latest`）。

删除/取消删除（仅限所有者/管理员）：

-   `clawhub delete  --yes`
-   `clawhub undelete  --yes`

同步（扫描本地技能 + 发布新的/更新的）：

-   `clawhub sync`
-   `--root <dir...>`: 额外的扫描根目录。
-   `--all`: 无需提示上传所有内容。
-   `--dry-run`: 显示将要上传的内容。
-   `--bump `: 更新的 `patch|minor|major`（默认：`patch`）。
-   `--changelog `: 非交互式更新的变更日志。
-   `--tags `: 逗号分隔的标签（默认：`latest`）。
-   `--concurrency `: 注册中心检查并发数（默认：4）。

## 智能体常用工作流

### 搜索技能

```bash
clawhub search "postgres backups"
```

### 下载新技能

```bash
clawhub install my-skill-pack
```

### 更新已安装的技能

```bash
clawhub update --all
```

### 备份您的技能（发布或同步）

对于单个技能文件夹：

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

要扫描并一次性备份多个技能：

```bash
clawhub sync --all
```

## 高级细节（技术性）

### 版本控制和标签

-   每次发布都会创建一个新的**语义化版本** `SkillVersion`。
-   标签（如 `latest`）指向一个版本；移动标签可以让您回滚。
-   变更日志附加到每个版本，在同步或发布更新时可以为空。

### 本地更改与注册中心版本

更新操作通过内容哈希将本地技能内容与注册中心版本进行比较。如果本地文件与任何已发布版本不匹配，CLI 会在覆盖前询问（或在非交互式运行中需要 `--force`）。

### 同步扫描和备用根目录

`clawhub sync` 首先扫描您当前的工作目录。如果未找到技能，它会回退到已知的旧位置（例如 `~/openclaw/skills` 和 `~/.openclaw/skills`）。这样设计是为了无需额外标志即可找到旧的技能安装。

### 存储和锁定文件

-   已安装的技能记录在您工作目录下的 `.clawhub/lock.json` 中。
-   认证令牌存储在 ClawHub CLI 配置文件中（可通过 `CLAWHUB_CONFIG_PATH` 覆盖）。

### 遥测（安装计数）

当您登录后运行 `clawhub sync` 时，CLI 会发送一个最小快照以计算安装计数。您可以完全禁用此功能：

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

## 环境变量

-   `CLAWHUB_SITE`: 覆盖站点 URL。
-   `CLAWHUB_REGISTRY`: 覆盖注册中心 API URL。
-   `CLAWHUB_CONFIG_PATH`: 覆盖 CLI 存储令牌/配置的位置。
-   `CLAWHUB_WORKDIR`: 覆盖默认工作目录。
-   `CLAWHUB_DISABLE_TELEMETRY=1`: 在 `sync` 时禁用遥测。

[技能配置](./skills-config.md)[插件](./plugin.md)

---