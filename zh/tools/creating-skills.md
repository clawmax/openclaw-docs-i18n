

  技能

  
# 创建技能

OpenClaw 的设计易于扩展。“技能”是为您的助手添加新功能的主要方式。

## 什么是技能？

技能是一个包含 `SKILL.md` 文件（该文件向 LLM 提供指令和工具定义）的目录，并可选择性地包含一些脚本或资源。

## 逐步指南：您的第一个技能

### 1\. 创建目录

技能位于您的工作区中，通常是 `~/.openclaw/workspace/skills/`。为您的技能创建一个新文件夹：

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2\. 定义 SKILL.md

在该目录中创建一个 `SKILL.md` 文件。此文件使用 YAML 前言表示元数据，使用 Markdown 表示指令。

```
---
name: hello_world
description: 一个简单的打招呼技能。
---

# Hello World 技能

当用户请求问候时，使用 `echo` 工具说“来自您自定义技能的问候！”。
```

### 3\. 添加工具（可选）

您可以在前言中定义自定义工具，或指示智能体使用现有的系统工具（如 `bash` 或 `browser`）。

### 4\. 刷新 OpenClaw

让您的智能体“刷新技能”或重启网关。OpenClaw 将发现新目录并索引 `SKILL.md` 文件。

## 最佳实践

-   **保持简洁**：指导模型*做什么*，而不是如何成为一个 AI。
-   **安全第一**：如果您的技能使用 `bash`，请确保提示不允许来自不受信任用户输入的任意命令注入。
-   **本地测试**：使用 `openclaw agent --message "use my new skill"` 进行测试。

## 共享技能

您也可以浏览 [ClawHub](https://clawhub.com) 上的技能并为其贡献技能。

[多智能体沙盒与工具](./multi-agent-sandbox-tools.md)[斜杠命令](./slash-commands.md)