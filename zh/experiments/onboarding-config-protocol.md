

  实验

  
# 引导与配置协议

目的：在 CLI、macOS 应用和 Web UI 之间共享引导与配置界面。

## 组件

-   向导引擎（共享会话 + 提示 + 引导状态）。
-   CLI 引导使用与 UI 客户端相同的向导流程。
-   网关 RPC 暴露向导 + 配置模式端点。
-   macOS 引导使用向导步骤模型。
-   Web UI 根据 JSON 模式 + 界面提示渲染配置表单。

## 网关 RPC

-   `wizard.start` 参数：`{ mode?: "local"|"remote", workspace?: string }`
-   `wizard.next` 参数：`{ sessionId, answer?: { stepId, value? } }`
-   `wizard.cancel` 参数：`{ sessionId }`
-   `wizard.status` 参数：`{ sessionId }`
-   `config.schema` 参数：`{}`
-   `config.schema.lookup` 参数：`{ path }`
    -   `path` 接受标准配置段以及斜杠分隔的插件 ID，例如 `plugins.entries.pack/one.config`。

响应（结构）

-   向导：`{ sessionId, done, step?, status?, error? }`
-   配置模式：`{ schema, uiHints, version, generatedAt }`
-   配置模式查找：`{ path, schema, hint?, hintPath?, children[] }`

## 界面提示

-   `uiHints` 按路径键控；可选元数据（标签/帮助/分组/顺序/高级/敏感/占位符）。
-   敏感字段渲染为密码输入框；无脱敏层。
-   不支持的模式节点回退到原始 JSON 编辑器。

## 备注

-   本文档是跟踪引导/配置协议重构的唯一位置。

[Kilo 网关集成](../design/kilo-gateway-integration.md)[ACP 线程绑定代理](./plans/acp-thread-bound-agents.md)

---