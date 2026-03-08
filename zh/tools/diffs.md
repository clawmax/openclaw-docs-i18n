

  内置工具

  
# 差异工具

`diffs` 是一个可选的插件工具，具有简短的内置系统指导和一个配套技能，可将变更内容转换为供代理使用的只读差异工件。它接受以下任一输入：

-   `before` 和 `after` 文本
-   统一的 `patch`（补丁）

它可以返回：

-   用于画布展示的网关查看器 URL
-   用于消息传递的渲染文件路径（PNG 或 PDF）
-   在一次调用中同时输出两者

启用后，该插件会将简洁的使用指导前置到系统提示空间，并同时暴露一个详细的技能，供代理需要完整指令时使用。

## 快速开始

1.  启用插件。
2.  使用 `mode: "view"` 调用 `diffs`，适用于画布优先的流程。
3.  使用 `mode: "file"` 调用 `diffs`，适用于聊天文件传递流程。
4.  使用 `mode: "both"` 调用 `diffs`，当你需要两种工件时。

## 启用插件

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## 禁用内置系统指导

如果你想保持 `diffs` 工具启用但禁用其内置的系统提示指导，将 `plugins.entries.diffs.hooks.allowPromptInjection` 设置为 `false`：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

这会阻止 diffs 插件的 `before_prompt_build` 钩子，同时保持插件、工具和配套技能可用。如果你想同时禁用指导和工具，请直接禁用插件。

## 典型代理工作流

1.  代理调用 `diffs`。
2.  代理读取 `details` 字段。
3.  代理执行以下操作之一：
    -   使用 `canvas present` 打开 `details.viewerUrl`
    -   使用 `message` 发送 `details.filePath`（通过 `path` 或 `filePath`）
    -   两者都执行

## 输入示例

Before 和 after 模式：

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

Patch 模式：

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## 工具输入参考

除非注明，否则所有字段都是可选的：

-   `before` (`string`): 原始文本。当省略 `patch` 时，与 `after` 一起为必需。
-   `after` (`string`): 更新后的文本。当省略 `patch` 时，与 `before` 一起为必需。
-   `patch` (`string`): 统一差异文本。与 `before` 和 `after` 互斥。
-   `path` (`string`): before 和 after 模式下的显示文件名。
-   `lang` (`string`): before 和 after 模式下的语言覆盖提示。
-   `title` (`string`): 查看器标题覆盖。
-   `mode` (`"view" | "file" | "both"`): 输出模式。默认为插件默认值 `defaults.mode`。
-   `theme` (`"light" | "dark"`): 查看器主题。默认为插件默认值 `defaults.theme`。
-   `layout` (`"unified" | "split"`): 差异布局。默认为插件默认值 `defaults.layout`。
-   `expandUnchanged` (`boolean`): 当有完整上下文可用时，展开未更改的部分。仅限每次调用选项（不是插件默认键）。
-   `fileFormat` (`"png" | "pdf"`): 渲染文件格式。默认为插件默认值 `defaults.fileFormat`。
-   `fileQuality` (`"standard" | "hq" | "print"`): PNG 或 PDF 渲染的质量预设。
-   `fileScale` (`number`): 设备缩放覆盖 (`1`\-`4`)。
-   `fileMaxWidth` (`number`): 最大渲染宽度（CSS 像素）(`640`\-`2400`)。
-   `ttlSeconds` (`number`): 查看器工件的 TTL（生存时间），单位秒。默认 1800，最大 21600。
-   `baseUrl` (`string`): 查看器 URL 来源覆盖。必须是 `http` 或 `https`，无查询/哈希。

验证和限制：

-   `before` 和 `after` 各自最大 512 KiB。
-   `patch` 最大 2 MiB。
-   `path` 最大 2048 字节。
-   `lang` 最大 128 字节。
-   `title` 最大 1024 字节。
-   补丁复杂度上限：最多 128 个文件和 120000 总行数。
-   同时提供 `patch` 和 `before` 或 `after` 会被拒绝。
-   渲染文件安全限制（适用于 PNG 和 PDF）：
    -   `fileQuality: "standard"`: 最大 8 MP（8,000,000 渲染像素）。
    -   `fileQuality: "hq"`: 最大 14 MP（14,000,000 渲染像素）。
    -   `fileQuality: "print"`: 最大 24 MP（24,000,000 渲染像素）。
    -   PDF 还有最多 50 页的限制。

## 输出详情契约

该工具在 `details` 下返回结构化元数据。为创建查看器的模式共享的字段：

-   `artifactId`
-   `viewerUrl`
-   `viewerPath`
-   `title`
-   `expiresAt`
-   `inputKind`
-   `fileCount`
-   `mode`

当渲染 PNG 或 PDF 时的文件字段：

-   `filePath`
-   `path`（与 `filePath` 值相同，用于消息工具兼容性）
-   `fileBytes`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`

模式行为摘要：

-   `mode: "view"`: 仅查看器字段。
-   `mode: "file"`: 仅文件字段，无查看器工件。
-   `mode: "both"`: 查看器字段加上文件字段。如果文件渲染失败，查看器仍会返回，并带有 `fileError`。

## 折叠的未更改部分

-   查看器可以显示类似 `N 行未修改` 的行。
-   这些行上的展开控件是有条件的，并非对所有输入类型都保证出现。
-   当渲染的差异具有可展开的上下文数据时，会出现展开控件，这对于 before 和 after 输入是典型情况。
-   对于许多统一的补丁输入，省略的上下文主体在解析的补丁块中不可用，因此该行可能出现但没有展开控件。这是预期行为。
-   `expandUnchanged` 仅当存在可展开的上下文时才适用。

## 插件默认值

在 `~/.openclaw/openclaw.json` 中设置插件范围的默认值：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
          },
        },
      },
    },
  },
}
```

