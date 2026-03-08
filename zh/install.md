

  安装概述

  
# 安装

已经按照[快速入门](./start/getting-started.md)操作过了吗？那您已经准备就绪了——本页面提供替代安装方法、特定平台说明和维护信息。

## 系统要求

-   **[Node 22+](./install/node.md)**（如果缺失，[安装脚本](#install-methods)会安装它）
-   macOS、Linux 或 Windows
-   仅当从源码构建时需要 `pnpm`

> **ℹ️** 在 Windows 上，我们强烈建议在 [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) 下运行 OpenClaw。

## 安装方法

> **💡** **安装脚本**是推荐的 OpenClaw 安装方式。它在一个步骤中处理 Node 检测、安装和初始化引导。

 

> **⚠️** 对于 VPS/云主机，请尽可能避免使用第三方“一键式”市场镜像。优先选择干净的基础操作系统镜像（例如 Ubuntu LTS），然后使用安装脚本自行安装 OpenClaw。

 

下载 CLI，通过 npm 全局安装，并启动初始化引导向导。

就这样——脚本会处理 Node 检测、安装和引导。要跳过引导，仅安装二进制文件：

有关所有标志、环境变量和 CI/自动化选项，请参阅[安装器内部机制](./install/installer.md)。

如果您已安装 Node 22+ 并希望自行管理安装：

适用于贡献者或希望从本地检出代码运行的用户。

1

[

](#)

克隆并构建

克隆 [OpenClaw 仓库](https://github.com/openclaw/openclaw) 并进行构建：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
```

2

[

](#)

链接 CLI

使 `openclaw` 命令全局可用：

```bash
pnpm link --global
```

或者，跳过链接，在仓库内部通过 `pnpm openclaw ...` 运行命令。

3

[

](#)

运行初始化引导

```bash
openclaw onboard --install-daemon
```

有关更深入的开发工作流程，请参阅[开发环境设置](./start/setup.md)。

## 其他安装方法

## 安装后

验证一切是否正常工作：

```bash
openclaw doctor         # 检查配置问题
openclaw status         # 网关状态
openclaw dashboard      # 打开浏览器 UI
```

如果需要自定义运行时路径，请使用：

-   `OPENCLAW_HOME` 用于基于主目录的内部路径
-   `OPENCLAW_STATE_DIR` 用于可变状态文件位置
-   `OPENCLAW_CONFIG_PATH` 用于配置文件位置

有关优先级和完整详细信息，请参阅[环境变量](./help/environment.md)。

## 故障排除：找不到 openclaw 命令

快速诊断：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

如果 `$(npm prefix -g)/bin` (macOS/Linux) 或 `$(npm prefix -g)` (Windows) **不在**您的 `$PATH` 中，您的 shell 将无法找到全局 npm 二进制文件（包括 `openclaw`）。修复——将其添加到您的 shell 启动文件（`~/.zshrc` 或 `~/.bashrc`）：

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

在 Windows 上，将 `npm prefix -g` 的输出添加到您的 PATH 中。然后打开一个新的终端（或在 zsh 中执行 `rehash` / 在 bash 中执行 `hash -r`）。

## 更新 / 卸载

[安装器内部机制](./install/installer.md)