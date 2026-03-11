

  首页

  
# OpenClaw（龙虾）官方文档中文版

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

> *"去角质！去角质！"* — 一只太空龙虾，大概吧

**OpenClaw（龙虾）** 是跨平台 AI 智能体（agent）网关，支持 WhatsApp、Telegram、Discord、iMessage 等更多平台。发一条消息，就能从口袋里获得智能体（agent）的回复。插件还支持 Mattermost 等更多平台。

本站是 OpenClaw 官方英文文档的**中文翻译版**，是面向中文用户的 **OpenClaw 龙虾中文文档**，帮助你更容易理解和上手 OpenClaw。

  
  
  

## OpenClaw（龙虾）是什么？

OpenClaw（龙虾）是一款**自托管网关（Gateway）**，将您常用的聊天应用——WhatsApp、Telegram、Discord、iMessage 等——连接到 Pi 等 AI 编程智能体（agent）。您只需在自己的机器（或服务器）上运行一个 Gateway 进程，它就能在您的消息应用和随时待命的 AI 助手之间架起桥梁。

**适合谁？** 希望随时随地通过消息使用个人 AI 助手的开发者和高级用户——无需交出数据控制权，也不依赖托管服务。

**有什么特别之处？**

- **自托管**：在您的硬件上运行，规则由您定
- **多频道（channel）**：一个 Gateway 同时支持 WhatsApp、Telegram、Discord 等平台
- **智能体（agent）原生**：专为编程智能体设计，支持工具调用、会话、记忆和多智能体路由
- **开源**：MIT 许可，社区驱动

**需要什么？** Node 22+、所选供应商的 API 密钥，以及 5 分钟时间。为获得最佳质量和安全性，请使用当前一代最强模型。

## 工作原理

网关（Gateway）是会话、路由和频道（channel）连接的唯一真实来源。

## 核心能力

  - **多频道网关** — 单个 Gateway 进程支持 WhatsApp、Telegram、Discord 和 iMessage。

  - **插件频道** — 通过扩展包添加 Mattermost 等更多平台。

  - **多智能体路由** — 按智能体（agent）、工作区或发送者隔离会话。

  - **媒体支持** — 发送和接收图片、音频和文档。

  - **Web 控制界面** — 浏览器仪表盘，用于聊天、配置、会话和节点管理。

  

## 快速开始

  ### 安装 OpenClaw

```bash
npm install -g openclaw@latest
```

  

  ### 运行引导安装服务

```bash
openclaw onboard --install-daemon
```

  

  ### 配对 WhatsApp 并启动网关

```bash
openclaw channels login
openclaw gateway --port 18789
```

  

需要完整的安装和开发环境设置？请参阅 [快速开始](./start/quickstart.md)。

## 仪表盘

网关（Gateway）启动后，打开浏览器控制界面。

- 本地默认地址：[http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- 远程访问：[Web 界面](./web.md) 和 [Tailscale](./gateway/tailscale.md)

  ![](../images/whatsapp-openclaw.jpg)

## 配置（可选）

配置文件位于 `~/.openclaw/openclaw.json`。

- 如果您**什么都不做**，OpenClaw 会使用内置的 Pi 二进制文件以 RPC 模式运行，并为每个发送者创建独立会话。
- 如果您想加强安全限制，可以从 `channels.whatsapp.allowFrom` 开始配置，对于群组则设置提及规则。

示例：

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": { "*": { "requireMention": true } }
    }
  },
  "messages": { "groupChat": { "mentionPatterns": ["@openclaw"] } }
}
```

## 从这里开始

  
  
  
  
  
  

## 了解更多

  
  
  
  
  

