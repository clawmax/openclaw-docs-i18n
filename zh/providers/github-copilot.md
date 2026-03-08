title: "如何在 OpenClaw 中使用 GitHub Copilot 作为模型提供商"
description: "学习两种将 GitHub Copilot 与 OpenClaw 集成的方法：内置提供商和 Copilot Proxy 插件。包含设置命令和配置。"
keywords: ["github copilot", "openclaw", "AI 编程助手", "模型提供商", "copilot proxy", "CLI 设置", "身份验证登录", "gpt-4o"]
---

  提供商

  
# GitHub Copilot

## 什么是 GitHub Copilot？

GitHub Copilot 是 GitHub 的 AI 编程助手。它为您 GitHub 账户和订阅计划提供对 Copilot 模型的访问。OpenClaw 可以通过两种不同的方式使用 Copilot 作为模型提供商。

## 在 OpenClaw 中使用 Copilot 的两种方式

### 1) 内置 GitHub Copilot 提供商 (github-copilot)

使用原生的设备登录流程获取 GitHub 令牌，然后在 OpenClaw 运行时将其交换为 Copilot API 令牌。这是**默认**且最简单的路径，因为它不需要 VS Code。

### 2) Copilot Proxy 插件 (copilot-proxy)

使用 **Copilot Proxy** VS Code 扩展作为本地桥接。OpenClaw 与代理的 `/v1` 端点通信，并使用您在那里配置的模型列表。当您已经在 VS Code 中运行 Copilot Proxy 或需要通过它路由时，请选择此方式。您必须启用该插件并保持 VS Code 扩展运行。使用 GitHub Copilot 作为模型提供商 (`github-copilot`)。登录命令会运行 GitHub 设备流程，保存一个身份验证配置文件，并更新您的配置以使用该配置文件。

## CLI 设置

```bash
openclaw models auth login-github-copilot
```

系统将提示您访问一个 URL 并输入一次性代码。请保持终端打开，直到流程完成。

### 可选标志

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## 设置默认模型

```bash
openclaw models set github-copilot/gpt-4o
```

### 配置片段

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## 注意事项

-   需要交互式 TTY；请直接在终端中运行。
-   Copilot 模型的可用性取决于您的订阅计划；如果某个模型被拒绝，请尝试另一个 ID（例如 `github-copilot/gpt-4.1`）。
-   登录操作会将 GitHub 令牌存储在身份验证配置文件中，并在 OpenClaw 运行时将其交换为 Copilot API 令牌。

[Deepgram](./deepgram.md)[Hugging Face (推理)](./huggingface.md)