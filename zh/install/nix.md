

  其他安装方式

  
# Nix

推荐通过 **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** 使用 Nix 运行 OpenClaw——这是一个开箱即用的 Home Manager 模块。

## 快速开始

将此粘贴到你的 AI 助手（Claude、Cursor 等）中：

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 完整指南：[github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** nix-openclaw 仓库是 Nix 安装的权威信息来源。本页只是一个快速概览。

## 你将获得

-   Gateway + macOS 应用 + 工具（whisper、spotify、cameras）—— 全部固定版本
-   可抵御重启的 Launchd 服务
-   具有声明式配置的插件系统
-   即时回滚：`home-manager switch --rollback`

* * *

## Nix 模式运行时行为

当设置 `OPENCLAW_NIX_MODE=1` 时（nix-openclaw 会自动设置）：OpenClaw 支持一种 **Nix 模式**，该模式使配置具有确定性并禁用自动安装流程。通过导出以下变量启用：

```
OPENCLAW_NIX_MODE=1
```

在 macOS 上，GUI 应用不会自动继承 shell 环境变量。你也可以通过 defaults 命令启用 Nix 模式：

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### 配置 + 状态路径

OpenClaw 从 `OPENCLAW_CONFIG_PATH` 读取 JSON5 配置，并将可变数据存储在 `OPENCLAW_STATE_DIR` 中。需要时，你也可以设置 `OPENCLAW_HOME` 来控制用于内部路径解析的基础主目录。

-   `OPENCLAW_HOME`（默认优先级：`HOME` / `USERPROFILE` / `os.homedir()`）
-   `OPENCLAW_STATE_DIR`（默认：`~/.openclaw`）
-   `OPENCLAW_CONFIG_PATH`（默认：`$OPENCLAW_STATE_DIR/openclaw.json`）

在 Nix 下运行时，请将这些变量显式设置为 Nix 管理的位置，以便运行时状态和配置保持在不可变的存储之外。

### Nix 模式下的运行时行为

-   自动安装和自我变异流程被禁用
-   缺少依赖项时会显示 Nix 特定的修复消息
-   存在时，UI 会显示一个只读的 Nix 模式横幅

## 打包说明（macOS）

macOS 打包流程期望在以下位置有一个稳定的 Info.plist 模板：

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 将此模板复制到应用包中并修补动态字段（bundle ID、版本/构建号、Git SHA、Sparkle 密钥）。这使得 plist 对于 SwiftPM 打包和 Nix 构建（不依赖完整的 Xcode 工具链）具有确定性。

## 相关链接

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — 完整设置指南
-   [向导](../start/wizard.md) — 非 Nix CLI 设置
-   [Docker](./docker.md) — 容器化设置

[Podman](./podman.md)[Ansible](./ansible.md)