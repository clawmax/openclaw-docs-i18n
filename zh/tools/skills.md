

  技能

  
# 技能

OpenClaw 使用 **[AgentSkills](https://agentskills.io) 兼容的** 技能文件夹来教导代理如何使用工具。每个技能是一个目录，其中包含一个带有 YAML 前置元数据和说明的 `SKILL.md` 文件。OpenClaw 加载**捆绑技能**以及可选的本地覆盖，并根据环境、配置和二进制文件的存在性在加载时对它们进行过滤。

## 位置与优先级

技能从**三个**位置加载：

1.  **捆绑技能**：随安装包（npm 包或 OpenClaw.app）一起提供。
2.  **托管/本地技能**：`~/.openclaw/skills`
3.  **工作区技能**：`/skills`

如果技能名称冲突，优先级为：`/skills`（最高）→ `~/.openclaw/skills` → 捆绑技能（最低）。此外，您可以通过 `~/.openclaw/openclaw.json` 中的 `skills.load.extraDirs` 配置额外的技能文件夹（最低优先级）。

## 每个代理专用技能与共享技能

在**多代理**设置中，每个代理都有自己的工作区。这意味着：

-   **每个代理专用技能**位于该代理的 `/skills` 目录中，仅对该代理可见。
-   **共享技能**位于 `~/.openclaw/skills`（托管/本地）中，对同一台机器上的**所有代理**可见。
-   **共享文件夹**也可以通过 `skills.load.extraDirs` 添加（最低优先级），如果您希望多个代理使用一个公共的技能包。

如果同一技能名称出现在多个位置，则应用通常的优先级规则：工作区优先，然后是托管/本地，最后是捆绑技能。

## 插件 + 技能

插件可以通过在 `openclaw.plugin.json` 中列出 `skills` 目录（路径相对于插件根目录）来提供自己的技能。插件技能在插件启用时加载，并遵循正常的技能优先级规则。您可以通过插件配置条目上的 `metadata.openclaw.requires.config` 对它们进行门控。有关发现/配置，请参阅[插件](./plugin.md)；有关这些技能所教授的工具界面，请参阅[工具](../tools.md)。

## ClawHub（安装 + 同步）

ClawHub 是 OpenClaw 的公共技能注册中心。在 [https://clawhub.com](https://clawhub.com) 浏览。使用它来发现、安装、更新和备份技能。完整指南：[ClawHub](./clawhub.md)。常见流程：

-   将技能安装到您的工作区：
    -   `clawhub install <skill-slug>`
-   更新所有已安装的技能：
    -   `clawhub update --all`
-   同步（扫描 + 发布更新）：
    -   `clawhub sync --all`

默认情况下，`clawhub` 会安装到您当前工作目录下的 `./skills` 目录（或回退到配置的 OpenClaw 工作区）。OpenClaw 将在下一次会话中将其作为 `/skills` 拾取。

## 安全注意事项

-   将第三方技能视为**不受信任的代码**。在启用前请仔细阅读。
-   对于不受信任的输入和风险工具，优先使用沙盒运行。请参阅[沙盒化](../gateway/sandboxing.md)。
-   工作区和额外目录的技能发现仅接受解析后的真实路径保持在配置根目录内的技能根目录和 `SKILL.md` 文件。
-   `skills.entries.*.env` 和 `skills.entries.*.apiKey` 将密钥注入到该代理轮次的**主机**进程中（而非沙盒）。请勿将密钥放入提示词和日志中。
-   有关更广泛的威胁模型和检查清单，请参阅[安全](../gateway/security.md)。

## 格式（AgentSkills + Pi 兼容）

`SKILL.md` 必须至少包含：

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

注意事项：

-   我们遵循 AgentSkills 规范进行布局/意图定义。
-   嵌入式代理使用的解析器仅支持**单行**前置元数据键。
-   `metadata` 应该是一个**单行 JSON 对象**。
-   在说明中使用 `{baseDir}` 来引用技能文件夹路径。
-   可选的前置元数据键：
    -   `homepage` — 在 macOS 技能 UI 中作为“网站”显示的 URL（也支持通过 `metadata.openclaw.homepage` 设置）。
    -   `user-invocable` — `true|false`（默认：`true`）。当为 `true` 时，该技能作为用户斜杠命令公开。
    -   `disable-model-invocation` — `true|false`（默认：`false`）。当为 `true` 时，该技能从模型提示词中排除（仍可通过用户调用使用）。
    -   `command-dispatch` — `tool`（可选）。当设置为 `tool` 时，斜杠命令绕过模型并直接分派给工具。
    -   `command-tool` — 当设置 `command-dispatch: tool` 时要调用的工具名称。
    -   `command-arg-mode` — `raw`（默认）。对于工具分派，将原始参数字符串转发给工具（不进行核心解析）。工具调用时使用的参数为：`{ command: "", commandName: "", skillName: "" }`。

## 门控（加载时过滤器）

OpenClaw **在加载时过滤技能**，使用 `metadata`（单行 JSON）：

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

`metadata.openclaw` 下的字段：

-   `always: true` — 始终包含该技能（跳过其他门控条件）。
-   `emoji` — macOS 技能 UI 使用的可选表情符号。
-   `homepage` — 在 macOS 技能 UI 中显示为“网站”的可选 URL。
-   `os` — 可选平台列表（`darwin`、`linux`、`win32`）。如果设置，该技能仅在这些操作系统上有资格使用。
-   `requires.bins` — 列表；每个必须在 `PATH` 中存在。
-   `requires.anyBins` — 列表；至少有一个必须在 `PATH` 中存在。
-   `requires.env` — 列表；环境变量必须存在**或**在配置中提供。
-   `requires.config` — 必须为真值的 `openclaw.json` 路径列表。
-   `primaryEnv` — 与 `skills.entries..apiKey` 关联的环境变量名称。
-   `install` — macOS 技能 UI 使用的可选安装程序规范数组（brew/node/go/uv/download）。

关于沙盒化的注意事项：

-   `requires.bins` 在技能加载时在**主机**上检查。
-   如果代理在沙盒中运行，二进制文件也必须**存在于容器内**。通过 `agents.defaults.sandbox.docker.setupCommand`（或自定义镜像）安装它。`setupCommand` 在容器创建后运行一次。软件包安装还需要网络出口、可写的根文件系统以及沙盒中的 root 用户。例如：`summarize` 技能（`skills/summarize/SKILL.md`）需要在沙盒容器中存在 `summarize` CLI 才能在那里运行。

安装程序示例：

```
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

注意事项：

-   如果列出了多个安装程序，网关会选择一个**单一**的首选选项（brew 可用时优先，否则 node）。
-   如果所有安装程序都是 `download`，OpenClaw 会列出每个条目，以便您查看可用的工件。
-   安装程序规范可以包含 `os: ["darwin"|"linux"|"win32"]` 以按平台过滤选项。
-   Node 安装遵循 `openclaw.json` 中的 `skills.install.nodeManager`（默认：npm；选项：npm/pnpm/yarn/bun）。这仅影响**技能安装**；网关运行时仍应为 Node（不推荐将 Bun 用于 WhatsApp/Telegram）。
-   Go 安装：如果缺少 `go` 但 `brew` 可用，网关会首先通过 Homebrew 安装 Go，并在可能时将 `GOBIN` 设置为 Homebrew 的 `bin`。
-   下载安装：`url`（必需）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（默认：检测到存档时自动）、`stripComponents`、`targetDir`（默认：`~/.openclaw/tools/`）。

如果不存在 `metadata.openclaw`，则该技能始终有资格使用（除非在配置中被禁用，或者对于捆绑技能被 `skills.allowBundled` 阻止）。

## 配置覆盖 (~/.openclaw/openclaw.json)

可以切换捆绑/托管技能并提供环境变量值：

```json
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // 或纯文本字符串
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

注意：如果技能名称包含连字符，请引用键名（JSON5 允许带引号的键）。配置键默认匹配**技能名称**。如果技能定义了 `metadata.openclaw.skillKey`，则在 `skills.entries` 下使用该键。规则：

-   `enabled: false` 会禁用该技能，即使它是捆绑/已安装的。
-   `env`：仅当该变量尚未在进程中设置时才注入。
-   `apiKey`：为声明了 `metadata.openclaw.primaryEnv` 的技能提供便利。支持纯文本字符串或 SecretRef 对象（`{ source, provider, id }`）。
-   `config`：用于自定义每个技能字段的可选包；自定义键必须放在此处。
-   `allowBundled`：仅针对**捆绑**技能的可选允许列表。如果设置，则只有列表中的捆绑技能才有资格（托管/工作区技能不受影响）。

## 环境变量注入（每个代理运行）

当代理运行时开始时，OpenClaw：

1.  读取技能元数据。
2.  将任何 `skills.entries..env` 或 `skills.entries..apiKey` 应用到 `process.env`。
3.  使用**有资格的**技能构建系统提示词。
4.  在运行结束后恢复原始环境。

这是**限定在代理运行范围内的**，而不是全局的 shell 环境。

## 会话快照（性能）

OpenClaw 在**会话开始时**对有资格的技能进行快照，并在同一会话的后续轮次中重用该列表。对技能或配置的更改将在下一次新会话中生效。当技能监视器启用或出现新的有资格的远程节点时（见下文），技能也可以在会话中刷新。可以将其视为**热重载**：刷新后的列表将在下一个代理轮次中被拾取。

## 远程 macOS 节点（Linux 网关）

如果网关运行在 Linux 上，但有一个 **macOS 节点**已连接**并且允许 `system.run`**（执行批准安全性未设置为 `deny`），那么当所需二进制文件存在于该节点上时，OpenClaw 可以将仅限 macOS 的技能视为有资格使用。代理应通过 `nodes` 工具（通常是 `nodes.run`）执行这些技能。这依赖于节点报告其命令支持以及通过 `system.run` 进行的二进制探测。如果 macOS 节点后来离线，技能仍然可见；调用可能会失败，直到节点重新连接。

## 技能监视器（自动刷新）

默认情况下，OpenClaw 监视技能文件夹，并在 `SKILL.md` 文件更改时更新技能快照。在 `skills.load` 下配置此功能：

```json
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## 令牌影响（技能列表）

当技能有资格时，OpenClaw 会将一个紧凑的可用技能 XML 列表注入到系统提示词中（通过 `pi-coding-agent` 中的 `formatSkillsForPrompt`）。成本是确定性的：

-   **基础开销（仅当 ≥1 个技能时）：** 195 个字符。
-   **每个技能：** 97 个字符 + XML 转义的 ``、`` 和 `` 值的长度。

公式（字符数）：

```bash
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

注意事项：

-   XML 转义会将 `& < > " '` 扩展为实体（`&amp;`、`<` 等），从而增加长度。
-   令牌计数因模型分词器而异。粗略的 OpenAI 风格估计约为 4 个字符/令牌，因此**97 个字符 ≈ 24 个令牌** 每个技能，再加上您实际字段的长度。

## 托管技能生命周期

OpenClaw 作为安装包（npm 包或 OpenClaw.app）的一部分，提供一组基线技能作为**捆绑技能**。`~/.openclaw/skills` 用于本地覆盖（例如，在不更改捆绑副本的情况下固定/修补技能）。工作区技能为用户所有，并在名称冲突时覆盖前两者。

## 配置参考

有关完整的配置模式，请参阅[技能配置](./skills-config.md)。

## 寻找更多技能？

浏览 [https://clawhub.com](https://clawhub.com)。

* * *

[斜杠命令](./slash-commands.md)[技能配置](./skills-config.md)