支持的默认值：

-   `fontFamily`
-   `fontSize`
-   `lineSpacing`
-   `layout`
-   `showLineNumbers`
-   `diffIndicators`
-   `wordWrap`
-   `background`
-   `theme`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`
-   `mode`

显式的工具参数会覆盖这些默认值。

## 安全配置

-   `security.allowRemoteViewer` (`boolean`, 默认 `false`)
    -   `false`: 对查看器路由的非环回请求被拒绝。
    -   `true`: 如果令牌化路径有效，则允许远程查看器。

示例：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## 工件生命周期和存储

-   工件存储在临时子文件夹下：`$TMPDIR/openclaw-diffs`。
-   查看器工件元数据包含：
    -   随机工件 ID（20 个十六进制字符）
    -   随机令牌（48 个十六进制字符）
    -   `createdAt` 和 `expiresAt`
    -   存储的 `viewer.html` 路径
-   未指定时，默认查看器 TTL 为 30 分钟。
-   最大可接受的查看器 TTL 为 6 小时。
-   清理在工件创建后机会性地运行。
-   过期的工件会被删除。
-   当元数据缺失时，后备清理会删除超过 24 小时的陈旧文件夹。

## 查看器 URL 和网络行为

查看器路由：

-   `/plugins/diffs/view/{artifactId}/{token}`

查看器资源：

-   `/plugins/diffs/assets/viewer.js`
-   `/plugins/diffs/assets/viewer-runtime.js`

URL 构建行为：

-   如果提供了 `baseUrl`，则在严格验证后使用它。
-   没有 `baseUrl` 时，查看器 URL 默认为环回地址 `127.0.0.1`。
-   如果网关绑定模式为 `custom` 且设置了 `gateway.customBindHost`，则使用该主机。

`baseUrl` 规则：

-   必须是 `http://` 或 `https://`。
-   查询和哈希被拒绝。
-   允许来源加可选的基础路径。

## 安全模型

查看器加固：

-   默认仅限环回访问。
-   令牌化的查看器路径，具有严格的 ID 和令牌验证。
-   查看器响应 CSP：
    -   `default-src 'none'`
    -   脚本和资源仅来自自身
    -   无出站 `connect-src`
-   启用远程访问时的远程失败限流：
    -   60 秒内 40 次失败
    -   60 秒锁定（`429 Too Many Requests`）

文件渲染加固：

-   截图浏览器请求路由默认拒绝。
-   仅允许来自 `http://127.0.0.1/plugins/diffs/assets/*` 的本地查看器资源。
-   阻止外部网络请求。

## 文件模式的浏览器要求

`mode: "file"` 和 `mode: "both"` 需要兼容 Chromium 的浏览器。解析顺序：

1.  OpenClaw 配置中的 `browser.executablePath`。
2.  环境变量：
    -   `OPENCLAW_BROWSER_EXECUTABLE_PATH`
    -   `BROWSER_EXECUTABLE_PATH`
    -   `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
3.  平台命令/路径发现后备。

常见的失败文本：

-   `Diff PNG/PDF 渲染需要兼容 Chromium 的浏览器...`

通过安装 Chrome、Chromium、Edge 或 Brave，或设置上述可执行路径选项之一来修复。

## 故障排除

输入验证错误：

-   `提供补丁或同时提供 before 和 after 文本。`
    -   同时包含 `before` 和 `after`，或提供 `patch`。
-   `请提供补丁或 before/after 输入，不要同时提供两者。`
    -   不要混合输入模式。
-   `无效的 baseUrl: ...`
    -   使用 `http(s)` 来源加可选路径，无查询/哈希。
-   `{field} 超过最大大小 (...)`
    -   减少负载大小。
-   大型补丁被拒绝
    -   减少补丁文件数量或总行数。

查看器可访问性问题：

-   查看器 URL 默认解析为 `127.0.0.1`。
-   对于远程访问场景，要么：
    -   每次工具调用传递 `baseUrl`，或者
    -   使用 `gateway.bind=custom` 和 `gateway.customBindHost`
-   仅当你打算进行外部查看器访问时，才启用 `security.allowRemoteViewer`。

未修改行没有展开按钮：

-   当补丁输入不携带可展开的上下文时，可能会发生这种情况。
-   这是预期的，并不表示查看器故障。

找不到工件：

-   工件因 TTL 过期。
-   令牌或路径已更改。
-   清理删除了陈旧数据。

## 操作指导

-   对于画布中的本地交互式审查，优先使用 `mode: "view"`。
-   对于需要附件的外发聊天渠道，优先使用 `mode: "file"`。
-   除非你的部署需要远程查看器 URL，否则保持 `allowRemoteViewer` 禁用。
-   为敏感差异设置明确的短 `ttlSeconds`。
-   除非必要，避免在差异输入中发送秘密信息。
-   如果你的渠道对图像压缩很激进（例如 Telegram 或 WhatsApp），优先使用 PDF 输出（`fileFormat: "pdf"`）。

差异渲染引擎：

## 相关文档

-   [工具概述](../tools.md)
-   [插件](./plugin.md)
-   [浏览器](./browser.md)

[Perplexity Sonar](../perplexity.md)[PDF 工具](./pdf.md)