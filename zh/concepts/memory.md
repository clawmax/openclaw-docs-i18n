

  会话与内存

  
# 内存

OpenClaw 内存是**智能体工作空间中的纯 Markdown 文件**。这些文件是唯一的事实来源；模型只“记住”写入磁盘的内容。内存搜索工具由活动的内存插件提供（默认：`memory-core`）。使用 `plugins.slots.memory = "none"` 可禁用内存插件。

## 内存文件 (Markdown)

默认的工作空间布局使用两层内存：

-   `memory/YYYY-MM-DD.md`
    -   每日日志（仅追加）。
    -   在会话开始时读取今天和昨天的日志。
-   `MEMORY.md` (可选)
    -   经过整理的长期记忆。
    -   **仅在主要的私有会话中加载**（绝不在群组上下文中加载）。

这些文件位于工作空间下（`agents.defaults.workspace`，默认为 `~/.openclaw/workspace`）。完整布局请参阅[智能体工作空间](./agent-workspace.md)。

## 内存工具

OpenClaw 为这些 Markdown 文件提供了两个面向智能体的工具：

-   `memory_search` — 对已索引的片段进行语义回忆。
-   `memory_get` — 定向读取特定的 Markdown 文件/行范围。

`memory_get` 现在**在文件不存在时会优雅降级**（例如，在第一次写入之前读取今天的每日日志）。内置管理器和 QMD 后端都会返回 `{ text: "", path }` 而不是抛出 `ENOENT` 错误，因此智能体可以处理“尚无记录”的情况，并在不将工具调用包装在 try/catch 逻辑中的情况下继续其工作流。

## 何时写入内存

-   决策、偏好和持久性事实写入 `MEMORY.md`。
-   日常笔记和运行上下文写入 `memory/YYYY-MM-DD.md`。
-   如果有人提到“记住这个”，请将其写下来（不要保存在 RAM 中）。
-   这个领域仍在发展中。提醒模型存储记忆会有所帮助；它会知道该怎么做。
-   如果你希望某些内容被记住，**请要求机器人将其写入**内存。

## 自动内存刷新（预压缩提醒）

当会话**接近自动压缩**时，OpenClaw 会触发一个**静默的、由智能体执行的轮次**，提醒模型在上下文被压缩**之前**写入持久性内存。默认提示明确说明模型*可以回复*，但通常 `NO_REPLY` 是正确的响应，因此用户永远不会看到这个轮次。这由 `agents.defaults.compaction.memoryFlush` 控制：

```json
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "会话即将压缩。现在存储持久性记忆。",
          prompt: "将任何持久的笔记写入 memory/YYYY-MM-DD.md；如果无需存储，请回复 NO_REPLY。",
        },
      },
    },
  },
}
```

详情：

-   **软阈值**：当会话令牌估计值超过 `contextWindow - reserveTokensFloor - softThresholdTokens` 时触发刷新。
-   默认**静默**：提示中包含 `NO_REPLY`，因此不会向用户传递任何内容。
-   **两个提示**：一个用户提示加上一个系统提示来附加提醒。
-   **每个压缩周期仅刷新一次**（在 `sessions.json` 中跟踪）。
-   **工作空间必须可写**：如果会话以 `workspaceAccess: "ro"` 或 `"none"` 在沙箱中运行，则跳过刷新。

完整的压缩生命周期，请参阅[会话管理 + 压缩](../reference/session-management-compaction.md)。

## 向量内存搜索

OpenClaw 可以在 `MEMORY.md` 和 `memory/*.md` 上构建一个小型向量索引，这样即使措辞不同，语义查询也能找到相关的笔记。默认设置：

-   默认启用。
-   监视内存文件的更改（防抖）。
-   在 `agents.defaults.memorySearch` 下配置内存搜索（而非顶层的 `memorySearch`）。
-   默认使用远程嵌入。如果未设置 `memorySearch.provider`，OpenClaw 会自动选择：
    1.  如果配置了 `memorySearch.local.modelPath` 且文件存在，则选择 `local`。
    2.  如果可以解析 OpenAI 密钥，则选择 `openai`。
    3.  如果可以解析 Gemini 密钥，则选择 `gemini`。
    4.  如果可以解析 Voyage 密钥，则选择 `voyage`。
    5.  如果可以解析 Mistral 密钥，则选择 `mistral`。
    6.  否则，在配置之前，内存搜索保持禁用状态。
