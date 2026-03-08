
  首页

  
# OpenClaw

  **面向 WhatsApp、Telegram、Discord、iMessage 等全平台 AI 代理网关。**
  

  发消息即可获得代理回复。插件支持 Mattermost 等更多平台。

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

  OpenClaw 是一款**自托管网关**，将您常用的聊天应用（WhatsApp、Telegram、Discord、iMessage 等）与 Pi 等 AI 编程代理连接。您在本机或服务器上运行一个 Gateway 进程，即可在消息应用与常驻 AI 助手之间架起桥梁。

  
  
  

  - **多频道网关** — 单一 Gateway 进程支持 WhatsApp、Telegram、Discord、iMessage；通过扩展包添加 Mattermost 等。

## 什么是 OpenClaw

**面向谁？** 希望在任何地方通过消息使用个人 AI 助手的开发者与高级用户，且不交出数据控制权、不依赖托管服务。

**需要什么？** Node 22+、所选提供商的 API 密钥与约 5 分钟。为获得最佳质量与安全，请使用当前一代最强模型。

## 工作原理

1. **安装 OpenClaw** — 执行 onboard 并安装服务。
2. **配对并启动** — 配对 WhatsApp（或其他频道）并启动 Gateway。

```bash
npx openclaw onboard
npx openclaw gateway --port 18789
```

需要完整安装与开发环境？参见 [快速开始](./start/quickstart.md)。

- **远程访问：** [Web 界面](./web.md) 与 [Tailscale](./gateway/tailscale.md)
- **控制台：** Gateway 启动后在浏览器中打开控制台。

  ![](../images/whatsapp-openclaw.jpg)

## 核心能力

- **Gateway** 是会话、路由与频道连接的唯一真实来源。
- **多频道** — 单一 Gateway 支持 WhatsApp、Telegram、Discord、iMessage；通过扩展包添加 Mattermost 等。
- **会话隔离**：按代理、工作区或发送者隔离。
- **收发** 图片、音频与文档。
- **浏览器仪表盘**：聊天、配置、会话与节点。
- **配对 iOS / Android 节点**：Canvas、相机/屏幕与语音工作流。

## 浏览文档

  
  
  
  
  
  
  
  
  
  
  

