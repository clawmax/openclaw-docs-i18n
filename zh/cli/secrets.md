

  CLI 命令

  
# secrets

使用 `openclaw secrets` 来管理 SecretRef 并保持活动运行时快照的健康状态。命令角色：

-   `reload`: 网关 RPC (`secrets.reload`)，仅在完全成功时重新解析引用并交换运行时快照（不写入配置）。
-   `audit`: 对配置/认证/生成模型存储以及遗留残留进行只读扫描，查找明文、未解析的引用和优先级漂移。
-   `configure`: 用于提供商设置、目标映射和预检的交互式规划器（需要 TTY）。
-   `apply`: 执行已保存的计划（`--dry-run` 仅用于验证），然后清除目标明文残留。

推荐的操作员循环：

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

CI/门控的退出代码说明：

-   `audit --check` 在发现问题时返回 `1`。
-   未解析的引用返回 `2`。

相关：

-   密钥指南：[密钥管理](../gateway/secrets.md)
-   凭证范围：[SecretRef 凭证范围](../reference/secretref-credential-surface.md)
-   安全指南：[安全](../gateway/security.md)

## 重新加载运行时快照

重新解析密钥引用并以原子方式交换运行时快照。

```bash
openclaw secrets reload
openclaw secrets reload --json
```

说明：

-   使用网关 RPC 方法 `secrets.reload`。
-   如果解析失败，网关将保留最后已知的良好快照并返回错误（无部分激活）。
-   JSON 响应包含 `warningCount`。

## 审计

扫描 OpenClaw 状态以查找：

-   明文密钥存储
-   未解析的引用
-   优先级漂移（`auth-profiles.json` 凭证遮蔽 `openclaw.json` 引用）
-   生成的 `agents/*/agent/models.json` 残留（提供商 `apiKey` 值和敏感的提供商头部）
-   遗留残留（遗留认证存储条目，OAuth 提醒）

头部残留说明：

-   敏感提供商头部检测基于名称启发式（常见的认证/凭证头部名称和片段，如 `authorization`、`x-api-key`、`token`、`secret`、`password` 和 `credential`）。

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

退出行为：

-   `--check` 在发现问题时以非零状态退出。
-   未解析的引用以更高优先级的非零代码退出。

报告结构要点：

-   `status`: `clean | findings | unresolved`
-   `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
-   发现代码：
    -   `PLAINTEXT_FOUND`
    -   `REF_UNRESOLVED`
    -   `REF_SHADOWED`
    -   `LEGACY_RESIDUE`

## 配置（交互式助手）

交互式地构建提供商和 SecretRef 更改，运行预检，并可选择应用：

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

流程：

-   首先进行提供商设置（`add/edit/remove` 用于 `secrets.providers` 别名）。
-   其次进行凭证映射（选择字段并分配 `{source, provider, id}` 引用）。
-   最后进行预检和可选的应用。

标志：

-   `--providers-only`: 仅配置 `secrets.providers`，跳过凭证映射。
-   `--skip-provider-setup`: 跳过提供商设置，将凭证映射到现有提供商。
-   `--agent `: 将 `auth-profiles.json` 目标发现和写入范围限定为一个代理存储。

说明：

-   需要交互式 TTY。
-   不能同时使用 `--providers-only` 和 `--skip-provider-setup`。
-   `configure` 的目标是 `openclaw.json` 中包含密钥的字段，以及所选代理范围内的 `auth-profiles.json`。
-   `configure` 支持在选取器流程中直接创建新的 `auth-profiles.json` 映射。
-   规范支持的范围：[SecretRef 凭证范围](../reference/secretref-credential-surface.md)。
-   它在应用前执行预检解析。
-   生成的计划默认启用清除选项（`scrubEnv`、`scrubAuthProfilesForProviderTargets`、`scrubLegacyAuthJson` 全部启用）。
-   应用路径对于已清除的明文值是单向的。
-   没有 `--apply` 时，CLI 在预检后仍会提示 `Apply this plan now?`。
-   使用 `--apply`（且没有 `--yes`）时，CLI 会提示额外的不可逆确认。

Exec 提供商安全说明：

-   Homebrew 安装通常会在 `/opt/homebrew/bin/*` 下暴露符号链接的二进制文件。
-   仅在需要用于受信任的包管理器路径时设置 `allowSymlinkCommand: true`，并将其与 `trustedDirs` 配对（例如 `["/opt/homebrew"]`）。
-   在 Windows 上，如果提供商路径的 ACL 验证不可用，OpenClaw 将失败关闭。仅对于受信任的路径，在该提供商上设置 `allowInsecurePath: true` 以绕过路径安全检查。

## 应用已保存的计划

应用或预检先前生成的计划：

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

计划合约详情（允许的目标路径、验证规则和失败语义）：

-   [密钥应用计划合约](../gateway/secrets-plan-contract.md)

`apply` 可能更新的内容：

-   `openclaw.json`（SecretRef 目标 + 提供商更新/删除）
-   `auth-profiles.json`（提供商目标清除）
-   遗留的 `auth.json` 残留
-   `~/.openclaw/.env` 中已迁移的已知密钥键值

## 为何没有回滚备份

`secrets apply` 有意不写入包含旧明文值的回滚备份。安全性来自严格的预检 + 原子化应用以及在失败时尽力进行内存恢复。

## 示例

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

如果 `audit --check` 仍然报告明文发现，请更新剩余的已报告目标路径并重新运行审计。

[沙盒 CLI](./sandbox.md)[安全](./security.md)