-   本地模式使用 node-llama-cpp，可能需要 `pnpm approve-builds`。
-   使用 sqlite-vec（如果可用）来加速 SQLite 内的向量搜索。
-   也支持 `memorySearch.provider = "ollama"`，用于本地/自托管的 Ollama 嵌入（`/api/embeddings`），但它不会被自动选择。

远程嵌入**需要**嵌入提供商的 API 密钥。OpenClaw 从身份验证配置文件、`models.providers.*.apiKey` 或环境变量中解析密钥。Codex OAuth 仅涵盖聊天/补全，**不**满足内存搜索的嵌入需求。对于 Gemini，使用 `GEMINI_API_KEY` 或 `models.providers.google.apiKey`。对于 Voyage，使用 `VOYAGE_API_KEY` 或 `models.providers.voyage.apiKey`。对于 Mistral，使用 `MISTRAL_API_KEY` 或 `models.providers.mistral.apiKey`。Ollama 通常不需要真实的 API 密钥（当本地策略需要时，像 `OLLAMA_API_KEY=ollama-local` 这样的占位符即可）。使用自定义的 OpenAI 兼容端点时，设置 `memorySearch.remote.apiKey`（以及可选的 `memorySearch.remote.headers`）。

### QMD 后端（实验性）

设置 `memory.backend = "qmd"` 可以将内置的 SQLite 索引器替换为 [QMD](https://github.com/tobi/qmd)：一个本地优先的搜索辅助程序，结合了 BM25 + 向量 + 重排序。Markdown 仍然是事实来源；OpenClaw 通过调用 QMD 进行检索。关键点：**先决条件**

-   默认禁用。需要通过配置选择加入（`memory.backend = "qmd"`）。
-   单独安装 QMD CLI（`bun install -g https://github.com/tobi/qmd` 或下载发布版本），并确保 `qmd` 二进制文件在网关的 `PATH` 上。
-   QMD 需要一个允许扩展的 SQLite 构建（在 macOS 上使用 `brew install sqlite`）。
-   QMD 通过 Bun + `node-llama-cpp` 完全在本地运行，并在首次使用时自动从 HuggingFace 下载 GGUF 模型（无需单独的 Ollama 守护进程）。
-   网关通过设置 `XDG_CONFIG_HOME` 和 `XDG_CACHE_HOME`，在 `~/.openclaw/agents//qmd/` 下运行一个自包含的 XDG 主目录。
-   操作系统支持：一旦安装了 Bun + SQLite，macOS 和 Linux 即可开箱即用。Windows 最好通过 WSL2 支持。

**辅助程序如何运行**

-   网关在 `~/.openclaw/agents//qmd/` 下写入一个自包含的 QMD 主目录（配置 + 缓存 + sqlite 数据库）。
-   集合通过 `qmd collection add` 从 `memory.qmd.paths`（加上默认的工作空间内存文件）创建，然后在启动时和可配置的间隔（`memory.qmd.update.interval`，默认 5 分钟）运行 `qmd update` + `qmd embed`。
-   网关现在在启动时初始化 QMD 管理器，因此即使在第一次调用 `memory_search` 之前，也会启动定期更新计时器。
-   启动刷新现在默认在后台运行，因此聊天启动不会被阻塞；设置 `memory.qmd.update.waitForBootSync = true` 可以保持之前的阻塞行为。
-   搜索通过 `memory.qmd.searchMode` 运行（默认 `qmd search --json`；也支持 `vsearch` 和 `query`）。如果所选模式在你的 QMD 构建上拒绝标志，OpenClaw 会使用 `qmd query` 重试。如果 QMD 失败或二进制文件缺失，OpenClaw 会自动回退到内置的 SQLite 管理器，以便内存工具继续工作。
-   OpenClaw 目前不暴露 QMD 嵌入批量大小调优；批量行为由 QMD 自身控制。
-   **首次搜索可能较慢**：QMD 可能在第一次运行 `qmd query` 时下载本地 GGUF 模型（重排序器/查询扩展）。
    -   OpenClaw 在运行 QMD 时自动设置 `XDG_CONFIG_HOME`/`XDG_CACHE_HOME`。
    -   如果你想手动预下载模型（并预热 OpenClaw 使用的相同索引），请使用智能体的 XDG 目录运行一次性查询。OpenClaw 的 QMD 状态位于你的**状态目录**下（默认为 `~/.openclaw`）。你可以通过导出与 OpenClaw 相同的 XDG 变量，将 `qmd` 指向完全相同的索引：
        
        Copy
        
        ```bash
        # 选择与 OpenClaw 使用的相同状态目录
        STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
        
        export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
        export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"
        
        # (可选) 强制索引刷新 + 嵌入
        qmd update
        qmd embed
        
        # 预热 / 触发首次模型下载
        qmd query "test" -c memory-root --json >/dev/null 2>&1
        ```
        

**配置表面（`memory.qmd.*`）**

-   `command`（默认 `qmd`）：覆盖可执行文件路径。
-   `searchMode`（默认 `search`）：选择哪个 QMD 命令支持 `memory_search`（`search`、`vsearch`、`query`）。
-   `includeDefaultMemory`（默认 `true`）：自动索引 `MEMORY.md` + `memory/**/*.md`。
-   `paths[]`：添加额外的目录/文件（`path`，可选的 `pattern`，可选的稳定 `name`）。
-   `sessions`：选择加入会话 JSONL 索引（`enabled`、`retentionDays`、`exportDir`）。
-   `update`：控制刷新节奏和维护执行：（`interval`、`debounceMs`、`onBoot`、`waitForBootSync`、`embedInterval`、`commandTimeoutMs`、`updateTimeoutMs`、`embedTimeoutMs`）。
-   `limits`：限制召回负载（`maxResults`、`maxSnippetChars`、`maxInjectedChars`、`timeoutMs`）。
-   `scope`：与 [`session.sendPolicy`](../gateway/configuration.md#session) 相同的模式。默认为仅限私聊（`deny` 所有，`allow` 直接聊天）；放宽它以在群组/频道中显示 QMD 匹配结果。
    -   `match.keyPrefix` 匹配**规范化**的会话键（小写，去除任何前导的 `agent::`）。例如：`discord:channel:`。
    -   `match.rawKeyPrefix` 匹配**原始**会话键（小写），包括 `agent::`。例如：`agent:main:discord:`。
    -   遗留：`match.keyPrefix: "agent:..."` 仍被视为原始键前缀，但为了清晰起见，建议使用 `rawKeyPrefix`。
-   当 `scope` 拒绝搜索时，OpenClaw 会记录一个警告，包含派生的 `channel`/`chatType`，以便更容易调试空结果。
-   源自工作空间之外的片段在 `memory_search` 结果中显示为 `qmd//<relative-path>`；`memory_get` 理解此前缀并从配置的 QMD 集合根目录读取。
-   当 `memory.qmd.sessions.enabled = true` 时，OpenClaw 将经过清理的会话转录（用户/助手轮次）导出到 `~/.openclaw/agents//qmd/sessions/` 下的专用 QMD 集合中，这样 `memory_search` 就可以回忆最近的对话，而无需触及内置的 SQLite 索引。
-   当 `memory.citations` 为 `auto`/`on` 时，`memory_search` 片段现在包含一个 `来源：<路径#行号>` 页脚；设置 `memory.citations = "off"` 可以将路径元数据保留在内部（智能体仍会收到用于 `memory_get` 的路径，但片段文本省略页脚，并且系统提示会警告智能体不要引用它）。

**示例**

```
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // 规范化的会话键前缀（去除 `agent:<id>:`）。
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // 原始会话键前缀（包含 `agent:<id>:`）。
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**引用与回退**

-   `memory.citations` 无论后端如何都适用（`auto`/`on`/`off`）。
-   当 `qmd` 运行时，我们标记 `status().backend = "qmd"`，以便诊断显示哪个引擎提供了结果。如果 QMD 子进程退出或 JSON 输出无法解析，搜索管理器会记录警告并返回内置提供程序（现有的 Markdown 嵌入），直到 QMD 恢复。

### 额外内存路径

如果你想索引默认工作空间布局之外的 Markdown 文件，请添加显式路径：

```
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

注意：

-   路径可以是绝对路径或相对于工作空间的路径。
-   目录会递归扫描 `.md` 文件。
-   仅索引 Markdown 文件。
-   忽略符号链接（文件或目录）。

### Gemini 嵌入（原生）

将提供程序设置为 `gemini` 以直接使用 Gemini 嵌入 API：

```
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

注意：

-   `remote.baseUrl` 是可选的（默认为 Gemini API 基础 URL）。
-   `remote.headers` 允许你在需要时添加额外的头部。
-   默认模型：`gemini-embedding-001`。

如果你想使用**自定义的 OpenAI 兼容端点**（OpenRouter、vLLM 或代理），你可以使用 `remote` 配置与 OpenAI 提供程序：

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

如果你不想设置 API 密钥，请使用 `memorySearch.provider = "local"` 或设置 `memorySearch.fallback = "none"`。回退：

-   `memorySearch.fallback` 可以是 `openai`、`gemini`、`voyage`、`mistral`、`ollama`、`local` 或 `none`。
-   仅当主要嵌入提供程序失败时，才会使用回退提供程序。

批量索引（OpenAI + Gemini + Voyage）：

-   默认禁用。设置 `agents.defaults.memorySearch.remote.batch.enabled = true` 可为大型语料库索引启用（OpenAI、Gemini 和 Voyage）。
-   默认行为等待批量完成；如果需要，可以调整 `remote.batch.wait`、`remote.batch.pollIntervalMs` 和 `remote.batch.timeoutMinutes`。
-   设置 `remote.batch.concurrency` 以控制我们并行提交多少个批量作业（默认：2）。
-   批量模式在 `memorySearch.provider = "openai"` 或 `"gemini"` 时适用，并使用相应的 API 密钥。
-   Gemini 批量作业使用异步嵌入批量端点，并要求 Gemini 批量 API 可用。

为什么 OpenAI 批量又快又便宜：

-   对于大型回填，OpenAI 通常是我们支持的最快选项，因为我们可以将许多嵌入请求提交到单个批量作业中，并让 OpenAI 异步处理它们。
-   OpenAI 为批量 API 工作负载提供折扣定价，因此大型索引运行通常比同步发送相同请求更便宜。
-   详情请参阅 OpenAI 批量 API 文档和定价：
    -   [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
    -   [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

配置示例：

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

工具：

-   `memory_search` — 返回包含文件 + 行范围的片段。
-   `memory_get` — 按路径读取内存文件内容。

本地模式：

-   设置 `agents.defaults.memorySearch.provider = "local"`。
-   提供 `agents.defaults.memorySearch.local.modelPath`（GGUF 或 `hf:` URI）。
-   可选：设置 `agents.defaults.memorySearch.fallback = "none"` 以避免远程回退。

### 内存工具如何工作

-   `memory_search` 对来自 `MEMORY.md` + `memory/**/*.md` 的 Markdown 块（约 400 个令牌目标，80 个令牌重叠）进行语义搜索。它返回片段文本（限制约 700 字符）、文件路径、行范围、分数、提供程序/模型，以及我们是否从本地 → 远程嵌入回退。不返回完整的文件负载。
-   `memory_get` 读取特定的内存 Markdown 文件（相对于工作空间），可选地从起始行开始读取 N 行。`MEMORY.md` / `memory/` 之外的路径会被拒绝。
-   仅当 `memorySearch.enabled` 对智能体解析为 true 时，这两个工具才启用。

### 索引内容（及时间）

-   文件类型：仅 Markdown（`MEMORY.md`、`memory/**/*.md`）。
-   索引存储：每个智能体的 SQLite 位于 `~/.openclaw/memory/.sqlite`（可通过 `agents.defaults.memorySearch.store.path` 配置，支持 `{agentId}` 令牌）。
-   新鲜度：`MEMORY.md` + `memory/` 上的监视器将索引标记为脏（防抖 1.5 秒）。同步在会话开始时、搜索时或按间隔调度，并异步运行。会话转录使用增量阈值来触发后台同步。
-   重新索引触发器：索引存储嵌入**提供程序/模型 + 端点指纹 + 分块参数**。如果其中任何一项发生更改，OpenClaw 会自动重置并重新索引整个存储。

### 混合搜索 (BM25 + 向量)

启用后，OpenClaw 结合：

-   **向量相似性**（语义匹配，措辞可以不同）
-   **BM25 关键词相关性**（精确令牌，如 ID、环境变量、代码符号）

如果你的平台不支持全文搜索，OpenClaw 会回退到仅向量搜索。

#### 为什么需要混合？

向量搜索擅长处理“这意味着相同的事情”：

-   “Mac Studio 网关主机” vs “运行网关的机器”
-   “防抖文件更新” vs “避免每次写入都索引”

但它在处理精确、高信号令牌时可能较弱：

-   ID（`a828e60`、`b3b9895a…`）
-   代码符号（`memorySearch.query.hybrid`）
-   错误字符串（“sqlite-vec 不可用”）

BM25（全文）则相反：擅长精确令牌，不擅长释义。混合搜索是务实的折中方案：**同时使用两种检索信号**，这样无论是“自然语言”查询还是“大海捞针”查询，你都能获得良好的结果。

#### 我们如何合并结果（当前设计）

实现概述：

1.  从双方检索候选池：

-   **向量**：按余弦相似度取前 `maxResults * candidateMultiplier`。
-   **BM25**：按 FTS5 BM25 排名取前 `maxResults * candidateMultiplier`（越低越好）。

2.  将 BM25 排名转换为 0..1 左右的分数：

-   `textScore = 1 / (1 + max(0, bm25Rank))`

3.  按块 ID 合并候选并计算加权分数：

-   `finalScore = vectorWeight * vectorScore + textWeight * textScore`

注意：

-   `vectorWeight` + `textWeight` 在配置解析时归一化为 1.0，因此权重表现为百分比。
-   如果嵌入不可用（或提供程序返回零向量），我们仍然运行 BM25 并返回关键词匹配。
-   如果无法创建 FTS5，我们保持仅向量搜索（不会硬性失败）。

这并非“IR 理论上的完美”，但它简单、快速，并且倾向于提高真实笔记的召回率/精确率。如果我们以后想变得更复杂，常见的下一步是在混合之前进行倒数排名融合（RRF）或分数归一化（最小/最大或 z 分数）。

#### 后处理流水线

在合并向量和关键词分数之后，两个可选的后处理阶段在结果列表到达智能体之前对其进行细化：

```bash
向量 + 关键词 → 加权合并 → 时间衰减 → 排序 → MMR → Top-K 结果
```

这两个阶段**默认关闭**，可以独立启用。

#### MMR 重排序（多样性）

当混合搜索返回结果时，多个块可能包含相似或重叠的内容。例如，搜索“家庭网络设置”可能会从不同的每日笔记中返回五个几乎相同的片段，这些片段都提到了相同的路由器配置。**MMR（最大边际相关性）** 对结果进行重新排序，以平衡相关性与多样性，确保顶部结果涵盖查询的不同方面，而不是重复相同的信息。工作原理：

1.  结果按其原始相关性（向量 + BM25 加权分数）评分。
2.  MMR 迭代选择最大化以下公式的结果：`λ × 相关性 − (1−λ) × 与已选结果的最大相似度`。
3.  结果之间的相似度使用基于分词内容的 Jaccard 文本相似度来衡量。

`lambda` 参数控制权衡：

-   `lambda = 1.0` → 纯相关性（无多样性惩罚）
-   `lambda = 0.0` → 最大多样性（忽略相关性）
-   默认：`0.7`（平衡，略微偏向相关性）

**示例 — 查询：“家庭网络设置”** 给定这些内存文件：

```bash
memory/2026-02-10.md  → "配置了 Omada 路由器，为 IoT 设备设置 VLAN 10"
memory/2026-02-08.md  → "配置了 Omada 路由器，将 IoT 移至 VLAN 10"
memory/2026-02-05.md  → "在 192.168.10.2 上设置了 AdGuard DNS"
memory/network.md     → "路由器：Omada ER605，AdGuard：192.168.10.2，VLAN 10：IoT"
```

没有 MMR — 前 3 个结果：

```bash
1. memory/2026-02-10.md  (分数: 0.92)  ← 路由器 + VLAN
2. memory/2026-02-08.md  (分数: 0.89)  ← 路由器 + VLAN (近乎重复！)
3. memory/network.md     (分数: 0.85)  ← 参考文档
```

有 MMR (λ=0.7) — 前 3 个结果：

```bash
1. memory/2026-02-10.md  (分数: 0.92)  ← 路由器 + VLAN
2. memory/network.md     (分数: 0.85)  ← 参考文档 (多样化！)
3. memory/2026-02-05.md  (分数: 0.78)  ← AdGuard DNS (多样化！)
```

2月8日的近乎重复项被排除，智能体获得了三个不同的信息片段。**何时启用**：如果你注意到 `memory_search` 返回冗余或近乎重复的片段，尤其是跨天的每日笔记经常重复相似信息时。

#### 时间衰减（近期性提升）

拥有每日笔记的智能体会随着时间的推移积累数百个带日期的文件。没有衰减的情况下，六个月前措辞良好的笔记可能会在相同主题上胜过昨天的更新。**时间衰减**根据每个结果的年龄对分数应用指数乘数，因此最近的记忆自然排名更高，而旧的记忆逐渐淡出：

```bash
衰减后分数 = 分数 × e^(-λ × 天数)
```

其中 `λ = ln(2) / 半衰期天数`。使用默认的 30 天半衰期：

-   今天的笔记：原始分数的 **100%**
-   7 天前：**~84%**
-   30 天前：**50%**
-   90 天前：**12.5%**
-   180 天前：**~1.6%**

**常青文件永不衰减**：

-   `MEMORY.md`（根内存文件）
-   `memory/` 中的非日期文件（例如，`memory/projects.md`、`memory/network.md`）
-   这些文件包含持久的参考信息，应始终正常排名。

**带日期的每日文件**（`memory/YYYY-MM-DD.md`）使用从文件名中提取的日期。其他来源（例如，会话转录）回退到文件修改时间（`mtime`）。**示例 — 查询：“Rod 的工作时间表是什么？”** 给定这些内存文件（今天是 2月10日）：

```bash
memory/2025-09-15.md  → "Rod 工作时间为周一至周五，站会在上午 10 点，配对在下午 2 点"  (148 天前)
memory/2026-02-10.md  → "Rod 的站会在 14:15，与 Zeb 的 1:1 在 14:45"    (今天)
memory/2026-02-03.md  → "Rod 开始了新团队，站会移至 14:15"        (7 天前)
```

没有衰减：

```bash
1. memory/2025-09-15.md  (分数: 0.91)  ← 最佳语义匹配，但已过时！
2. memory/2026-02-10.md  (分数: 0.82)
3. memory/2026-02-03.md  (分数: 0.80)
```

有衰减（半衰期=30）：

```bash
1. memory/2026-02-10.md  (分数: 0.82 × 1.00 = 0.82)  ← 今天，无衰减
2. memory/2026-02-03.md  (分数: 0.80 × 0.85 = 0.68)  ← 7 天前，轻微衰减
3. memory/2025-09-15.md  (分数: 0.91 × 0.03 = 0.03)  ← 148 天前，几乎消失
```

尽管 9 月的笔记具有最佳的原始语义匹配，但它降到了底部。**何时启用**：如果你的智能体有数月的每日笔记，并且发现旧的、过时的信息比最近的上下文排名更高。对于每日笔记较多的工作流，30 天的半衰期效果很好；如果你经常参考较旧的笔记，可以增加它（例如，90 天）。

#### 配置

这两个功能都在 `memorySearch.query.hybrid` 下配置：

```
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4,
          // 多样性：减少冗余结果
          mmr: {
            enabled: true,    // 默认: false
            lambda: 0.7       // 0 = 最大多样性，1 = 最大相关性
          },
          // 近期性：提升较新的记忆
          temporalDecay: {
            enabled: true,    // 默认: false
            halfLifeDays: 30  // 分数每 30 天减半
          }
        }
      }
    }
  }
}
```

你可以独立启用任一功能：

-   **仅 MMR** — 当你有许多相似笔记但年龄不重要时有用。
-   **仅时间衰减** — 当近期性重要但你的结果已经多样化时有用。
-   **两者都启用** — 对于拥有大型、长期运行的每日笔记历史的智能体推荐。

### 嵌入缓存

OpenClaw 可以在 SQLite 中缓存**块嵌入**，以便重新索引和频繁更新（尤其是会话转录）不会重新嵌入未更改的文本。配置：

```
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

