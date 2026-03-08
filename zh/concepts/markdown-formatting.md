

  概念内部

  
# Markdown 格式化

OpenClaw 通过将外发的 Markdown 转换为共享的中间表示（IR），然后再渲染为特定频道的输出，来进行格式化。IR 保持源文本不变，同时携带样式/链接跨度，以便分块和渲染在不同频道间保持一致。

## 目标

-   **一致性：** 一次解析步骤，多个渲染器。
-   **安全分块：** 在渲染前分割文本，使内联格式化永远不会跨块中断。
-   **频道适配：** 将相同的 IR 映射到 Slack mrkdwn、Telegram HTML 和 Signal 样式范围，而无需重新解析 Markdown。

## 处理流程

1.  **解析 Markdown -> IR**
    -   IR 是纯文本加上样式跨度（粗体/斜体/删除线/代码/剧透）和链接跨度。
    -   偏移量使用 UTF-16 代码单元，以便 Signal 样式范围与其 API 对齐。
    -   仅当频道选择启用表格转换时，才会解析表格。
2.  **对 IR 进行分块（格式优先）**
    -   分块发生在渲染前的 IR 文本上。
    -   内联格式化不会跨块分割；跨度会按块进行切片。
3.  **按频道渲染**
    -   **Slack：** mrkdwn 标记（粗体/斜体/删除线/代码），链接格式为 `<url|label>`。
    -   **Telegram：** HTML 标签（``, ``, ``, ``, ``, ``）。
    -   **Signal：** 纯文本 + `text-style` 范围；当标签与 URL 不同时，链接变为 `label (url)`。

## IR 示例

输入 Markdown：

```bash
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR（示意图）：

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## 使用场景

-   Slack、Telegram 和 Signal 的外发适配器从 IR 进行渲染。
-   其他频道（WhatsApp、iMessage、MS Teams、Discord）仍使用纯文本或其自身的格式化规则，在启用时，Markdown 表格转换会在分块前应用。

## 表格处理

Markdown 表格在聊天客户端中并未得到一致支持。使用 `markdown.tables` 来控制每个频道（以及每个账户）的转换方式。

-   `code`：将表格渲染为代码块（大多数频道的默认设置）。
-   `bullets`：将每一行转换为项目符号点（Signal 和 WhatsApp 的默认设置）。
-   `off`：禁用表格解析和转换；原始表格文本直接通过。

配置键：

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## 分块规则

-   分块限制来自频道适配器/配置，并应用于 IR 文本。
-   代码围栏作为一个单独的块保留，并带有尾随换行符，以便频道正确渲染它们。
-   列表前缀和块引用前缀是 IR 文本的一部分，因此分块不会在前缀中间分割。
-   内联样式（粗体/斜体/删除线/内联代码/剧透）永远不会跨块分割；渲染器会在每个块内重新打开样式。

如果您需要更多关于跨频道分块行为的信息，请参阅 [流式传输 + 分块](./streaming.md)。

## 链接策略

-   **Slack：** `[label](url)` -> `<url|label>`；裸 URL 保持原样。解析期间禁用自动链接，以避免重复链接。
-   **Telegram：** `[label](url)` -> `label
`（HTML 解析模式）。
-   **Signal：** `[label](url)` -> `label (url)`，除非标签与 URL 匹配。

## 剧透

剧透标记（`||spoiler||`）仅针对 Signal 进行解析，在那里它们映射到 SPOILER 样式范围。其他频道将它们视为纯文本。

## 如何添加或更新频道格式化器

1.  **一次性解析：** 使用共享的 `markdownToIR(...)` 辅助函数，并配置频道适当的选项（自动链接、标题样式、块引用前缀）。
2.  **渲染：** 使用 `renderMarkdownWithMarkers(...)` 和一个样式标记映射（或 Signal 样式范围）实现一个渲染器。
3.  **分块：** 在渲染前调用 `chunkMarkdownIR(...)`；渲染每个块。
4.  **连接适配器：** 更新频道外发适配器以使用新的分块器和渲染器。
5.  **测试：** 添加或更新格式化测试，如果频道使用分块，则添加外发交付测试。

## 常见陷阱

-   Slack 尖括号标记（`<@U123>`, `<#C123>`, `<https://...>`）必须保留；安全地转义原始 HTML。
-   Telegram HTML 要求对标签外的文本进行转义，以避免标记损坏。
-   Signal 样式范围依赖于 UTF-16 偏移量；不要使用代码点偏移量。
-   保留围栏代码块的尾随换行符，以便结束标记位于其单独的行上。

[TypeBox](./typebox.md)[输入指示器](./typing-indicators.md)