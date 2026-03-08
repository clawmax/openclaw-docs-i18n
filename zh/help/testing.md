

  环境与调试

  
# 测试

OpenClaw 拥有三个 Vitest 测试套件（单元/集成、端到端、实时）和一小部分 Docker 运行器。本文档是一份“我们如何测试”的指南：

-   每个套件涵盖的范围（以及它*故意不*涵盖的范围）
-   针对常见工作流（本地、推送前、调试）应运行哪些命令
-   实时测试如何发现凭据并选择模型/提供商
-   如何为现实世界中的模型/提供商问题添加回归测试

## 快速开始

大多数情况下：

-   完整门禁（推送前预期执行）：`pnpm build && pnpm check && pnpm test`

当你修改了测试或需要额外信心时：

-   覆盖率门禁：`pnpm test:coverage`
-   端到端测试套件：`pnpm test:e2e`

当调试真实提供商/模型时（需要真实凭据）：

-   实时测试套件（模型 + 网关工具/图像探针）：`pnpm test:live`

提示：当你只需要运行一个失败的用例时，建议使用下文描述的允许列表环境变量来缩小实时测试的范围。

## 测试套件（分别在何处运行）

可以将这些套件视为“真实性递增”（以及不稳定性/成本递增）：

### 单元 / 集成测试（默认）

-   命令：`pnpm test`
-   配置：`scripts/test-parallel.mjs`（运行 `vitest.unit.config.ts`、`vitest.extensions.config.ts`、`vitest.gateway.config.ts`）
-   文件：`src/**/*.test.ts`、`extensions/**/*.test.ts`
-   范围：
    -   纯单元测试
    -   进程内集成测试（网关认证、路由、工具、解析、配置）
    -   针对已知错误的确定性回归测试
-   期望：
    -   在 CI 中运行
    -   无需真实密钥
    -   应快速且稳定
-   池子说明：
    -   OpenClaw 在 Node 22/23 上使用 Vitest `vmForks` 以实现更快的单元分片。
    -   在 Node 24+ 上，OpenClaw 会自动回退到常规 `forks`，以避免 Node VM 链接错误（`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`）。
    -   使用 `OPENCLAW_TEST_VM_FORKS=0`（强制使用 `forks`）或 `OPENCLAW_TEST_VM_FORKS=1`（强制使用 `vmForks`）手动覆盖。

### 端到端测试（网关冒烟测试）

-   命令：`pnpm test:e2e`
-   配置：`vitest.e2e.config.ts`
-   文件：`src/**/*.e2e.test.ts`
-   运行时默认值：
    -   使用 Vitest `vmForks` 以加快文件启动速度。
    -   使用自适应工作线程（CI：2-4，本地：4-8）。
    -   默认在静默模式下运行以减少控制台 I/O 开销。
-   有用的覆盖选项：
    -   `OPENCLAW_E2E_WORKERS=` 强制指定工作线程数量（上限为 16）。
    -   `OPENCLAW_E2E_VERBOSE=1` 重新启用详细控制台输出。
-   范围：
    -   多实例网关端到端行为
    -   WebSocket/HTTP 接口、节点配对以及更重的网络操作
-   期望：
    -   在 CI 中运行（当在流水线中启用时）
    -   无需真实密钥
    -   比单元测试有更多移动部件（可能更慢）

### 实时测试（真实提供商 + 真实模型）

-   命令：`pnpm test:live`
-   配置：`vitest.live.config.ts`
-   文件：`src/**/*.live.test.ts`
-   默认：**由 `pnpm test:live` 启用**（设置 `OPENCLAW_LIVE_TEST=1`）
-   范围：
    -   “这个提供商/模型今天是否真的能用真实凭据工作？”
    -   捕获提供商格式变更、工具调用怪癖、认证问题和速率限制行为
-   期望：
    -   设计上在 CI 中不稳定（真实网络、真实提供商策略、配额、中断）
    -   消耗资金 / 使用速率限制
    -   建议运行缩小的子集，而不是“全部”
    -   实时运行会读取 `~/.profile` 以获取缺失的 API 密钥
