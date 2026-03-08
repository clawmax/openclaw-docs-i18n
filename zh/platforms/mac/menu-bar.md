

  macOS 配套应用

  
# 菜单栏

## 显示内容

-   我们在菜单栏图标和菜单的第一个状态行中展示当前智能体的工作状态。
-   当有工作处于活动状态时，健康状态会被隐藏；当所有会话都空闲时，健康状态会恢复显示。
-   菜单中的“节点”区块仅列出**设备**（通过 `node.list` 配对的节点），不包含客户端/在线状态条目。
-   当有提供者使用量快照可用时，上下文下方会出现一个“使用情况”部分。

## 状态模型

-   会话：事件携带 `runId`（每次运行）以及有效载荷中的 `sessionKey`。“主”会话的键是 `main`；如果不存在，则回退到最近更新的会话。
-   优先级：主会话始终优先。如果主会话处于活动状态，其状态会立即显示。如果主会话空闲，则显示最近活动的非主会话。我们不会在活动进行中切换；只有当当前会话变为空闲或主会话变为活动状态时才会切换。
-   活动类型：
    -   `job`：高级命令执行（`state: started|streaming|done|error`）。
    -   `tool`：`phase: start|result`，附带 `toolName` 和 `meta/args`。

## IconState 枚举 (Swift)

-   `idle`
-   `workingMain(ActivityKind)`
-   `workingOther(ActivityKind)`
-   `overridden(ActivityKind)` （调试覆盖）

### ActivityKind → 字形

-   `exec` → 💻
-   `read` → 📄
-   `write` → ✍️
-   `edit` → 📝
-   `attach` → 📎
-   默认 → 🛠️

### 视觉映射

-   `idle`：正常的小动物图标。
-   `workingMain`：带字形的徽章，完全着色，腿部有“工作中”动画。
-   `workingOther`：带字形的徽章，柔和着色，无快速移动动画。
-   `overridden`：无论活动如何，都使用选定的字形/着色。

## 状态行文本（菜单）

-   当工作处于活动状态时：`<会话角色> · <活动标签>`
    -   示例：`主 · exec: pnpm test`, `其他 · read: apps/macos/Sources/OpenClaw/AppState.swift`。
-   空闲时：回退到健康摘要。

## 事件摄取

-   来源：控制通道 `agent` 事件（`ControlChannel.handleAgentEvent`）。
-   解析字段：
    -   `stream: "job"` 附带 `data.state` 用于开始/停止。
    -   `stream: "tool"` 附带 `data.phase`、`name`、可选的 `meta`/`args`。
-   标签：
    -   `exec`：`args.command` 的第一行。
    -   `read`/`write`：缩短的路径。
    -   `edit`：路径加上从 `meta`/差异计数推断出的变更类型。
    -   回退：工具名称。

## 调试覆盖

-   设置 ▸ 调试 ▸ “图标覆盖”选择器：
    -   `系统（自动）` （默认）
    -   `工作中：主会话` （按工具类型）
    -   `工作中：其他会话` （按工具类型）
    -   `空闲`
-   通过 `@AppStorage("iconOverride")` 存储；映射到 `IconState.overridden`。

## 测试清单

-   触发主会话任务：验证图标立即切换且状态行显示主标签。
-   在主会话空闲时触发非主会话任务：图标/状态显示非主会话；保持稳定直到任务完成。
-   在其他会话活动时启动主会话：图标立即切换到主会话。
-   快速连续的工具调用：确保徽章不闪烁（工具结果有 TTL 宽限期）。
-   所有会话空闲后，健康行重新出现。

[macOS 开发设置](./dev-setup.md)[语音唤醒](./voicewake.md)

---