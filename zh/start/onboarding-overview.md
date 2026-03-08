title: "OpenClaw 入门概览：CLI 向导与 macOS 应用设置"
description: "了解如何使用 CLI 向导或 macOS 应用来配置 OpenClaw。选择您的设置路径，并为您的网关配置自定义 AI 提供商。"
keywords: ["openclaw 入门", "cli 向导", "macos 应用设置", "自定义提供商", "网关配置", "openai 兼容", "anthropic 兼容", "入门路径"]
---

  第一步

  
# 入门概览

OpenClaw 支持多种入门路径，具体取决于网关的运行位置以及您配置提供商的偏好方式。

## 选择您的入门路径

-   **CLI 向导** 适用于 macOS、Linux 和 Windows（通过 WSL2）。
-   **macOS 应用** 适用于 Apple 芯片或 Intel 芯片 Mac 的引导式首次运行。

## CLI 入门向导

在终端中运行向导：

```bash
openclaw onboard
```

当您希望对网关、工作区、通道和技能拥有完全控制权时，请使用 CLI 向导。相关文档：

-   [入门向导 (CLI)](./wizard.md)
-   [`openclaw onboard` 命令](../cli/onboard.md)

## macOS 应用入门

当您希望在 macOS 上进行完全引导式设置时，请使用 OpenClaw 应用。相关文档：

-   [入门 (macOS 应用)](./onboarding.md)

## 自定义提供商

如果您需要的端点不在列表中，包括暴露标准 OpenAI 或 Anthropic API 的托管提供商，请在 CLI 向导中选择 **自定义提供商**。系统将要求您：

-   选择 OpenAI 兼容、Anthropic 兼容或 **未知**（自动检测）。
-   输入基础 URL 和 API 密钥（如果提供商需要）。
-   提供模型 ID 和可选别名。
-   选择一个端点 ID，以便多个自定义端点可以共存。

详细步骤，请遵循上述 CLI 入门文档。

[开始使用](./getting-started.md)[入门：CLI](./wizard.md)