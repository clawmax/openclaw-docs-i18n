

  配置与运维

  
# 身份验证

OpenClaw 支持模型提供商的 OAuth 和 API 密钥。对于常驻网关主机，API 密钥通常是最可预测的选择。当订阅/OAuth 流程与您的提供商账户模型匹配时，也支持这些流程。完整的 OAuth 流程和存储布局请参阅 [/concepts/oauth](../concepts/oauth.md)。对于基于 SecretRef 的身份验证（`env`/`file`/`exec` 提供商），请参阅 [密钥管理](./secrets.md)。关于 `models status --probe` 使用的凭证资格/原因代码规则，请参阅 [身份验证凭证语义](../auth-credential-semantics.md)。

## 推荐设置（API 密钥，任何提供商）

如果您正在运行一个长期存活的网关，请从为您选择的提供商创建 API 密钥开始。特别是对于 Anthropic，API 密钥身份验证是安全的途径，推荐优先于订阅设置令牌身份验证。

1.  在您的提供商控制台中创建一个 API 密钥。
2.  将其放在 **网关主机**（运行 `openclaw gateway` 的机器）上。

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  如果网关在 systemd/launchd 下运行，建议将密钥放在 `~/.openclaw/.env` 中，以便守护进程可以读取它：

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

然后重启守护进程（或重启您的网关进程）并重新检查：

```bash
openclaw models status
openclaw doctor
```

如果您不想自己管理环境变量，入门向导可以为守护进程存储 API 密钥：`openclaw onboard`。有关环境继承（`env.shellEnv`、`~/.openclaw/.env`、systemd/launchd）的详细信息，请参阅 [帮助](../help.md)。

## Anthropic：设置令牌（订阅身份验证）

如果您使用的是 Claude 订阅，则支持设置令牌流程。在 **网关主机** 上运行：

```bash
claude setup-token
```

然后将其粘贴到 OpenClaw 中：

```bash
openclaw models auth setup-token --provider anthropic
```

如果令牌是在另一台机器上创建的，请手动粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

如果您看到类似以下的 Anthropic 错误：

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

...请改用 Anthropic API 密钥。

> **⚠️** Anthropic 设置令牌支持仅为技术兼容性。Anthropic 过去曾阻止在 Claude Code 之外使用某些订阅。仅在您认为政策风险可接受时使用，并请自行核实 Anthropic 的当前条款。

手动令牌输入（任何提供商；写入 `auth-profiles.json` 并更新配置）：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

身份验证配置文件引用也支持静态凭证：

-   `api_key` 凭证可以使用 `keyRef: { source, provider, id }`
-   `token` 凭证可以使用 `tokenRef: { source, provider, id }`

自动化友好检查（过期/缺失时退出 `1`，即将过期时退出 `2`）：

```bash
openclaw models status --check
```

可选的运维脚本（systemd/Termux）在此处有文档记录：[/automation/auth-monitoring](../automation/auth-monitoring.md)

> `claude setup-token` 需要一个交互式 TTY。

## 检查模型身份验证状态

```bash
openclaw models status
openclaw doctor
```

## API 密钥轮换行为（网关）

当 API 调用达到提供商速率限制时，某些提供商支持使用备用密钥重试请求。

-   优先级顺序：
    -   `OPENCLAW_LIVE__KEY`（单一覆盖）
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   Google 提供商还包括 `GOOGLE_API_KEY` 作为额外的后备。
-   相同的密钥列表在使用前会去重。
-   OpenClaw 仅在遇到速率限制错误（例如 `429`、`rate_limit`、`quota`、`resource exhausted`）时才会使用下一个密钥重试。
-   非速率限制错误不会使用备用密钥重试。
-   如果所有密钥都失败，则返回最后一次尝试的最终错误。

## 控制使用哪个凭证

### 每个会话（聊天命令）

使用 `/model <alias-or-id>@` 为当前会话固定使用特定的提供商凭证（示例配置文件 ID：`anthropic:default`、`anthropic:work`）。使用 `/model`（或 `/model list`）获取紧凑的选择器；使用 `/model status` 获取完整视图（候选者 + 下一个身份验证配置文件，以及配置时的提供商端点详细信息）。

### 每个代理（CLI 覆盖）

为代理设置明确的身验证配置文件顺序覆盖（存储在该代理的 `auth-profiles.json` 中）：

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

使用 `--agent ` 来定位特定代理；省略它以使用配置的默认代理。

## 故障排除

### “未找到凭证”

如果缺少 Anthropic 令牌配置文件，请在 **网关主机** 上运行 `claude setup-token`，然后重新检查：

```bash
openclaw models status
```

### 令牌即将过期/已过期

运行 `openclaw models status` 以确认哪个配置文件即将过期。如果配置文件缺失，请重新运行 `claude setup-token` 并再次粘贴令牌。

## 要求

-   Anthropic 订阅账户（用于 `claude setup-token`）
-   已安装 Claude Code CLI（`claude` 命令可用）

[配置示例](./configuration-examples.md)[身份验证凭证语义](../auth-credential-semantics.md)