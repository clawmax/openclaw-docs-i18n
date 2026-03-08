title: "OpenClaw CI 流水线文档与任务概览"
description: "了解 OpenClaw CI 流水线、其智能作用域、任务目的，以及用于检查、测试和发布的本地等效命令。"
keywords: ["ci 流水线", "持续集成", "智能作用域", "ci 任务", "本地开发命令", "pnpm check", "blacksmith runner", "贡献工作流"]
---

  贡献

  
# CI 流水线

CI 在每次推送到 `main` 分支以及每个拉取请求时运行。它使用智能作用域，以便在仅文档或原生代码发生更改时跳过开销较大的任务。

## 任务概览

| 任务 | 目的 | 何时运行 |
| --- | --- | --- |
| `docs-scope` | 检测仅文档更改 | 始终 |
| `changed-scope` | 检测哪些区域发生更改 (node/macos/android/windows) | 非文档 PR |
| `check` | TypeScript 类型检查、代码检查、格式化 | 推送到 `main`，或包含 Node 相关更改的 PR |
| `check-docs` | Markdown 检查 + 损坏链接检查 | 文档发生更改 |
| `code-analysis` | 代码行数阈值检查 (1000 行) | 仅限 PR |
| `secrets` | 检测泄露的密钥 | 始终 |
| `build-artifacts` | 构建一次 dist，与其他任务共享 | 非文档，Node 更改 |
| `release-check` | 验证 npm 包内容 | 构建之后 |
| `checks` | Node/Bun 测试 + 协议检查 | 非文档，Node 更改 |
| `checks-windows` | Windows 特定测试 | 非文档，与 Windows 相关的更改 |
| `macos` | Swift 代码检查/构建/测试 + TS 测试 | 包含 macOS 更改的 PR |
| `android` | Gradle 构建 + 测试 | 非文档，Android 更改 |

## 快速失败顺序

任务按顺序排列，以便在开销较大的任务运行之前，开销较小的检查先失败：

1.  `docs-scope` + `code-analysis` + `check` (并行，约 1-2 分钟)
2.  `build-artifacts` (等待上述任务完成)
3.  `checks`, `checks-windows`, `macos`, `android` (等待构建完成)

作用域逻辑位于 `scripts/ci-changed-scope.mjs` 中，并由 `src/scripts/ci-changed-scope.test.ts` 中的单元测试覆盖。

## 运行器

| 运行器 | 任务 |
| --- | --- |
| `blacksmith-16vcpu-ubuntu-2404` | 大多数 Linux 任务，包括作用域检测 |
| `blacksmith-32vcpu-windows-2025` | `checks-windows` |
| `macos-latest` | `macos`, `ios` |

## 本地等效命令

```bash
pnpm check          # 类型检查 + 代码检查 + 格式化
pnpm test           # vitest 测试
pnpm check:docs     # 文档格式化 + 检查 + 损坏链接检查
pnpm release:check  # 验证 npm 包
```

[Pi 开发工作流](./pi-dev.md)[文档中心](./start/hubs.md)

---