-   API 密钥轮换（提供商特定）：设置 `*_API_KEYS` 使用逗号/分号格式，或 `*_API_KEY_1`、`*_API_KEY_2`（例如 `OPENAI_API_KEYS`、`ANTHROPIC_API_KEYS`、`GEMINI_API_KEYS`）或通过 `OPENCLAW_LIVE_*_KEY` 进行每次实时覆盖；测试会在收到速率限制响应时重试。

## 我应该运行哪个测试套件？

使用此决策表：

-   编辑逻辑/测试：运行 `pnpm test`（如果改动较大，则运行 `pnpm test:coverage`）
-   涉及网关网络 / WS 协议 / 配对：添加 `pnpm test:e2e`
-   调试“我的机器人宕机” / 提供商特定故障 / 工具调用：运行缩小的 `pnpm test:live`

## 实时：安卓节点能力扫描

-   测试：`src/gateway/android-node.capabilities.live.test.ts`
-   脚本：`pnpm android:test:integration`
-   目标：调用已连接的安卓节点**当前公布的所有命令**，并断言命令契约行为。
-   范围：
    -   预置条件/手动设置（套件不安装/运行/配对应用）。
    -   针对所选安卓节点，逐个命令进行网关 `node.invoke` 验证。
-   必需的预设：
    -   安卓应用已连接并配对到网关。
    -   应用保持在前台。
    -   已授予你期望通过的权限/捕获同意。
-   可选目标覆盖：
    -   `OPENCLAW_ANDROID_NODE_ID` 或 `OPENCLAW_ANDROID_NODE_NAME`。
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`。
-   完整的安卓设置详情：[安卓应用](../platforms/android.md)

## 实时：模型冒烟测试（配置文件密钥）

实时测试分为两层，以便隔离故障：

-   “直接模型”测试告诉我们，在给定密钥下，提供商/模型是否能够响应。
-   “网关冒烟”测试告诉我们，针对该模型的完整网关+代理流水线是否工作（会话、历史记录、工具、沙箱策略等）。

### 第 1 层：直接模型补全（无网关）

-   测试：`src/agents/models.profiles.live.test.ts`
-   目标：
    -   枚举发现的模型
    -   使用 `getApiKeyForModel` 选择你拥有凭据的模型
    -   对每个模型运行一个小型补全（并在需要时运行有针对性的回归测试）
-   如何启用：
    -   `pnpm test:live`（或直接调用 Vitest 时使用 `OPENCLAW_LIVE_TEST=1`）
-   设置 `OPENCLAW_LIVE_MODELS=modern`（或 `all`，modern 的别名）以实际运行此套件；否则它会跳过，以保持 `pnpm test:live` 专注于网关冒烟测试
-   如何选择模型：
    -   `OPENCLAW_LIVE_MODELS=modern` 运行现代允许列表（Opus/Sonnet/Haiku 4.5、GPT-5.x + Codex、Gemini 3、GLM 4.7、MiniMax M2.5、Grok 4）
    -   `OPENCLAW_LIVE_MODELS=all` 是现代允许列表的别名
    -   或 `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."`（逗号分隔的允许列表）
-   如何选择提供商：
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"`（逗号分隔的允许列表）
-   密钥来源：
    -   默认：配置文件存储和环境变量回退
    -   设置 `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` 以强制仅使用**配置文件存储**
-   存在的原因：
    -   将“提供商 API 损坏 / 密钥无效”与“网关代理流水线损坏”分开
    -   包含小型、隔离的回归测试（例如：OpenAI Responses/Codex Responses 推理回放 + 工具调用流程）

### 第 2 层：网关 + 开发代理冒烟测试（“@openclaw”实际执行的内容）

-   测试：`src/gateway/gateway-models.profiles.live.test.ts`
-   目标：
    -   启动一个进程内网关
    -   创建/修补一个 `agent:dev:*` 会话（每次运行覆盖模型）
    -   遍历拥有密钥的模型并断言：
        -   “有意义”的响应（无工具）
        -   真实的工具调用有效（读取探针）
        -   可选的其他工具探针（执行+读取探针）
        -   OpenAI 回归路径（仅工具调用 → 后续）保持工作
