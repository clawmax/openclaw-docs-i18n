

  托管与部署

  
# 部署到 Northflank

通过一键模板在 Northflank 上部署 OpenClaw，并在浏览器中完成设置。这是最简单的“服务器上无需终端”的路径：Northflank 为您运行网关，您通过 `/setup` 网页向导配置一切。

## 如何开始

1.  点击 [部署 OpenClaw](https://northflank.com/stacks/deploy-openclaw) 打开模板。
2.  如果您还没有账户，请先 [在 Northflank 上注册](https://app.northflank.com/signup)。
3.  点击 **立即部署 OpenClaw**。
4.  设置必需的环境变量：`SETUP_PASSWORD`。
5.  点击 **部署堆栈** 以构建并运行 OpenClaw 模板。
6.  等待部署完成，然后点击 **查看资源**。
7.  打开 OpenClaw 服务。
8.  打开公共的 OpenClaw URL 并在 `/setup` 完成设置。
9.  在 `/openclaw` 打开控制界面。

## 您将获得

-   托管的 OpenClaw 网关 + 控制界面
-   位于 `/setup` 的网页设置向导（无需终端命令）
-   通过 Northflank 卷 (`/data`) 实现的持久化存储，确保配置/凭据/工作空间在重新部署后得以保留

## 设置流程

1.  访问 `https://<您的-northflank-域名>/setup` 并输入您的 `SETUP_PASSWORD`。
2.  选择一个模型/认证提供商并粘贴您的密钥。
3.  （可选）添加 Telegram/Discord/Slack 令牌。
4.  点击 **运行设置**。
5.  在 `https://<您的-northflank-域名>/openclaw` 打开控制界面。

如果 Telegram 私聊设置为配对模式，设置向导可以批准配对码。

## 获取聊天令牌

### Telegram 机器人令牌

1.  在 Telegram 中联系 `@BotFather`
2.  运行 `/newbot`
3.  复制令牌（格式类似 `123456789:AA...`）
4.  将其粘贴到 `/setup` 中

### Discord 机器人令牌

1.  前往 [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **新建应用程序** → 选择一个名称
3.  **机器人** → **添加机器人**
4.  在 机器人 → 特权网关意图 下 **启用 MESSAGE CONTENT INTENT**（必需，否则机器人启动时会崩溃）
5.  复制 **机器人令牌** 并粘贴到 `/setup` 中
6.  将机器人邀请到您的服务器（OAuth2 URL 生成器；权限范围：`bot`, `applications.commands`）

[部署到 Render](./render.md)[开发频道](./development-channels.md)

---