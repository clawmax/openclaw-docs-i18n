

  自动化

  
# Gmail PubSub

目标：Gmail watch -> Pub/Sub 推送 -> `gog gmail watch serve` -> OpenClaw webhook。

## 前提条件

-   已安装并登录 `gcloud` ([安装指南](https://docs.cloud.google.com/sdk/docs/install-sdk))。
-   已安装 `gog` (gogcli) 并已为 Gmail 账户授权 ([gogcli.sh](https://gogcli.sh/))。
-   已启用 OpenClaw 钩子 (参见 [Webhooks](./webhook.md))。
-   已登录 `tailscale` ([tailscale.com](https://tailscale.com/))。支持的设置使用 Tailscale Funnel 作为公共 HTTPS 端点。其他隧道服务也可用，但需自行配置且不受支持。目前我们支持的是 Tailscale。

示例钩子配置（启用 Gmail 预设映射）：

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

要将 Gmail 摘要投递到聊天界面，请使用设置了 `deliver` 及可选 `channel`/`to` 的映射来覆盖预设：

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

如果需要固定频道，请设置 `channel` + `to`。否则 `channel: "last"` 会使用上次的投递路由（回退到 WhatsApp）。要为 Gmail 运行强制使用成本更低的模型，请在映射中设置 `model` (`provider/model` 或别名)。如果设置了 `agents.defaults.models`，请确保 Gmail 模型包含在允许列表中。要专门为 Gmail 钩子设置默认模型和思考级别，请在配置中添加 `hooks.gmail.model` / `hooks.gmail.thinking`：

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

注意：

-   映射中的每个钩子 `model`/`thinking` 仍会覆盖这些默认值。
-   回退顺序：`hooks.gmail.model` → `agents.defaults.model.fallbacks` → 主要模型（授权/速率限制/超时）。
-   如果设置了 `agents.defaults.models`，Gmail 模型必须在允许列表中。
-   Gmail 钩子内容默认使用外部内容安全边界包装。要禁用（危险），请设置 `hooks.gmail.allowUnsafeExternalContent: true`。

要进一步自定义负载处理，请添加 `hooks.mappings` 或在 `~/.openclaw/hooks/transforms` 下添加 JS/TS 转换模块（参见 [Webhooks](./webhook.md)）。

## 向导（推荐）

使用 OpenClaw 助手来连接所有组件（在 macOS 上通过 brew 安装依赖）：

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

默认值：

-   使用 Tailscale Funnel 作为公共推送端点。
-   为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。
-   启用 Gmail 钩子预设 (`hooks.presets: ["gmail"]`)。

路径说明：当启用 `tailscale.mode` 时，OpenClaw 会自动将 `hooks.gmail.serve.path` 设置为 `/`，并将公共路径保持在 `hooks.gmail.tailscale.path`（默认为 `/gmail-pubsub`），因为 Tailscale 在代理之前会剥离设置的路径前缀。如果需要后端接收带前缀的路径，请将 `hooks.gmail.tailscale.target`（或 `--tailscale-target`）设置为完整的 URL，例如 `http://127.0.0.1:8788/gmail-pubsub`，并匹配 `hooks.gmail.serve.path`。需要自定义端点？使用 `--push-endpoint ` 或 `--tailscale off`。平台说明：在 macOS 上，向导通过 Homebrew 安装 `gcloud`、`gogcli` 和 `tailscale`；在 Linux 上请先手动安装它们。网关自动启动（推荐）：

-   当 `hooks.enabled=true` 且设置了 `hooks.gmail.account` 时，网关会在启动时启动 `gog gmail watch serve` 并自动续订 watch。
-   设置 `OPENCLAW_SKIP_GMAIL_WATCHER=1` 以选择退出（适用于自行运行守护进程的情况）。
-   不要同时运行手动守护进程，否则会遇到 `listen tcp 127.0.0.1:8788: bind: address already in use` 错误。

手动守护进程（启动 `gog gmail watch serve` + 自动续订）：

```bash
openclaw webhooks gmail run
```

## 一次性设置

1.  选择 **拥有 `gog` 使用的 OAuth 客户端** 的 GCP 项目。

```bash
gcloud auth login
gcloud config set project <project-id>
```

注意：Gmail watch 要求 Pub/Sub 主题与 OAuth 客户端位于同一项目中。

2.  启用 API：

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  创建主题：

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  允许 Gmail 推送进行发布：

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## 启动 watch

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

保存输出中的 `history_id`（用于调试）。

## 运行推送处理器

本地示例（共享令牌认证）：

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

注意：

-   `--token` 保护推送端点 (`x-gog-token` 或 `?token=`)。
-   `--hook-url` 指向 OpenClaw 的 `/hooks/gmail`（已映射；独立运行 + 摘要发送到主进程）。
-   `--include-body` 和 `--max-bytes` 控制发送到 OpenClaw 的正文片段。

推荐：`openclaw webhooks gmail run` 封装了相同的流程并自动续订 watch。

## 暴露处理器（高级，不受支持）

如果需要非 Tailscale 隧道，请手动配置并在推送订阅中使用公共 URL（不受支持，无防护）：

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

使用生成的 URL 作为推送端点：

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

生产环境：使用稳定的 HTTPS 端点并配置 Pub/Sub OIDC JWT，然后运行：

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## 测试

向被监视的收件箱发送消息：

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

检查 watch 状态和历史记录：

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## 故障排除

-   `Invalid topicName`：项目不匹配（主题不在 OAuth 客户端项目中）。
-   `User not authorized`：主题上缺少 `roles/pubsub.publisher` 权限。
-   空消息：Gmail 推送仅提供 `historyId`；通过 `gog gmail history` 获取。

## 清理

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Webhooks](./webhook.md)[轮询](./poll.md)