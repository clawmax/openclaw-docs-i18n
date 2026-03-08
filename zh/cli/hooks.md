

  CLI 命令

  
# hooks

管理代理钩子（针对 `/new`、`/reset` 和网关启动等命令的事件驱动自动化）。相关：

-   钩子：[Hooks](../automation/hooks.md)
-   插件钩子：[Plugins](../tools/plugin.md#plugin-hooks)

## 列出所有钩子

```bash
openclaw hooks list
```

列出从工作区、托管和捆绑目录中发现的所有钩子。**选项：**

-   `--eligible`: 仅显示符合条件的钩子（满足要求）
-   `--json`: 以 JSON 格式输出
-   `-v, --verbose`: 显示详细信息，包括缺失的要求

**示例输出：**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - 在网关启动时运行 BOOT.md
  📎 bootstrap-extra-files ✓ - 在代理引导期间注入额外的工作区引导文件
  📝 command-logger ✓ - 将所有命令事件记录到集中审计文件
  💾 session-memory ✓ - 当发出 /new 命令时将会话上下文保存到内存
```

**示例（详细模式）：**

```bash
openclaw hooks list --verbose
```

显示不符合条件钩子的缺失要求。**示例（JSON）：**

```bash
openclaw hooks list --json
```

返回结构化 JSON 以供编程使用。

## 获取钩子信息

```bash
openclaw hooks info <name>
```

显示特定钩子的详细信息。**参数：**

-   ``: 钩子名称（例如，`session-memory`）

**选项：**

-   `--json`: 以 JSON 格式输出

**示例：**

```bash
openclaw hooks info session-memory
```

**输出：**

```
💾 session-memory ✓ 就绪

当发出 /new 命令时将会话上下文保存到内存

详细信息：
  来源：openclaw-bundled
  路径：/path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  处理器：/path/to/openclaw/hooks/bundled/session-memory/handler.ts
  主页：https://docs.openclaw.ai/automation/hooks#session-memory
  事件：command:new

要求：
  配置：✓ workspace.dir
```

## 检查钩子资格

```bash
openclaw hooks check
```

显示钩子资格状态摘要（有多少已就绪 vs. 未就绪）。**选项：**

-   `--json`: 以 JSON 格式输出

**示例输出：**

```
钩子状态

钩子总数：4
就绪：4
未就绪：0
```

## 启用钩子

```bash
openclaw hooks enable <name>
```

通过将其添加到你的配置（`~/.openclaw/config.json`）来启用特定钩子。**注意：** 由插件管理的钩子在 `openclaw hooks list` 中显示为 `plugin:`，无法在此处启用/禁用。请改为启用/禁用插件。**参数：**

-   ``: 钩子名称（例如，`session-memory`）

**示例：**

```bash
openclaw hooks enable session-memory
```

**输出：**

```
✓ 已启用钩子：💾 session-memory
```

**功能说明：**

-   检查钩子是否存在且符合条件
-   在你的配置中更新 `hooks.internal.entries..enabled = true`
-   将配置保存到磁盘

**启用后：**

-   重启网关以使钩子重新加载（在 macOS 上重启菜单栏应用，或在开发中重启你的网关进程）。

## 禁用钩子

```bash
openclaw hooks disable <name>
```

通过更新你的配置来禁用特定钩子。**参数：**

-   ``: 钩子名称（例如，`command-logger`）

**示例：**

```bash
openclaw hooks disable command-logger
```

**输出：**

```
⏸ 已禁用钩子：📝 command-logger
```

**禁用后：**

-   重启网关以使钩子重新加载

## 安装钩子

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

从本地文件夹/存档或 npm 安装钩子包。Npm 规范是**仅限注册表**（包名 + 可选的**确切版本**或**分发标签**）。Git/URL/文件规范和 semver 范围会被拒绝。依赖安装会以 `--ignore-scripts` 运行以确保安全。裸规范和 `@latest` 保持在稳定轨道上。如果 npm 将其中任何一个解析为预发布版本，OpenClaw 会停止并要求你使用预发布标签（如 `@beta`/`@rc`）或确切的预发布版本明确选择加入。**功能说明：**

-   将钩子包复制到 `~/.openclaw/hooks/`
-   在 `hooks.internal.entries.*` 中启用已安装的钩子
-   在 `hooks.internal.installs` 下记录安装

**选项：**

-   `-l, --link`: 链接本地目录而不是复制（将其添加到 `hooks.internal.load.extraDirs`）
-   `--pin`: 将 npm 安装记录为 `hooks.internal.installs` 中确切解析的 `name@version`

**支持的存档格式：** `.zip`、`.tgz`、`.tar.gz`、`.tar` **示例：**

```bash
# 本地目录
openclaw hooks install ./my-hook-pack

# 本地存档
openclaw hooks install ./my-hook-pack.zip

# NPM 包
openclaw hooks install @openclaw/my-hook-pack

# 链接本地目录而不复制
openclaw hooks install -l ./my-hook-pack
```

## 更新钩子

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

更新已安装的钩子包（仅限 npm 安装）。**选项：**

-   `--all`: 更新所有已跟踪的钩子包
-   `--dry-run`: 显示将要更改的内容而不实际写入

当存储的完整性哈希存在且获取的工件哈希发生变化时，OpenClaw 会打印警告并在继续之前请求确认。在 CI/非交互式运行中使用全局 `--yes` 来绕过提示。

## 捆绑的钩子

### session-memory

当你发出 `/new` 命令时将会话上下文保存到内存。**启用：**

```bash
openclaw hooks enable session-memory
```

**输出：** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **参见：** [session-memory 文档](../automation/hooks.md#session-memory)

### bootstrap-extra-files

在 `agent:bootstrap` 期间注入额外的引导文件（例如 monorepo 本地的 `AGENTS.md` / `TOOLS.md`）。**启用：**

```bash
openclaw hooks enable bootstrap-extra-files
```

**参见：** [bootstrap-extra-files 文档](../automation/hooks.md#bootstrap-extra-files)

### command-logger

将所有命令事件记录到集中审计文件。**启用：**

```bash
openclaw hooks enable command-logger
```

**输出：** `~/.openclaw/logs/commands.log` **查看日志：**

```bash
# 最近命令
tail -n 20 ~/.openclaw/logs/commands.log

# 美化打印
cat ~/.openclaw/logs/commands.log | jq .

# 按操作过滤
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**参见：** [command-logger 文档](../automation/hooks.md#command-logger)

### boot-md

在网关启动时（通道启动后）运行 `BOOT.md`。**事件：** `gateway:startup` **启用：**

```bash
openclaw hooks enable boot-md
```

**参见：** [boot-md 文档](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)