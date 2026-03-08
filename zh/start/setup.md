

  开发者设置

  
# 设置

> **ℹ️** 如果你是首次设置，请从 [入门指南](./getting-started.md) 开始。关于向导详情，请参阅 [入门向导](./wizard.md)。

 最后更新：2026-01-01

## 太长不看版

-   **定制内容位于仓库外：** `~/.openclaw/workspace` (工作区) + `~/.openclaw/openclaw.json` (配置)。
-   **稳定工作流：** 安装 macOS 应用；让它运行捆绑的 Gateway。
-   **前沿工作流：** 通过 `pnpm gateway:watch` 自行运行 Gateway，然后让 macOS 应用以本地模式连接。

## 前提条件 (从源码安装)

-   Node `>=22`
-   `pnpm`
-   Docker (可选；仅用于容器化设置/端到端测试 — 参见 [Docker](../install/docker.md))

## 定制策略 (避免更新破坏配置)

如果你想要“100% 为我定制”*并且*易于更新，请将你的自定义内容保存在：

-   **配置：** `~/.openclaw/openclaw.json` (JSON/类 JSON5 格式)
-   **工作区：** `~/.openclaw/workspace` (技能、提示词、记忆；可设为私有 git 仓库)

一次性引导设置：

```bash
openclaw setup
```

在此仓库内部，使用本地 CLI 入口：

```bash
openclaw setup
```

如果你尚未全局安装，可通过 `pnpm openclaw setup` 运行。

## 从此仓库运行 Gateway

在 `pnpm build` 之后，你可以直接运行打包好的 CLI：

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## 稳定工作流 (先安装 macOS 应用)

1.  安装并启动 **OpenClaw.app** (菜单栏应用)。
2.  完成入门/权限检查清单 (TCC 提示)。
3.  确保 Gateway 处于 **本地** 模式并正在运行 (应用会管理它)。
4.  连接通信渠道 (例如：WhatsApp)：

```bash
openclaw channels login
```

5.  完整性检查：

```bash
openclaw health
```

如果你的构建版本中没有入门向导：

-   运行 `openclaw setup`，然后 `openclaw channels login`，接着手动启动 Gateway (`openclaw gateway`)。

## 前沿工作流 (在终端中运行 Gateway)

目标：在 TypeScript Gateway 上开发，获得热重载，同时保持 macOS 应用 UI 连接。

### 0) (可选) 也从源码运行 macOS 应用

如果你也希望 macOS 应用处于前沿版本：

```
./scripts/restart-mac.sh
```

### 1) 启动开发版 Gateway

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` 以监视模式运行 gateway，并在 TypeScript 代码变更时重新加载。

### 2) 将 macOS 应用指向你正在运行的 Gateway

在 **OpenClaw.app** 中：

-   连接模式：**本地** 应用将连接到指定端口上正在运行的 gateway。

### 3) 验证

-   应用内的 Gateway 状态应显示 **“正在使用现有 gateway …”**
-   或通过 CLI：

```bash
openclaw health
```

### 常见陷阱

-   **端口错误：** Gateway WebSocket 默认使用 `ws://127.0.0.1:18789`；确保应用和 CLI 使用相同端口。
-   **状态存储位置：**
    -   凭证：`~/.openclaw/credentials/`
    -   会话：`~/.openclaw/agents//sessions/`
    -   日志：`/tmp/openclaw/`

## 凭证存储映射

在调试认证或决定备份内容时使用此映射：

-   **WhatsApp：** `~/.openclaw/credentials/whatsapp//creds.json`
-   **Telegram 机器人令牌：** 配置/环境变量 或 `channels.telegram.tokenFile`
-   **Discord 机器人令牌：** 配置/环境变量 或 SecretRef (环境变量/文件/执行提供者)
-   **Slack 令牌：** 配置/环境变量 (`channels.slack.*`)
-   **配对允许列表：**
    -   `~/.openclaw/credentials/-allowFrom.json` (默认账户)
    -   `~/.openclaw/credentials/--allowFrom.json` (非默认账户)
-   **模型认证配置文件：** `~/.openclaw/agents//agent/auth-profiles.json`
-   **文件存储的秘密载荷 (可选)：** `~/.openclaw/secrets.json`
-   **旧版 OAuth 导入：** `~/.openclaw/credentials/oauth.json` 更多详情：[安全](../gateway/security.md#credential-storage-map)。

## 更新 (不破坏你的设置)

-   将 `~/.openclaw/workspace` 和 `~/.openclaw/` 视为“你的东西”；不要将个人提示词/配置放入 `openclaw` 仓库。
-   更新源码：`git pull` + `pnpm install` (当 lockfile 变更时) + 继续使用 `pnpm gateway:watch`。

## Linux (systemd 用户服务)

Linux 安装使用 systemd **用户** 服务。默认情况下，systemd 会在用户登出/空闲时停止用户服务，这会终止 Gateway。入门向导会尝试为你启用 lingering (可能需要 sudo 密码)。如果它仍然关闭，请运行：

```bash
sudo loginctl enable-linger $USER
```

对于常开或多用户服务器，请考虑使用 **系统** 服务而非用户服务 (无需 lingering)。参见 [Gateway 运行手册](../gateway.md) 中的 systemd 说明。

## 相关文档

-   [Gateway 运行手册](../gateway.md) (标志、监控、端口)
-   [Gateway 配置](../gateway/configuration.md) (配置模式与示例)
-   [Discord](../channels/discord.md) 和 [Telegram](../channels/telegram.md) (回复标签与 replyToMode 设置)
-   [OpenClaw 助手设置](./openclaw.md)
-   [macOS 应用](../platforms/macos.md) (gateway 生命周期)

[会话管理深度解析](../reference/session-management-compaction.md)[Pi 开发工作流](../pi-dev.md)