-   探针详情（以便快速解释故障）：
    -   `read` 探针：测试在工作区写入一个临时值文件，并要求代理 `read` 它并回显该临时值。
    -   `exec+read` 探针：测试要求代理 `exec` 写入一个临时值到临时文件，然后 `read` 回来。
    -   图像探针：测试附加一个生成的 PNG（猫 + 随机代码），并期望模型返回 `cat `。
    -   实现参考：`src/gateway/gateway-models.profiles.live.test.ts` 和 `src/gateway/live-image-probe.ts`。
-   如何启用：
    -   `pnpm test:live`（或直接调用 Vitest 时使用 `OPENCLAW_LIVE_TEST=1`）
-   如何选择模型：
    -   默认：现代允许列表（Opus/Sonnet/Haiku 4.5、GPT-5.x + Codex、Gemini 3、GLM 4.7、MiniMax M2.5、Grok 4）
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` 是现代允许列表的别名
    -   或设置 `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"`（或逗号分隔列表）以缩小范围
-   如何选择提供商（避免“OpenRouter 所有内容”）：
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"`（逗号分隔的允许列表）
-   工具和图像探针在此实时测试中始终启用：
    -   `read` 探针 + `exec+read` 探针（工具压力测试）
    -   当模型声明支持图像输入时，运行图像探针
    -   流程（高层次）：
        -   测试生成一个带有“CAT”+随机代码的微小 PNG（`src/gateway/live-image-probe.ts`）
        -   通过 `agent` `attachments: [{ mimeType: "image/png", content: "" }]` 发送
        -   网关将附件解析为 `images[]`（`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`）
        -   嵌入式代理将多模态用户消息转发给模型
        -   断言：回复包含 `cat` + 代码（OCR 容错：允许轻微错误）

提示：要查看你可以在机器上测试的内容（以及确切的 `provider/model` id），请运行：

```bash
openclaw models list
openclaw models list --json
```

## 实时：Anthropic 设置令牌冒烟测试

-   测试：`src/agents/anthropic.setup-token.live.test.ts`
-   目标：验证 Claude Code CLI 设置令牌（或粘贴的设置令牌配置文件）能否完成一个 Anthropic 提示。
-   启用：
    -   `pnpm test:live`（或直接调用 Vitest 时使用 `OPENCLAW_LIVE_TEST=1`）
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   令牌来源（选择一种）：
    -   配置文件：`OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   原始令牌：`OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   模型覆盖（可选）：
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

设置示例：

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## 实时：CLI 后端冒烟测试（Claude Code CLI 或其他本地 CLI）

-   测试：`src/gateway/gateway-cli-backend.live.test.ts`
-   目标：使用本地 CLI 后端验证网关 + 代理流水线，而不触及你的默认配置。
-   启用：
    -   `pnpm test:live`（或直接调用 Vitest 时使用 `OPENCLAW_LIVE_TEST=1`）
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   默认值：
    -   模型：`claude-cli/claude-sonnet-4-6`
    -   命令：`claude`
    -   参数：`["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   覆盖选项（可选）：
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` 发送真实的图像附件（路径会注入到提示中）。
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` 将图像文件路径作为 CLI 参数传递，而不是提示注入。
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"`（或 `"list"`）控制当设置了 `IMAGE_ARG` 时如何传递图像参数。
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` 发送第二轮对话并验证恢复流程。
-   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` 保持 Claude Code CLI MCP 配置启用（默认使用临时空文件禁用 MCP 配置）。

示例：

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### 推荐的实时测试方案

缩小、明确的允许列表最快且最稳定：

-   单个模型，直接测试（无网关）：
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   单个模型，网关冒烟测试：
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   跨多个提供商的工具调用：
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Google 重点测试（Gemini API 密钥 + Antigravity）：
    -   Gemini（API 密钥）：`OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity（OAuth）：`OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

注意：

-   `google/...` 使用 Gemini API（API 密钥）。
-   `google-antigravity/...` 使用 Antigravity OAuth 桥接（Cloud Code Assist 风格的代理端点）。
-   `google-gemini-cli/...` 使用你机器上的本地 Gemini CLI（独立的认证 + 工具怪癖）。
-   Gemini API 与 Gemini CLI：
    -   API：OpenClaw 通过 HTTP 调用 Google 托管的 Gemini API（API 密钥 / 配置文件认证）；这是大多数用户所说的“Gemini”。
    -   CLI：OpenClaw 调用本地 `gemini` 二进制文件；它有自己的认证，并且行为可能不同（流式传输/工具支持/版本偏差）。

