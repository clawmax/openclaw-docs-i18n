

  配置与操作

  
# Secrets Apply 计划合约

本页面定义了 `openclaw secrets apply` 强制执行的严格合约。如果目标不符合这些规则，apply 将在修改配置之前失败。

## 计划文件结构

`openclaw secrets apply --from <plan.json>` 期望一个包含计划目标的 `targets` 数组：

```json
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
    {
      type: "auth-profiles.api_key.key",
      path: "profiles.openai:default.key",
      pathSegments: ["profiles", "openai:default", "key"],
      agentId: "main",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## 支持的目标范围

计划目标适用于以下受支持的凭证路径：

-   [SecretRef 凭证表面](../reference/secretref-credential-surface.md)

## 目标类型行为

通用规则：

-   `target.type` 必须被识别，并且必须与规范化的 `target.path` 形状匹配。

兼容性别名对现有计划仍然接受：

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## 路径验证规则

每个目标都会通过以下所有规则进行验证：

-   `type` 必须是已识别的目标类型。
-   `path` 必须是非空的点分隔路径。
-   `pathSegments` 可以省略。如果提供，它必须规范化为与 `path` 完全相同的路径。
-   禁止的片段会被拒绝：`__proto__`、`prototype`、`constructor`。
-   规范化后的路径必须与目标类型注册的路径形状匹配。
-   如果设置了 `providerId` 或 `accountId`，它必须与路径中编码的 id 匹配。
-   `auth-profiles.json` 目标需要 `agentId`。
-   创建新的 `auth-profiles.json` 映射时，需包含 `authProfileProvider`。

## 失败行为

如果目标验证失败，apply 会以类似错误退出：

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

对于无效的计划，不会提交任何写入操作。

## 运行时与审计范围说明

-   仅引用类型的 `auth-profiles.json` 条目（`keyRef`/`tokenRef`）包含在运行时解析和审计覆盖范围内。
-   `secrets apply` 写入受支持的 `openclaw.json` 目标、受支持的 `auth-profiles.json` 目标以及可选的清理目标。

## 操作员检查

```bash
# 验证计划而不写入
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# 然后实际应用
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

如果 apply 因目标路径无效消息而失败，请使用 `openclaw secrets configure` 重新生成计划，或将目标路径修复为上述支持的形状。

## 相关文档

-   [密钥管理](./secrets.md)
-   [CLI `secrets`](../cli/secrets.md)
-   [SecretRef 凭证表面](../reference/secretref-credential-surface.md)
-   [配置参考](./configuration-reference.md)

[密钥管理](./secrets.md)[可信代理认证](./trusted-proxy-auth.md)