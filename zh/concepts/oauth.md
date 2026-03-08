

  基础

  
# OAuth

OpenClaw 通过 OAuth 支持提供此功能的供应商的“订阅认证”（特别是 **OpenAI Codex (ChatGPT OAuth)**）。对于 Anthropic 订阅，请使用 **setup-token** 流程。过去 Anthropic 订阅在 Claude Code 之外的使用曾对部分用户受限，因此请将其视为用户自选风险，并自行核实 Anthropic 的当前政策。OpenAI Codex OAuth 明确支持在 OpenClaw 等外部工具中使用。本页说明：对于生产环境中的 Anthropic，API 密钥认证是比订阅 setup-token 认证更安全、更推荐的路径。

-   OAuth **令牌交换** 如何工作（PKCE）
-   令牌**存储**在何处（及原因）
-   如何处理**多个账户**（配置文件 + 每会话覆盖）

OpenClaw 还支持**供应商插件**，这些插件自带其 OAuth 或 API 密钥流程。通过以下命令运行：

```bash
openclaw models auth login --provider <id>
```

## 令牌接收器（为何存在）

OAuth 供应商通常在登录/刷新流程中生成**新的刷新令牌**。某些供应商（或 OAuth 客户端）在为同一用户/应用颁发新令牌时，可能会使旧的刷新令牌失效。实际表现：

-   你通过 OpenClaw *以及* Claude Code / Codex CLI 登录 → 之后其中一个会随机“被登出”

为了减少这种情况，OpenClaw 将 `auth-profiles.json` 视为**令牌接收器**：

-   运行时从**一个位置**读取凭据
-   我们可以保留多个配置文件并确定性地路由它们

## 存储（令牌存放位置）

密钥按**代理**存储：

-   认证配置文件（OAuth + API 密钥 + 可选的值级引用）：`~/.openclaw/agents//agent/auth-profiles.json`
-   旧版兼容文件：`~/.openclaw/agents//agent/auth.json`（静态 `api_key` 条目在发现时会被清除）

旧版仅导入文件（仍受支持，但非主存储）：

-   `~/.openclaw/credentials/oauth.json`（首次使用时导入到 `auth-profiles.json`）

以上所有也遵循 `$OPENCLAW_STATE_DIR`（状态目录覆盖）。完整参考：[/gateway/configuration](../gateway/configuration.md#auth-storage-oauth--api-keys) 关于静态密钥引用和运行时快照激活行为，请参阅[密钥管理](../gateway/secrets.md)。

## Anthropic setup-token（订阅认证）

> **⚠️** Anthropic setup-token 支持是技术兼容性，而非政策保证。Anthropic 过去曾阻止在 Claude Code 之外的部分订阅使用。请自行决定是否使用订阅认证，并核实 Anthropic 的当前条款。

 在任何机器上运行 `claude setup-token`，然后将其粘贴到 OpenClaw：

```bash
openclaw models auth setup-token --provider anthropic
```

如果你在其他地方生成了令牌，请手动粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

验证：

```bash
openclaw models status
```

## OAuth 交换（登录如何工作）

OpenClaw 的交互式登录流程在 `@mariozechner/pi-ai` 中实现，并连接到向导/命令。

### Anthropic setup-token

流程形式：

1.  运行 `claude setup-token`
2.  将令牌粘贴到 OpenClaw
3.  存储为令牌认证配置文件（无刷新）

向导路径是 `openclaw onboard` → 认证选择 `setup-token`（Anthropic）。

### OpenAI Codex (ChatGPT OAuth)

OpenAI Codex OAuth 明确支持在 Codex CLI 之外使用，包括 OpenClaw 工作流。流程形式（PKCE）：

1.  生成 PKCE 验证器/挑战 + 随机 `state`
2.  打开 `https://auth.openai.com/oauth/authorize?...`
3.  尝试在 `http://127.0.0.1:1455/auth/callback` 捕获回调
4.  如果回调无法绑定（或者你是远程/无头环境），请粘贴重定向 URL/代码
5.  在 `https://auth.openai.com/oauth/token` 进行交换
6.  从访问令牌中提取 `accountId` 并存储 `{ access, refresh, expires, accountId }`

向导路径是 `openclaw onboard` → 认证选择 `openai-codex`。

## 刷新 + 过期

配置文件存储一个 `expires` 时间戳。在运行时：

-   如果 `expires` 在未来 → 使用存储的访问令牌
-   如果已过期 → 刷新（在文件锁下）并覆盖存储的凭据

刷新流程是自动的；通常你不需要手动管理令牌。

## 多个账户（配置文件）+ 路由

两种模式：

### 1) 推荐：独立的代理

如果你希望“个人”和“工作”永不交互，请使用隔离的代理（独立的会话 + 凭据 + 工作空间）：

```bash
openclaw agents add work
openclaw agents add personal
```

然后按代理配置认证（向导）并将聊天路由到正确的代理。

### 2) 高级：一个代理中的多个配置文件

`auth-profiles.json` 支持同一供应商的多个配置文件 ID。选择使用哪个配置文件：

-   通过配置顺序全局指定（`auth.order`）
-   通过 `/model ...@` 按会话指定

示例（会话覆盖）：

-   `/model Opus@anthropic:work`

如何查看存在的配置文件 ID：

-   `openclaw channels list --json`（显示 `auth[]`）

相关文档：

-   [/concepts/model-failover](./model-failover.md)（轮换 + 冷却规则）
-   [/tools/slash-commands](../tools/slash-commands.md)（命令界面）

[代理工作空间](./agent-workspace.md)[引导启动](../start/bootstrapping.md)