

  扩展

  
# OpenProse

OpenProse 是一种可移植的、Markdown 优先的工作流格式，用于编排 AI 会话。在 OpenClaw 中，它作为一个插件提供，安装了 OpenProse 技能包以及一个 `/prose` 斜杠命令。程序存在于 `.prose` 文件中，可以生成多个具有明确控制流的子智能体（agent）。官方网站：[https://www.prose.md](https://www.prose.md)

## 功能特点

-   具有明确并行性的多智能体研究 + 综合
-   可重复的、审批安全的工作流（代码审查、事件分类、内容管道）
-   可在支持的智能体运行时中运行的、可复用的 `.prose` 程序

## 安装与启用

捆绑的插件默认是禁用的。启用 OpenProse：

```bash
openclaw plugins enable open-prose
```

启用插件后重启网关（Gateway）。开发/本地检出：`openclaw plugins install ./extensions/open-prose` 相关文档：[插件](./tools/plugin.md), [插件清单](./plugins/manifest.md), [技能](./tools/skills.md)。

## 斜杠命令

OpenProse 将 `/prose` 注册为用户可调用的技能命令。它路由到 OpenProse VM 指令，并在底层使用 OpenClaw 工具。常用命令：

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## 示例：一个简单的 .prose 文件

```bash
# 两个智能体并行运行的研究 + 综合。

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## 文件位置

OpenProse 在你的工作区下的 `.prose/` 目录中保存状态：

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

用户级别的持久化智能体位于：

```
~/.prose/agents/
```

## 状态模式

OpenProse 支持多种状态后端：

-   **filesystem**（默认）：`.prose/runs/...`
-   **in-context**：临时的，适用于小程序
-   **sqlite**（实验性）：需要 `sqlite3` 二进制文件
-   **postgres**（实验性）：需要 `psql` 和连接字符串

注意：

-   sqlite/postgres 是可选且实验性的
-   postgres 凭证会流入子智能体日志；请使用专用的、权限最低的数据库

## 远程程序

`/prose run <handle/slug>` 解析为 `https://p.prose.md//`。直接 URL 会按原样获取。这使用了 `web_fetch` 工具（或用于 POST 的 `exec`）。

## OpenClaw 运行时映射

OpenProse 程序映射到 OpenClaw 原语：

| OpenProse 概念 | OpenClaw 工具 |
| --- | --- |
| 生成会话 / 任务工具 | `sessions_spawn` |
| 文件读/写 | `read` / `write` |
| 网络获取 | `web_fetch` |

如果你的工具允许列表阻止了这些工具，OpenProse 程序将失败。参见[技能配置](./tools/skills-config.md)。

## 安全与审批

将 `.prose` 文件视为代码。运行前请审查。使用 OpenClaw 工具允许列表和审批关卡来控制副作用。对于确定性的、需要审批的工作流，可以与 [Lobster](./tools/lobster.md) 进行比较。

[插件智能体工具](./plugins/agent-tools.md)[钩子](./automation/hooks.md)