## 实时：模型矩阵（我们涵盖的内容）

没有固定的“CI 模型列表”（实时测试是选择加入的），但以下是在拥有密钥的开发机器上**推荐**定期覆盖的模型。

### 现代冒烟测试集（工具调用 + 图像）

这是我们认为应保持工作的“常见模型”运行集：

-   OpenAI（非 Codex）：`openai/gpt-5.2`（可选：`openai/gpt-5.1`）
-   OpenAI Codex：`openai-codex/gpt-5.4`
-   Anthropic：`anthropic/claude-opus-4-6`（或 `anthropic/claude-sonnet-4-5`）
-   Google（Gemini API）：`google/gemini-3-pro-preview` 和 `google/gemini-3-flash-preview`（避免旧的 Gemini 2.x 模型）
-   Google（Antigravity）：`google-antigravity/claude-opus-4-6-thinking` 和 `google-antigravity/gemini-3-flash`
-   Z.AI（GLM）：`zai/glm-4.7`
-   MiniMax：`minimax/minimax-m2.5`

使用工具 + 图像运行网关冒烟测试：`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### 基线：工具调用（读取 + 可选执行）

每个提供商家族至少选择一个：

-   OpenAI：`openai/gpt-5.2`（或 `openai/gpt-5-mini`）
-   Anthropic：`anthropic/claude-opus-4-6`（或 `anthropic/claude-sonnet-4-5`）
-   Google：`google/gemini-3-flash-preview`（或 `google/gemini-3-pro-preview`）
-   Z.AI（GLM）：`zai/glm-4.7`
-   MiniMax：`minimax/minimax-m2.5`

可选额外覆盖（最好拥有）：

-   xAI：`xai/grok-4`（或最新可用版本）
-   Mistral：`mistral/`…（选择一个你启用的支持“工具”的模型）
-   Cerebras：`cerebras/`…（如果你有访问权限）
-   LM Studio：`lmstudio/`…（本地；工具调用取决于 API 模式）

### 视觉：图像发送（附件 → 多模态消息）

在 `OPENCLAW_LIVE_GATEWAY_MODELS` 中至少包含一个支持图像的模型（Claude/Gemini/OpenAI 支持视觉的变体等），以运行图像探针。

### 聚合器 / 替代网关

如果你启用了密钥，我们还支持通过以下方式测试：

-   OpenRouter：`openrouter/...`（数百个模型；使用 `openclaw models scan` 查找支持工具+图像的候选模型）
-   OpenCode Zen：`opencode/...`（通过 `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY` 认证）

更多可以包含在实时矩阵中的提供商（如果你有凭据/配置）：

-   内置：`openai`、`openai-codex`、`anthropic`、`google`、`google-vertex`、`google-antigravity`、`google-gemini-cli`、`zai`、`openrouter`、`opencode`、`xai`、`groq`、`cerebras`、`mistral`、`github-copilot`
-   通过 `models.providers`（自定义端点）：`minimax`（云/API），以及任何 OpenAI/Anthropic 兼容的代理（LM Studio、vLLM、LiteLLM 等）

提示：不要试图在文档中硬编码“所有模型”。权威列表是你的机器上 `discoverModels(...)` 返回的内容 + 任何可用的密钥。

## 凭据（切勿提交）

实时测试发现凭据的方式与 CLI 相同。实际影响：

-   如果 CLI 工作，实时测试应该能找到相同的密钥。
-   如果实时测试说“没有凭据”，请按照调试 `openclaw models list` / 模型选择的方式进行调试。
-   配置文件存储：`~/.openclaw/credentials/`（首选；测试中“配置文件密钥”的含义）
-   配置：`~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）

如果你想依赖环境变量密钥（例如，在你的 `~/.profile` 中导出），请在 `source ~/.profile` 后运行本地测试，或使用下面的 Docker 运行器（它们可以将 `~/.profile` 挂载到容器中）。

## Deepgram 实时测试（音频转录）

-   测试：`src/media-understanding/providers/deepgram/audio.live.test.ts`
-   启用：`DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## BytePlus 编码计划实时测试

-   测试：`src/agents/byteplus.live.test.ts`
-   启用：`BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   可选模型覆盖：`BYTEPLUS_CODING_MODEL=ark-code-latest`

