

  配置

  
# 模型故障转移

OpenClaw 分两个阶段处理故障：

1.  在当前提供商内部进行**认证配置文件轮换**。
2.  **模型回退**到 `agents.defaults.model.fallbacks` 中的下一个模型。

本文档解释了运行时规则及其背后的数据。

## 认证存储（密钥 + OAuth）

OpenClaw 使用**认证配置文件**来管理 API 密钥和 OAuth 令牌。

-   密钥存储在 `~/.openclaw/agents//agent/auth-profiles.json`（旧版：`~/.openclaw/agent/auth-profiles.json`）。
-   配置中的 `auth.profiles` / `auth.order` 仅用于**元数据和路由**（不包含密钥）。
-   旧版仅用于导入的 OAuth 文件：`~/.openclaw/credentials/oauth.json`（首次使用时导入到 `auth-profiles.json`）。

更多详情：[/concepts/oauth](./oauth.md) 凭证类型：

-   `type: "api_key"` → `{ provider, key }`
-   `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（某些提供商还包含 `projectId`/`enterpriseUrl`）

## 配置文件 ID

OAuth 登录会创建独立的配置文件，以便多个账户可以共存。

-   默认：当没有可用邮箱时，使用 `provider:default`。
-   带邮箱的 OAuth：`provider:`（例如 `google-antigravity:user@gmail.com`）。

配置文件位于 `~/.openclaw/agents//agent/auth-profiles.json` 的 `profiles` 下。

## 轮换顺序

当一个提供商有多个配置文件时，OpenClaw 按以下顺序选择：

1.  **显式配置**：`auth.order[provider]`（如果已设置）。
2.  **已配置的配置文件**：按提供商筛选的 `auth.profiles`。
3.  **存储的配置文件**：`auth-profiles.json` 中该提供商的条目。

如果未配置显式顺序，OpenClaw 使用轮询顺序：

-   **主键：** 配置文件类型（**OAuth 优先于 API 密钥**）。
-   **次键：** `usageStats.lastUsed`（在每个类型内，最久未使用的优先）。
-   **处于冷却/禁用状态的配置文件** 被移到末尾，按最早到期时间排序。

### 会话粘性（缓存友好）

OpenClaw **将选定的认证配置文件固定到每个会话**，以保持提供商缓存的热度。它**不会**在每个请求上都进行轮换。固定的配置文件会被重复使用，直到：

-   会话被重置（`/new` / `/reset`）
-   一次压缩完成（压缩计数递增）
-   配置文件处于冷却/禁用状态

通过 `/model …@` 进行手动选择会为该会话设置一个**用户覆盖**，并且在会话结束前不会自动轮换。自动固定的配置文件（由会话路由器选择）被视为一种**偏好**：它们会首先被尝试，但 OpenClaw 可能会在遇到速率限制/超时时轮换到其他配置文件。用户固定的配置文件则保持锁定到该配置文件；如果它失败且配置了模型回退，OpenClaw 将移动到下一个模型，而不是切换配置文件。

### 为什么 OAuth 可能“看起来丢失”

如果你为同一个提供商同时拥有 OAuth 配置文件和 API 密钥配置文件，除非被固定，否则轮询可能会在不同消息之间切换它们。要强制使用单一配置文件：

-   使用 `auth.order[provider] = ["provider:profileId"]` 进行固定，或者
-   通过 `/model …` 使用配置文件覆盖进行每个会话的覆盖（当你的 UI/聊天界面支持时）。

## 冷却时间

当配置文件因认证/速率限制错误（或看起来像速率限制的超时）而失败时，OpenClaw 会将其标记为冷却状态并移动到下一个配置文件。格式/无效请求错误（例如 Cloud Code Assist 工具调用 ID 验证失败）被视为需要故障转移的错误，并使用相同的冷却时间。OpenAI 兼容的停止原因错误，如 `Unhandled stop reason: error`、`stop reason: error` 和 `reason: error`，被归类为超时/故障转移信号。冷却时间使用指数退避：

-   1 分钟
-   5 分钟
-   25 分钟
-   1 小时（上限）

状态存储在 `auth-profiles.json` 的 `usageStats` 下：

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## 禁用计费

计费/信用失败（例如“信用不足”/“信用余额过低”）被视为需要故障转移的错误，但它们通常不是暂时性的。OpenClaw 不会使用短冷却时间，而是将配置文件标记为**禁用**（使用更长的退避时间），并轮换到下一个配置文件/提供商。状态存储在 `auth-profiles.json` 中：

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

默认值：

-   计费退避从 **5 小时** 开始，每次计费失败后加倍，上限为 **24 小时**。
-   如果配置文件 **24 小时** 内未发生失败，则退避计数器重置（可配置）。

## 模型回退

如果一个提供商的所有配置文件都失败，OpenClaw 将移动到 `agents.defaults.model.fallbacks` 中的下一个模型。这适用于认证失败、速率限制以及耗尽了配置文件轮换的超时（其他错误不会触发回退）。当运行开始时带有模型覆盖（通过钩子或 CLI），在尝试任何已配置的回退后，回退仍会在 `agents.defaults.model.primary` 处结束。

## 相关配置

有关以下配置，请参阅 [网关配置](../gateway/configuration.md)：

-   `auth.profiles` / `auth.order`
-   `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
-   `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
-   `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel` 路由

有关更广泛的模型选择和回退概述，请参阅 [模型](./models.md)。

[模型提供商](./model-providers.md)[Anthropic](../providers/anthropic.md)