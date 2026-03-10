

  实验

  
# 引导与配置协议

本文档定义了一套统一的引导和配置协议，让 CLI、macOS 应用和 Web UI 能够共享相同的配置界面与交互流程。

## 核心组件

- **向导引擎**：统一管理会话状态、提示信息和引导流程
- **CLI 引导**：与 UI 客户端使用相同的向导流程
- **网关 RPC**：提供向导和配置模式的 API 端点
- **macOS 引导**：基于向导步骤模型实现
- **Web UI**：根据 JSON Schema 和 UI 提示动态渲染配置表单

## 网关 RPC 接口

以下是可用的 RPC 方法：

| 方法 | 参数说明 |
|------|----------|
| `wizard.start` | `{ mode?: "local"\|"remote", workspace?: string }` — 启动向导会话 |
| `wizard.next` | `{ sessionId, answer?: { stepId, value? } }` — 推进到下一步 |
| `wizard.cancel` | `{ sessionId }` — 取消当前会话 |
| `wizard.status` | `{ sessionId }` — 查询会话状态 |
| `config.schema` | `{}` — 获取完整配置模式 |
| `config.schema.lookup` | `{ path }` — 按路径查询配置模式 |

`config.schema.lookup` 的 `path` 参数支持标准配置段和斜杠分隔的插件 ID，例如：`plugins.entries.pack/one.config`

### 响应结构

- **向导响应**：`{ sessionId, done, step?, status?, error? }`
- **配置模式响应**：`{ schema, uiHints, version, generatedAt }`
- **配置模式查询响应**：`{ path, schema, hint?, hintPath?, children[] }`

## UI 提示机制

`uiHints` 按路径索引，为每个配置项提供可选的元数据：

- `label` — 字段标签
- `help` — 帮助文本
- `group` — 分组名称
- `order` — 排序权重
- `advanced` — 是否为高级选项
- `sensitive` — 是否为敏感字段
- `placeholder` — 占位文本

特殊处理：

- 敏感字段自动渲染为密码输入框，无需额外的脱敏处理层
- 不支持的 Schema 节点会自动回退到原始 JSON 编辑器

## 备注

本文档是跟踪引导/配置协议重构的唯一权威来源。

[Kilo 网关集成](../design/kilo-gateway-integration.md) · [ACP 线程绑定智能体](./plans/acp-thread-bound-agents.md)