## Docker 运行器（可选的“在 Linux 中工作”检查）

这些在仓库 Docker 镜像内运行 `pnpm test:live`，挂载你的本地配置目录和工作区（如果挂载了，还会读取 `~/.profile`）：

-   直接模型：`pnpm test:docker:live-models`（脚本：`scripts/test-live-models-docker.sh`）
-   网关 + 开发代理：`pnpm test:docker:live-gateway`（脚本：`scripts/test-live-gateway-models-docker.sh`）
-   入门向导（TTY，完整脚手架）：`pnpm test:docker:onboard`（脚本：`scripts/e2e/onboard-docker.sh`）
-   网关网络（两个容器，WS 认证 + 健康检查）：`pnpm test:docker:gateway-network`（脚本：`scripts/e2e/gateway-network-docker.sh`）
-   插件（自定义扩展加载 + 注册表冒烟测试）：`pnpm test:docker:plugins`（脚本：`scripts/e2e/plugins-docker.sh`）

手动 ACP 纯语言线程冒烟测试（非 CI）：

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   保留此脚本用于回归/调试工作流。未来可能再次需要它进行 ACP 线程路由验证，因此不要删除它。

有用的环境变量：

-   `OPENCLAW_CONFIG_DIR=...`（默认：`~/.openclaw`）挂载到 `/home/node/.openclaw`
-   `OPENCLAW_WORKSPACE_DIR=...`（默认：`~/.openclaw/workspace`）挂载到 `/home/node/.openclaw/workspace`
-   `OPENCLAW_PROFILE_FILE=...`（默认：`~/.profile`）挂载到 `/home/node/.profile` 并在运行测试前读取
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` 以缩小运行范围
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` 确保凭据来自配置文件存储（而非环境变量）

## 文档完整性检查

文档编辑后运行文档检查：`pnpm docs:list`。

## 离线回归测试（CI 安全）

这些是“真实流水线”回归测试，但无需真实提供商：

-   网关工具调用（模拟 OpenAI，真实网关 + 代理循环）：`src/gateway/gateway.test.ts`（用例：“通过网关代理循环运行模拟 OpenAI 工具调用端到端”）
-   网关向导（WS `wizard.start`/`wizard.next`，写入配置 + 强制认证）：`src/gateway/gateway.test.ts`（用例：“通过 ws 运行向导并写入认证令牌配置”）

## 代理可靠性评估（技能）

我们已经有一些 CI 安全的测试，其行为类似于“代理可靠性评估”：

-   通过真实网关 + 代理循环进行模拟工具调用（`src/gateway/gateway.test.ts`）。
-   验证会话连接和配置影响的端到端向导流程（`src/gateway/gateway.test.ts`）。

对于技能（参见[技能](../tools/skills.md)）仍然缺失的内容：

-   **决策：** 当技能在提示中列出时，代理是否选择了正确的技能（或避免了不相关的技能）？
-   **合规性：** 代理在使用前是否读取了 `SKILL.md` 并遵循了必需的步骤/参数？
-   **工作流契约：** 多轮场景，断言工具顺序、会话历史记录延续和沙箱边界。

未来的评估应首先保持确定性：

-   使用模拟提供商的场景运行器，断言工具调用 + 顺序、技能文件读取和会话连接。
-   一小部分专注于技能的场景（使用与避免、门控、提示注入）。
-   只有在 CI 安全套件就位后，才进行可选的实时评估（选择加入，环境变量控制）。

## 添加回归测试（指南）

当你修复了在实时测试中发现的提供商/模型问题时：

-   如果可能，添加一个 CI 安全的回归测试（模拟/存根提供商，或捕获确切的请求形状转换）
-   如果它本质上是仅限实时的（速率限制、认证策略），请保持实时测试范围狭窄并通过环境变量选择加入
-   倾向于定位能捕获错误的最小层：
    -   提供商请求转换/回放错误 → 直接模型测试
    -   网关会话/历史记录/工具流水线错误 → 网关实时冒烟测试或 CI 安全的网关模拟测试

[调试](./debugging.md)[脚本](./scripts.md)