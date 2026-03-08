

  内置工具

  
# PDF 工具

`pdf` 工具分析一个或多个 PDF 文档并返回文本。快速行为：

-   针对 Anthropic 和 Google 模型提供商的本地提供商模式。
-   针对其他提供商的提取回退模式（先提取文本，需要时再提取页面图像）。
-   支持单输入 (`pdf`) 或多输入 (`pdfs`)，每次调用最多 10 个 PDF。

## 可用性

仅当 OpenClaw 能为代理解析到支持 PDF 的模型配置时，才会注册该工具：

1.  `agents.defaults.pdfModel`
2.  回退到 `agents.defaults.imageModel`
3.  回退到基于可用认证的最佳提供商默认值

如果无法解析到可用的模型，则不会暴露 `pdf` 工具。

## 输入参考

-   `pdf` (`string`): 一个 PDF 路径或 URL
-   `pdfs` (`string[]`): 多个 PDF 路径或 URL，最多总共 10 个
-   `prompt` (`string`): 分析提示词，默认为 `Analyze this PDF document.`
-   `pages` (`string`): 页面过滤器，例如 `1-5` 或 `1,3,7-9`
-   `model` (`string`): 可选的模型覆盖 (`provider/model`)
-   `maxBytesMb` (`number`): 每个 PDF 的大小上限（单位 MB）

输入说明：

-   `pdf` 和 `pdfs` 在加载前会合并并去重。
-   如果未提供 PDF 输入，工具会报错。
-   `pages` 被解析为基于 1 的页码，去重、排序并限制在配置的最大页数内。
-   `maxBytesMb` 默认为 `agents.defaults.pdfMaxBytesMb` 或 `10`。

## 支持的 PDF 引用

-   本地文件路径（包括 `~` 扩展）
-   `file://` URL
-   `http://` 和 `https://` URL

引用说明：

-   其他 URI 方案（例如 `ftp://`）会被拒绝，并返回 `unsupported_pdf_reference`。
-   在沙盒模式下，远程 `http(s)` URL 会被拒绝。
-   启用仅工作空间文件策略时，允许根目录之外的本地文件路径会被拒绝。

## 执行模式

### 本地提供商模式

本地模式用于 `anthropic` 和 `google` 提供商。该工具将原始 PDF 字节直接发送给提供商 API。本地模式限制：

-   不支持 `pages`。如果设置了，工具会返回错误。

### 提取回退模式

回退模式用于非本地提供商。流程：

1.  从选定的页面提取文本（最多 `agents.defaults.pdfMaxPages`，默认 `20`）。
2.  如果提取的文本长度低于 `200` 个字符，则将选定页面渲染为 PNG 图像并包含它们。
3.  将提取的内容加上提示词发送给选定的模型。

回退详情：

-   页面图像提取使用 `4,000,000` 的像素预算。
-   如果目标模型不支持图像输入且没有可提取的文本，工具会报错。
-   提取回退需要 `pdfjs-dist`（以及用于图像渲染的 `@napi-rs/canvas`）。

## 配置

```json
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

完整字段详情请参阅[配置参考](../gateway/configuration-reference.md)。

## 输出详情

工具在 `content[0].text` 中返回文本，在 `details` 中返回结构化元数据。常见的 `details` 字段：

-   `model`: 解析后的模型引用 (`provider/model`)
-   `native`: 本地提供商模式为 `true`，回退模式为 `false`
-   `attempts`: 成功前失败的回退尝试次数

路径字段：

-   单 PDF 输入：`details.pdf`
-   多 PDF 输入：`details.pdfs[]` 包含 `pdf` 条目
-   沙盒路径重写元数据（如适用）：`rewrittenFrom`

## 错误行为

-   缺少 PDF 输入：抛出 `pdf required: provide a path or URL to a PDF document`
-   PDF 过多：在 `details.error = "too_many_pdfs"` 中返回结构化错误
-   不支持的引用方案：返回 `details.error = "unsupported_pdf_reference"`
-   本地模式设置了 `pages`：抛出清晰的 `pages is not supported with native PDF providers` 错误

## 示例

单个 PDF：

```json
{
  "pdf": "/tmp/report.pdf",
  "prompt": "Summarize this report in 5 bullets"
}
```

多个 PDF：

```json
{
  "pdfs": ["/tmp/q1.pdf", "/tmp/q2.pdf"],
  "prompt": "Compare risks and timeline changes across both documents"
}
```

页面过滤的回退模型：

```json
{
  "pdf": "https://example.com/report.pdf",
  "pages": "1-3,7",
  "model": "openai/gpt-5-mini",
  "prompt": "Extract only customer-impacting incidents"
}
```

[差异](./diffs.md)[提升模式](./elevated.md)