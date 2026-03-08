

  托管与部署

  
# 部署到 Railway

通过一键模板在 Railway 上部署 OpenClaw，并在浏览器中完成设置。这是最简单的“服务器上无需终端”的路径：Railway 为您运行网关，您通过 `/setup` 网页向导配置一切。

## 快速清单（新用户）

1.  点击下方的 **Deploy on Railway**。
2.  添加一个挂载在 `/data` 的 **Volume**。
3.  设置必需的 **Variables**（至少 `SETUP_PASSWORD`）。
4.  在端口 `8080` 上启用 **HTTP Proxy**。
5.  打开 `https://<your-railway-domain>/setup` 并完成向导。

## 一键部署

[Deploy on Railway](https://railway.com/deploy/clawdbot-railway-template) 部署后，在 **Railway → 你的服务 → Settings → Domains** 中找到你的公共 URL。Railway 会提供：

-   一个生成的域名（通常是 `https://.up.railway.app`），或者
-   如果你附加了自定义域名，则使用你的自定义域名。

然后打开：

-   `https://<your-railway-domain>/setup` — 设置向导（受密码保护）
-   `https://<your-railway-domain>/openclaw` — 控制界面

## 你将获得

-   托管的 OpenClaw 网关 + 控制界面
-   位于 `/setup` 的网页设置向导（无需终端命令）
-   通过 Railway Volume (`/data`) 实现持久化存储，使配置/凭证/工作空间在重新部署后得以保留
-   可在 `/setup/export` 进行备份导出，以便日后迁移出 Railway

## 必需的 Railway 设置

### 公共网络

为服务启用 **HTTP Proxy**。

-   端口：`8080`

### Volume（必需）

附加一个挂载在以下路径的卷：

-   `/data`

### 变量

在服务上设置这些变量：

-   `SETUP_PASSWORD`（必需）
-   `PORT=8080`（必需 — 必须与公共网络中的端口匹配）
-   `OPENCLAW_STATE_DIR=/data/.openclaw`（推荐）
-   `OPENCLAW_WORKSPACE_DIR=/data/workspace`（推荐）
-   `OPENCLAW_GATEWAY_TOKEN`（推荐；视为管理员密钥）

## 设置流程

1.  访问 `https://<your-railway-domain>/setup` 并输入你的 `SETUP_PASSWORD`。
2.  选择一个模型/认证提供商并粘贴你的密钥。
3.  （可选）添加 Telegram/Discord/Slack 令牌。
4.  点击 **Run setup**。

如果 Telegram 私聊设置为配对模式，设置向导可以批准配对码。

## 获取聊天令牌

### Telegram 机器人令牌

1.  在 Telegram 中私聊 `@BotFather`
2.  运行 `/newbot`
3.  复制令牌（格式类似 `123456789:AA...`）
4.  将其粘贴到 `/setup` 中

### Discord 机器人令牌

1.  前往 [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **New Application** → 选择一个名称
3.  **Bot** → **Add Bot**
4.  在 Bot → Privileged Gateway Intents 下 **Enable MESSAGE CONTENT INTENT**（必需，否则机器人启动时会崩溃）
5.  复制 **Bot Token** 并粘贴到 `/setup` 中
6.  邀请机器人到你的服务器（OAuth2 URL Generator；权限范围：`bot`, `applications.commands`）

## 备份与迁移

在以下地址下载备份：

-   `https://<your-railway-domain>/setup/export`

这将导出你的 OpenClaw 状态 + 工作空间，以便你可以迁移到其他主机而不会丢失配置或记忆。

[exe.dev](./exe-dev.md)[部署到 Render](./render.md)