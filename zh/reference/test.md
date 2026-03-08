

  发布说明

  
# 测试

-   完整测试套件（测试套件、实时测试、Docker）：[测试](../help/testing.md)
-   `pnpm test:force`：终止任何占用默认控制端口的残留网关进程，然后使用隔离的网关端口运行完整的 Vitest 套件，以避免服务器测试与正在运行的实例冲突。当之前的网关运行占用了端口 18789 时，请使用此命令。
-   `pnpm test:coverage`：使用 V8 覆盖率运行单元测试套件（通过 `vitest.unit.config.ts`）。全局阈值是 70% 的行/分支/函数/语句覆盖率。覆盖率排除了集成密集的入口点（CLI 布线、网关/Telegram 桥接、webchat 静态服务器），以使目标专注于可单元测试的逻辑。
-   Node 24+ 上的 `pnpm test`：OpenClaw 自动禁用 Vitest 的 `vmForks` 并使用 `forks`，以避免 `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked` 错误。您可以使用 `OPENCLAW_TEST_VM_FORKS=0|1` 强制指定行为。
-   `pnpm test`：默认运行快速核心单元测试通道，以获取快速的本地反馈。
-   `pnpm test:channels`：运行频道密集的测试套件。
-   `pnpm test:extensions`：运行扩展/插件测试套件。
-   网关集成：通过 `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` 或 `pnpm test:gateway` 选择启用。
-   `pnpm test:e2e`：运行网关端到端冒烟测试（多实例 WS/HTTP/节点配对）。默认使用 `vitest.e2e.config.ts` 中的 `vmForks` + 自适应工作线程；可通过 `OPENCLAW_E2E_WORKERS=` 调整，并设置 `OPENCLAW_E2E_VERBOSE=1` 以获取详细日志。
-   `pnpm test:live`：运行供应商实时测试（minimax/zai）。需要 API 密钥和 `LIVE=1`（或特定于供应商的 `*_LIVE_TEST=1`）来取消跳过。

## 本地 PR 门控

对于本地 PR 合并/门控检查，请运行：

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

如果 `pnpm test` 在负载较重的主机上出现不稳定，请在将其视为回归问题前重试一次，然后使用 `pnpm vitest run <path/to/test>` 进行隔离测试。对于内存受限的主机，请使用：

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## 模型延迟基准测试（本地密钥）

脚本：[`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) 用法：

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   可选环境变量：`MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   默认提示词：“用一个词回复：ok。不要标点或额外文本。”

最近一次运行（2025-12-31，20 次运行）：

-   minimax 中位数 1279ms（最小 1114，最大 2431）
-   opus 中位数 2454ms（最小 1224，最大 3170）

## CLI 启动基准测试

脚本：[`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) 用法：

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

此基准测试针对以下命令：

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

输出包括每个命令的平均值、p50、p95、最小/最大值以及退出码/信号分布。

## 入门端到端测试（Docker）

Docker 是可选的；仅在进行容器化的入门冒烟测试时需要。在干净的 Linux 容器中完整的冷启动流程：

```
scripts/e2e/onboard-docker.sh
```

此脚本通过伪终端驱动交互式向导，验证配置/工作区/会话文件，然后启动网关并运行 `openclaw health`。

## QR 导入冒烟测试（Docker）

确保 `qrcode-terminal` 在 Docker 中的 Node 22+ 环境下能正常加载：

```bash
pnpm test:docker:qr
```

[发布清单](./RELEASING.md)[Kilo 网关集成](../design/kilo-gateway-integration.md)

---