### 会话内存搜索（实验性）

你可以选择性地索引**会话转录**并通过 `memory_search` 呈现它们。这由一个实验性标志控制。

```
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

注意：

-   会话索引是**选择加入**的（默认关闭）。
-   会话更新会进行防抖，并在超过增量阈值后**异步索引**（尽力而为）。
-   `memory_search` 从不因索引而阻塞；结果可能略有延迟，直到后台同步完成。
-   结果仍然只包含片段；`memory_get` 仍然仅限于内存文件。
-   会话索引按智能体隔离（仅索引该智能体的会话日志）。
-   会话日志存储在磁盘上（`~/.openclaw/agents//sessions/*.jsonl`）。任何具有文件系统访问权限的进程/用户都可以读取它们，因此请将磁盘访问视为信任边界。为了更严格的隔离，请在单独的操作系统用户或主机下运行智能体。

增量阈值（显示默认值）：

```
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // JSONL 行数
        }
      }
    }
  }
}
```

### SQLite 向量加速 (sqlite-vec)

当 sqlite-vec 扩展可用时，OpenClaw 将嵌入存储在 SQLite 虚拟表（`vec0`）中，并在数据库中执行向量距离查询。这使得搜索保持快速，而无需将每个嵌入加载到 JS 中。配置（可选）：

