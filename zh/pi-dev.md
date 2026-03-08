

  开发者设置

  
# Pi 开发工作流

本指南总结了在 OpenClaw 中处理 Pi 集成的合理工作流程。

## 类型检查与代码检查

-   类型检查与构建：`pnpm build`
-   代码检查：`pnpm lint`
-   格式检查：`pnpm format`
-   推送前的完整检查门禁：`pnpm lint && pnpm build && pnpm test`

## 运行 Pi 测试

使用 Vitest 直接运行 Pi 相关的测试集：

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

要包含实时提供者练习：

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

这涵盖了主要的 Pi 单元测试套件：

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## 手动测试

推荐流程：

-   在开发模式下运行网关：
    -   `pnpm gateway:dev`
-   直接触发代理：
    -   `pnpm openclaw agent --message "Hello" --thinking low`
-   使用 TUI 进行交互式调试：
    -   `pnpm tui`

对于工具调用行为，可以提示一个 `read` 或 `exec` 操作，以便观察工具流式处理和负载处理。

## 全新状态重置

状态位于 OpenClaw 状态目录下。默认为 `~/.openclaw`。如果设置了 `OPENCLAW_STATE_DIR`，则使用该目录。要重置所有内容：

-   `openclaw.json` 用于配置
-   `credentials/` 用于认证配置文件和令牌
-   `agents//sessions/` 用于代理会话历史
-   `agents//sessions.json` 用于会话索引
-   `sessions/` 如果存在遗留路径
-   `workspace/` 如果你想要一个空白的工作区

如果只想重置会话，删除该代理对应的 `agents//sessions/` 和 `agents//sessions.json`。如果不想重新认证，请保留 `credentials/`。

## 参考

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[设置](./start/setup.md)[CI 流水线](./ci.md)

---