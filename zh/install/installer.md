

  安装概述

  
# 安装器内部原理

OpenClaw 提供了三个安装脚本，由 `openclaw.ai` 提供。

| 脚本 | 平台 | 功能 |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | 如果需要则安装 Node，通过 npm（默认）或 git 安装 OpenClaw，并可运行初始化引导。 |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | 将 Node + OpenClaw 安装到本地前缀目录（`~/.openclaw`）。无需 root 权限。 |
| [`install.ps1`](#installps1) | Windows (PowerShell) | 如果需要则安装 Node，通过 npm（默认）或 git 安装 OpenClaw，并可运行初始化引导。 |

## 快速命令

 

> **ℹ️** 如果安装成功但在新终端中找不到 `openclaw` 命令，请参阅 [Node.js 故障排除](./node.md#troubleshooting)。

* * *

## install.sh

> **💡** 推荐用于 macOS/Linux/WSL 上的大多数交互式安装。

### 流程 (install.sh)

### 步骤 1：检测操作系统

支持 macOS 和 Linux（包括 WSL）。如果检测到 macOS，会在缺失时安装 Homebrew。

### 步骤 2：确保 Node.js 22+

检查 Node 版本，并在需要时安装 Node 22（在 macOS 上使用 Homebrew，在 Linux 上使用 apt/dnf/yum 的 NodeSource 安装脚本）。

### 步骤 3：确保 Git 已安装

如果缺失则安装 Git。

### 步骤 4：安装 OpenClaw

-   `npm` 方法（默认）：全局 npm 安装
-   `git` 方法：克隆/更新仓库，使用 pnpm 安装依赖，构建，然后在 `~/.local/bin/openclaw` 安装包装器

### 步骤 5：安装后任务

-   在升级和 git 安装时运行 `openclaw doctor --non-interactive`（尽力而为）
-   在适当时尝试运行初始化引导（TTY 可用、未禁用引导且引导/配置检查通过）
-   默认设置 `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### 源码检出检测

如果在 OpenClaw 检出目录内运行（存在 `package.json` + `pnpm-workspace.yaml`），脚本会提供选项：

-   使用检出目录（`git`），或
-   使用全局安装（`npm`）

如果没有可用的 TTY 且未设置安装方法，则默认使用 `npm` 并发出警告。脚本会以退出码 `2` 退出，表示选择了无效的安装方法或提供了无效的 `--install-method` 值。

### 示例 (install.sh)

 

| 标志 | 描述 |
| --- | --- |
| `--install-method npm\|git` | 选择安装方法（默认：`npm`）。别名：`--method` |
| `--npm` | npm 方法的快捷方式 |
| `--git` | git 方法的快捷方式。别名：`--github` |
| `--version <version\|dist-tag>` | npm 版本或分发标签（默认：`latest`） |
| `--beta` | 如果可用则使用 beta 分发标签，否则回退到 `latest` |
| `--git-dir ` | 检出目录（默认：`~/openclaw`）。别名：`--dir` |
| `--no-git-update` | 跳过现有检出目录的 `git pull` |
| `--no-prompt` | 禁用提示 |
| `--no-onboard` | 跳过初始化引导 |
| `--onboard` | 启用初始化引导 |
| `--dry-run` | 仅打印操作而不实际应用更改 |
| `--verbose` | 启用调试输出（`set -x`，npm notice 级别日志） |
| `--help` | 显示用法（`-h`） |

| 变量 | 描述 |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | 安装方法 |
| `OPENCLAW_VERSION=latest\|next\|` | npm 版本或分发标签 |
| `OPENCLAW_BETA=0\|1` | 如果可用则使用 beta 版本 |
| `OPENCLAW_GIT_DIR=` | 检出目录 |
| `OPENCLAW_GIT_UPDATE=0\|1` | 切换 git 更新 |
| `OPENCLAW_NO_PROMPT=1` | 禁用提示 |
| `OPENCLAW_NO_ONBOARD=1` | 跳过初始化引导 |
| `OPENCLAW_DRY_RUN=1` | 干运行模式 |
| `OPENCLAW_VERBOSE=1` | 调试模式 |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm 日志级别 |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | 控制 sharp/libvips 行为（默认：`1`） |

* * *

## install-cli.sh

> **ℹ️** 专为希望将所有内容安装在本地前缀目录（默认 `~/.openclaw`）且不依赖系统 Node 的环境设计。

### 流程 (install-cli.sh)

### 步骤 1：安装本地 Node 运行时

下载 Node 压缩包（默认 `22.22.0`）到 `/tools/node-v` 并验证 SHA-256。

### 步骤 2：确保 Git 已安装

如果 Git 缺失，尝试在 Linux 上通过 apt/dnf/yum 或在 macOS 上通过 Homebrew 安装。

### 步骤 3：在前缀目录下安装 OpenClaw

使用 `--prefix ` 通过 npm 安装，然后将包装器写入 `/bin/openclaw`。

### 示例 (install-cli.sh)

 

| 标志 | 描述 |
| --- | --- |
| `--prefix ` | 安装前缀目录（默认：`~/.openclaw`） |
| `--version ` | OpenClaw 版本或分发标签（默认：`latest`） |
| `--node-version ` | Node 版本（默认：`22.22.0`） |
| `--json` | 输出 NDJSON 事件 |
| `--onboard` | 安装后运行 `openclaw onboard` |
| `--no-onboard` | 跳过初始化引导（默认） |
| `--set-npm-prefix` | 在 Linux 上，如果当前 npm 前缀目录不可写，则强制将其设为 `~/.npm-global` |
| `--help` | 显示用法（`-h`） |

| 变量 | 描述 |
| --- | --- |
| `OPENCLAW_PREFIX=` | 安装前缀目录 |
| `OPENCLAW_VERSION=` | OpenClaw 版本或分发标签 |
| `OPENCLAW_NODE_VERSION=` | Node 版本 |
| `OPENCLAW_NO_ONBOARD=1` | 跳过初始化引导 |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm 日志级别 |
| `OPENCLAW_GIT_DIR=` | 遗留清理查找路径（用于移除旧的 `Peekaboo` 子模块检出） |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | 控制 sharp/libvips 行为（默认：`1`） |

* * *

## install.ps1

### 流程 (install.ps1)

### 步骤 1：确保 PowerShell + Windows 环境

需要 PowerShell 5+。

### 步骤 2：确保 Node.js 22+

如果缺失，尝试通过 winget、然后 Chocolatey、最后 Scoop 安装。

### 步骤 3：安装 OpenClaw

-   `npm` 方法（默认）：使用指定的 `-Tag` 进行全局 npm 安装
-   `git` 方法：克隆/更新仓库，使用 pnpm 安装/构建，并在 `%USERPROFILE%\.local\bin\openclaw.cmd` 安装包装器

### 步骤 4：安装后任务

在可能的情况下将所需的 bin 目录添加到用户 PATH，然后在升级和 git 安装时运行 `openclaw doctor --non-interactive`（尽力而为）。

### 示例 (install.ps1)

 

| 标志 | 描述 |
| --- | --- |
| `-InstallMethod npm\|git` | 安装方法（默认：`npm`） |
| `-Tag ` | npm 分发标签（默认：`latest`） |
| `-GitDir ` | 检出目录（默认：`%USERPROFILE%\openclaw`） |
| `-NoOnboard` | 跳过初始化引导 |
| `-NoGitUpdate` | 跳过 `git pull` |
| `-DryRun` | 仅打印操作 |

| 变量 | 描述 |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | 安装方法 |
| `OPENCLAW_GIT_DIR=` | 检出目录 |
| `OPENCLAW_NO_ONBOARD=1` | 跳过初始化引导 |
| `OPENCLAW_GIT_UPDATE=0` | 禁用 git pull |
| `OPENCLAW_DRY_RUN=1` | 干运行模式 |

 

> **ℹ️** 如果使用 `-InstallMethod git` 但 Git 缺失，脚本将退出并打印 Git for Windows 的链接。

* * *

## CI 与自动化

使用非交互式标志/环境变量以实现可预测的运行。

* * *

## 故障排除

`git` 安装方法需要 Git。对于 `npm` 安装，仍然会检查/安装 Git，以避免依赖项使用 git URL 时出现 `spawn git ENOENT` 失败。

某些 Linux 设置将 npm 全局前缀指向 root 拥有的路径。`install.sh` 可以将前缀切换到 `~/.npm-global`，并将 PATH 导出追加到 shell rc 文件（当这些文件存在时）。

脚本默认设置 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 以避免 sharp 针对系统 libvips 构建。要覆盖此设置：

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

安装 Git for Windows，重新打开 PowerShell，重新运行安装程序。

运行 `npm config get prefix` 并将该目录添加到你的用户 PATH 中（Windows 上不需要 `\bin` 后缀），然后重新打开 PowerShell。

`install.ps1` 目前未公开 `-Verbose` 开关。使用 PowerShell 跟踪进行脚本级诊断：

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

通常是 PATH 问题。请参阅 [Node.js 故障排除](./node.md#troubleshooting)。

[安装](../install.md)[Docker](./docker.md)