```
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

注意：

-   `enabled` 默认为 true；禁用时，搜索回退到对存储的嵌入进行进程内余弦相似度计算。
-   如果 sqlite-vec 扩展缺失或加载失败，OpenClaw 会记录错误并继续使用 JS 回退（无向量表）。
-   `extensionPath` 覆盖捆绑的 sqlite-vec 路径（对于自定义构建或非标准安装位置很有用）。

### 本地嵌入自动下载

-   默认本地嵌入模型：`hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF/embeddinggemma-300m-qat-Q8_0.gguf`（约 0.6 GB）。
-   当 `memorySearch.provider = "local"` 时，`node-llama-cpp` 解析 `modelPath`；如果 GGUF 缺失，它会**自动下载**到缓存（或如果设置了 `local.modelCacheDir`），然后加载它。下载会在重试时恢复。
-   原生构建要求：运行 `pnpm approve-builds`，选择 `node-llama-cpp`，然后运行 `pnpm rebuild node-llama-cpp`。
-   回退：如果本地设置失败且 `memorySearch.fallback = "openai"`，我们会自动切换到远程嵌入（`openai/text-embedding-3-small`，除非被覆盖）并记录原因。

### 自定义 OpenAI 兼容端点示例

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

注意：

-   `remote.*` 优先于 `models.providers.openai.*`。
-   `remote.headers` 与 OpenAI 头部合并；远程在键冲突时胜出。省略 `remote.headers` 以使用 OpenAI 默认值。

[会话工具](./session-tool.md)[压缩](./compaction.md)