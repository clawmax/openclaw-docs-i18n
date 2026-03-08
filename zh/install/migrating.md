

  维护

  
# 迁移指南

本指南指导如何将 OpenClaw 网关从一台机器迁移到另一台机器，**无需重新进行初始配置**。从概念上讲，迁移很简单：

-   复制**状态目录**（`$OPENCLAW_STATE_DIR`，默认：`~/.openclaw/`）—— 这包括配置、认证、会话和频道状态。
-   复制你的**工作区**（默认 `~/.openclaw/workspace/`）—— 这包括你的智能体文件（记忆、提示词等）。

但需要注意关于**配置文件**、**权限**和**部分复制**的常见陷阱。

## 开始之前（你要迁移什么）

### 1) 识别你的状态目录

大多数安装使用默认路径：

-   **状态目录：** `~/.openclaw/`

但如果你使用了以下方式，路径可能不同：

-   `--profile `（通常会变成 `~/.openclaw-/`）
-   `OPENCLAW_STATE_DIR=/some/path`

如果不确定，请在**旧**机器上运行：

```bash
openclaw status
```

在输出中查找 `OPENCLAW_STATE_DIR` / 配置文件的提及。如果你运行多个网关，请为每个配置文件重复此操作。

### 2) 识别你的工作区

常见默认路径：

-   `~/.openclaw/workspace/`（推荐的工作区）
-   你创建的自定义文件夹

你的工作区是存放 `MEMORY.md`、`USER.md` 和 `memory/*.md` 等文件的地方。

### 3) 了解你将保留什么

如果你复制**状态目录和工作区两者**，你将保留：

-   网关配置 (`openclaw.json`)
-   认证配置文件 / API 密钥 / OAuth 令牌
-   会话历史记录 + 智能体状态
-   频道状态（例如 WhatsApp 登录/会话）
-   你的工作区文件（记忆、技能笔记等）

如果你**只**复制工作区（例如通过 Git），你将**不会**保留：

-   会话
-   凭据
-   频道登录信息

这些内容位于 `$OPENCLAW_STATE_DIR` 下。

## 迁移步骤（推荐）

### 步骤 0 — 创建备份（旧机器）

在**旧**机器上，首先停止网关，以便文件在复制过程中不会改变：

```bash
openclaw gateway stop
```

（可选但推荐）归档状态目录和工作区：

```bash
# 如果你使用配置文件或自定义位置，请调整路径
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

如果你有多个配置文件/状态目录（例如 `~/.openclaw-main`、`~/.openclaw-work`），请分别归档。

### 步骤 1 — 在新机器上安装 OpenClaw

在**新**机器上，安装 CLI（如果需要，也安装 Node）：

-   参见：[安装](../install.md)

在此阶段，即使初始配置创建了一个新的 `~/.openclaw/` 也没关系 —— 你将在下一步覆盖它。

### 步骤 2 — 将状态目录和工作区复制到新机器

复制**两者**：

-   `$OPENCLAW_STATE_DIR`（默认 `~/.openclaw/`）
-   你的工作区（默认 `~/.openclaw/workspace/`）

常用方法：

-   使用 `scp` 复制压缩包并解压
-   通过 SSH 使用 `rsync -a`
-   外部驱动器

复制后，确保：

-   包含了隐藏目录（例如 `.openclaw/`）
-   文件所有权对于运行网关的用户是正确的

### 步骤 3 — 运行 Doctor（迁移 + 服务修复）

在**新**机器上：

```bash
openclaw doctor
```

Doctor 是一个“安全无趣”的命令。它会修复服务、应用配置迁移并警告不匹配的情况。然后：

```bash
openclaw gateway restart
openclaw status
```

## 常见陷阱（以及如何避免）

### 陷阱：配置文件 / 状态目录不匹配

如果你在旧网关上使用了配置文件（或 `OPENCLAW_STATE_DIR`），而新网关使用了不同的配置，你可能会遇到以下症状：

-   配置更改不生效
-   频道丢失 / 已登出
-   空的会话历史记录

修复方法：使用你迁移的**相同**配置文件/状态目录运行网关/服务，然后重新运行：

```bash
openclaw doctor
```

### 陷阱：只复制 openclaw.json

`openclaw.json` 是不够的。许多提供商将状态存储在：

-   `$OPENCLAW_STATE_DIR/credentials/`
-   `$OPENCLAW_STATE_DIR/agents//...`

始终迁移整个 `$OPENCLAW_STATE_DIR` 文件夹。

### 陷阱：权限 / 所有权

如果你以 root 身份复制或更改了用户，网关可能无法读取凭据/会话。修复方法：确保状态目录 + 工作区由运行网关的用户拥有。

### 陷阱：在远程/本地模式之间迁移

-   如果你的 UI（WebUI/TUI）指向一个**远程**网关，则远程主机拥有会话存储 + 工作区。
-   迁移你的笔记本电脑不会移动远程网关的状态。

如果你处于远程模式，请迁移**网关主机**。

### 陷阱：备份中的密钥

`$OPENCLAW_STATE_DIR` 包含密钥（API 密钥、OAuth 令牌、WhatsApp 凭据）。请像对待生产密钥一样对待备份：

-   加密存储
-   避免通过不安全渠道共享
-   如果怀疑暴露，请轮换密钥

## 验证清单

在新机器上，确认：

-   `openclaw status` 显示网关正在运行
-   你的频道仍然保持连接（例如 WhatsApp 不需要重新配对）
-   仪表板可以打开并显示现有会话
-   你的工作区文件（记忆、配置）存在

## 相关链接

-   [Doctor](../gateway/doctor.md)
-   [网关故障排除](../gateway/troubleshooting.md)
-   [OpenClaw 将其数据存储在哪里？](../help/faq.md#where-does-openclaw-store-its-data)

[更新](./updating.md)[卸载](./uninstall.md)