

  环境与调试

  
# 环境变量

OpenClaw 从多个来源拉取环境变量。其规则是**绝不覆盖现有值**。

## 优先级（从高到低）

1.  **进程环境**（Gateway 进程从其父 Shell/守护进程已继承的环境）。
2.  **当前工作目录中的 `.env` 文件**（dotenv 默认行为；不覆盖）。
3.  **全局 `.env` 文件**，位于 `~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`；不覆盖）。
4.  **配置文件中的 `env` 块**，位于 `~/.openclaw/openclaw.json`（仅对缺失的键生效）。
5.  **可选的登录 Shell 导入**（通过 `env.shellEnv.enabled` 或 `OPENCLAW_LOAD_SHELL_ENV=1` 启用），仅对缺失的预期键生效。

如果配置文件完全缺失，则跳过第 4 步；若已启用，Shell 导入仍会运行。

## 配置中的 env 块

有两种等效的方式来设置内联环境变量（两者均为非覆盖式）：

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Shell 环境导入

`env.shellEnv` 会运行您的登录 Shell 并仅导入**缺失的**预期键：

```json
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

等效的环境变量：

-   `OPENCLAW_LOAD_SHELL_ENV=1`
-   `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## 运行时注入的环境变量

OpenClaw 还会向生成的子进程中注入上下文标记：

-   `OPENCLAW_SHELL=exec`：为通过 `exec` 工具运行的命令设置。
-   `OPENCLAW_SHELL=acp`：为 ACP 运行时后端进程生成（例如 `acpx`）设置。
-   `OPENCLAW_SHELL=acp-client`：当 `openclaw acp client` 生成 ACP 桥接进程时设置。
-   `OPENCLAW_SHELL=tui-local`：为本地 TUI `!` Shell 命令设置。

这些是运行时标记（非必需的用户配置）。它们可用于 Shell/配置文件逻辑中以应用特定于上下文的规则。

## 配置中的环境变量替换

您可以使用 `${VAR_NAME}` 语法直接在配置字符串值中引用环境变量：

```json
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

完整详情请参阅[配置：环境变量替换](../gateway/configuration.md#env-var-substitution-in-config)。

## Secret 引用 vs `\${ENV}` 字符串

OpenClaw 支持两种环境变量驱动的模式：

-   在配置值中使用 `${VAR}` 字符串替换。
-   对于支持密钥引用的字段，使用 SecretRef 对象（`{ source: "env", provider: "default", id: "VAR" }`）。

两者均在激活时从进程环境中解析。SecretRef 的详细信息记录在[密钥管理](../gateway/secrets.md)中。

## 路径相关的环境变量

| 变量 | 用途 |
| --- | --- |
| `OPENCLAW_HOME` | 覆盖用于所有内部路径解析的主目录（`~/.openclaw/`、代理目录、会话、凭据）。在以专用服务用户身份运行 OpenClaw 时很有用。 |
| `OPENCLAW_STATE_DIR` | 覆盖状态目录（默认为 `~/.openclaw`）。 |
| `OPENCLAW_CONFIG_PATH` | 覆盖配置文件路径（默认为 `~/.openclaw/openclaw.json`）。 |

## 日志

| 变量 | 用途 |
| --- | --- |
| `OPENCLAW_LOG_LEVEL` | 覆盖文件和控制台的日志级别（例如 `debug`、`trace`）。优先级高于配置中的 `logging.level` 和 `logging.consoleLevel`。无效值将被忽略并发出警告。 |

### OPENCLAW\_HOME

当设置时，`OPENCLAW_HOME` 将替换系统主目录（`$HOME` / `os.homedir()`）用于所有内部路径解析。这使得无头服务账户可以实现完整的文件系统隔离。**优先级：** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()` **示例**（macOS LaunchDaemon）：

```
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` 也可以设置为波浪号路径（例如 `~/svc`），该路径会在使用前使用 `$HOME` 进行扩展。

## 相关链接

-   [Gateway 配置](../gateway/configuration.md)
-   [FAQ：环境变量和 .env 加载](./faq.md#env-vars-and-env-loading)
-   [模型概述](../concepts/models.md)

[OpenClaw 背景知识](../start/lore.md)[调试](./debugging.md)