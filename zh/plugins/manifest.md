

  扩展

  
# 插件清单

每个插件**必须**在其**插件根目录**下包含一个 `openclaw.plugin.json` 文件。OpenClaw 使用此清单来验证配置，**而无需执行插件代码**。缺失或无效的清单将被视为插件错误，并会阻止配置验证。查看完整的插件系统指南：[插件](../tools/plugin.md)。

## 必需字段

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

必需键：

-   `id` (字符串): 规范插件 ID。
-   `configSchema` (对象): 插件配置的 JSON Schema（内联）。

可选键：

-   `kind` (字符串): 插件种类（例如：`"memory"`、`"context-engine"`）。
-   `channels` (数组): 此插件注册的频道 ID（例如：`["matrix"]`）。
-   `providers` (数组): 此插件注册的提供者 ID。
-   `skills` (数组): 要加载的技能目录（相对于插件根目录）。
-   `name` (字符串): 插件的显示名称。
-   `description` (字符串): 插件简短摘要。
-   `uiHints` (对象): 用于 UI 渲染的配置字段标签/占位符/敏感标志。
-   `version` (字符串): 插件版本（信息性）。

## JSON Schema 要求

-   **每个插件都必须附带一个 JSON Schema**，即使它不接受任何配置。
-   空模式是可以接受的（例如，`{ "type": "object", "additionalProperties": false }`）。
-   Schema 在配置读取/写入时验证，而非运行时。

## 验证行为

-   未知的 `channels.*` 键是**错误**，除非该频道 ID 已在插件清单中声明。
-   `plugins.entries.`、`plugins.allow`、`plugins.deny` 和 `plugins.slots.*` 必须引用**可发现**的插件 ID。未知 ID 是**错误**。
-   如果插件已安装但其清单或 Schema 损坏或缺失，验证将失败，并且 Doctor 会报告插件错误。
-   如果插件配置存在但插件**被禁用**，配置将被保留，并在 Doctor 和日志中显示**警告**。

## 注意事项

-   清单**对所有插件都是必需的**，包括本地文件系统加载的插件。
-   运行时仍会单独加载插件模块；清单仅用于发现和验证。
-   独占插件种类通过 `plugins.slots.*` 选择。
    -   `kind: "memory"` 由 `plugins.slots.memory` 选择。
    -   `kind: "context-engine"` 由 `plugins.slots.contextEngine` 选择（默认：内置 `legacy`）。
-   如果你的插件依赖于原生模块，请记录构建步骤以及任何包管理器允许列表要求（例如，pnpm `allow-build-scripts`
    -   `pnpm rebuild `）。

[Zalo 个人插件](./zalouser.md)[插件代理工具](./agent-tools.md)

---