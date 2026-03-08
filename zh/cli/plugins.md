

  CLI 命令

  
# plugins

管理网关插件/扩展（在进程中加载）。相关：

-   插件系统：[插件](../tools/plugin.md)
-   插件清单 + 模式：[插件清单](../plugins/manifest.md)
-   安全加固：[安全性](../gateway/security.md)

## 命令

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

捆绑插件随 OpenClaw 一起提供，但初始状态为禁用。使用 `plugins enable` 来激活它们。所有插件都必须提供一个包含内联 JSON 模式（`configSchema`，即使为空）的 `openclaw.plugin.json` 文件。缺失或无效的清单或模式将阻止插件加载并导致配置验证失败。

### 安装

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

安全提示：将插件安装视为运行代码。建议使用固定版本。Npm 规范**仅限注册表**（包名 + 可选的**精确版本**或**分发标签**）。Git/URL/文件规范和 semver 范围将被拒绝。依赖安装使用 `--ignore-scripts` 以确保安全。裸规范和 `@latest` 保持在稳定轨道上。如果 npm 将其中任何一个解析为预发布版本，OpenClaw 将停止并提示您使用预发布标签（如 `@beta`/`@rc`）或精确的预发布版本（如 `@1.2.3-beta.4`）明确选择加入。如果裸安装规范匹配捆绑插件的 ID（例如 `diffs`），OpenClaw 将直接安装该捆绑插件。要安装同名的 npm 包，请使用显式的作用域规范（例如 `@scope/diffs`）。支持的归档格式：`.zip`、`.tgz`、`.tar.gz`、`.tar`。使用 `--link` 可避免复制本地目录（添加到 `plugins.load.paths`）：

```bash
openclaw plugins install -l ./my-plugin
```

在 npm 安装时使用 `--pin`，将解析出的精确规范（`name@version`）保存到 `plugins.installs` 中，同时保持默认的非固定行为。

### 卸载

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` 从 `plugins.entries`、`plugins.installs`、插件允许列表以及适用的链接 `plugins.load.paths` 条目中移除插件记录。对于活动的内存插件，内存槽将重置为 `memory-core`。默认情况下，卸载还会删除活动状态目录扩展根目录（`$OPENCLAW_STATE_DIR/extensions/`）下的插件安装目录。使用 `--keep-files` 可保留磁盘上的文件。`--keep-config` 作为 `--keep-files` 的已弃用别名被支持。

### 更新

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新仅适用于从 npm 安装的插件（在 `plugins.installs` 中跟踪）。当存在存储的完整性哈希且获取的工件哈希发生变化时，OpenClaw 会打印警告并在继续之前请求确认。在 CI/非交互式运行中，使用全局 `--yes` 来绕过提示。

[pairing](./pairing.md)[qr](./qr.md)