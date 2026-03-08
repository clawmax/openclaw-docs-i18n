

  环境与调试

  
# 调试

本页涵盖了用于调试流式输出的助手工具，特别是当提供者将推理内容混入普通文本时。

## 运行时调试覆盖

在聊天中使用 `/debug` 来设置**仅运行时**的配置覆盖（存储在内存中，而非磁盘）。`/debug` 默认是禁用的；通过设置 `commands.debug: true` 来启用。当您需要切换一些不常用的设置而无需编辑 `openclaw.json` 时，这非常方便。示例：

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` 会清除所有覆盖并恢复到磁盘上的配置。

## 网关监视模式

为了快速迭代，在文件监视器下运行网关：

```bash
pnpm gateway:watch
```

这映射到：

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

在 `gateway:watch` 之后添加任何网关 CLI 标志，它们将在每次重启时被传递。

## 开发配置文件 + 开发网关 (—dev)

使用开发配置文件来隔离状态，并启动一个安全、可丢弃的设置用于调试。这里有两个 `--dev` 标志：

-   **全局 `--dev` (配置文件):** 将状态隔离在 `~/.openclaw-dev` 下，并将网关端口默认设置为 `19001`（派生的端口会随之偏移）。
-   **`gateway --dev`: 告诉网关在配置和工作区缺失时自动创建默认配置 + 工作区**（并跳过 BOOTSTRAP.md）。

推荐流程（开发配置文件 + 开发引导）：

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

如果您尚未全局安装 CLI，可以通过 `pnpm openclaw ...` 运行。此操作的作用：

1.  **配置文件隔离** (全局 `--dev`)
    -   `OPENCLAW_PROFILE=dev`
    -   `OPENCLAW_STATE_DIR=~/.openclaw-dev`
    -   `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
    -   `OPENCLAW_GATEWAY_PORT=19001`（浏览器/画布端口相应偏移）
2.  **开发引导** (`gateway --dev`)
    -   如果配置缺失，则写入一个最小配置 (`gateway.mode=local`, 绑定环回地址)。
    -   将 `agent.workspace` 设置为开发工作区。
    -   设置 `agent.skipBootstrap=true`（不执行 BOOTSTRAP.md）。
    -   如果缺失，则种子化工作区文件：`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`。
    -   默认身份：**C3‑PO**（礼仪机器人）。
    -   在开发模式下跳过频道提供者 (`OPENCLAW_SKIP_CHANNELS=1`)。

重置流程（全新开始）：

```bash
pnpm gateway:dev:reset
```

注意：`--dev` 是一个**全局**配置文件标志，会被某些运行器“吞掉”。如果需要明确指定，请使用环境变量形式：

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` 会清除配置、凭据、会话和开发工作区（使用 `trash`，而非 `rm`），然后重新创建默认的开发设置。提示：如果非开发网关已在运行（通过 launchd/systemd），请先停止它：

```bash
openclaw gateway stop
```

## 原始流日志记录 (OpenClaw)

OpenClaw 可以在任何过滤/格式化之前记录**原始助手流**。这是查看推理内容是否作为纯文本增量（或作为独立的思考块）到达的最佳方式。通过 CLI 启用：

```bash
pnpm gateway:watch --raw-stream
```

可选路径覆盖：

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

等效的环境变量：

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

默认文件：`~/.openclaw/logs/raw-stream.jsonl`

## 原始块日志记录 (pi-mono)

为了在原始块被解析成块之前捕获**原始的 OpenAI 兼容块**，pi-mono 暴露了一个单独的记录器：

```
PI_RAW_STREAM=1
```

可选路径：

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

默认文件：`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注意：这仅由使用 pi-mono 的 `openai-completions` 提供者的进程发出。

## 安全须知

-   原始流日志可能包含完整的提示词、工具输出和用户数据。
-   请将日志保存在本地，并在调试后删除它们。
-   如果共享日志，请先清理掉其中的秘密信息和 PII（个人身份信息）。

[环境变量](./environment.md)[测试](./testing.md)

---