

  实验

  
# 浏览器 Evaluate CDP 重构

## 背景

`act:evaluate` 在页面中执行用户提供的 JavaScript。目前它通过 Playwright（`page.evaluate` 或 `locator.evaluate`）运行。Playwright 按页面序列化 CDP 命令，因此一个卡住或长时间运行的 evaluate 会阻塞页面命令队列，并使该标签页上的后续每个操作看起来都“卡住”。PR #13498 增加了一个实用的安全网（有界 evaluate、中止传播和尽力恢复）。本文档描述了一个更大的重构，使 `act:evaluate` 本质上与 Playwright 隔离，这样卡住的 evaluate 就无法阻塞正常的 Playwright 操作。

## 目标

-   `act:evaluate` 不能永久阻塞同一标签页上的后续浏览器操作。
-   超时是端到端的单一事实来源，以便调用者可以依赖预算。
-   中止和超时在 HTTP 和进程内分派中以相同方式处理。
-   支持 evaluate 的元素定位，而无需将所有内容从 Playwright 切换出去。
-   保持对现有调用者和负载的向后兼容性。

## 非目标

-   用 CDP 实现替换所有浏览器操作（点击、输入、等待等）。
-   移除 PR #13498 中引入的现有安全网（它仍然是一个有用的后备方案）。
-   在现有的 `browser.evaluateEnabled` 门控之外引入新的不安全功能。
-   为 evaluate 添加进程隔离（工作进程/线程）。如果在此次重构后我们仍然看到难以恢复的卡死状态，那将是一个后续想法。

## 当前架构（为何会卡住）

从高层次看：

-   调用者向浏览器控制服务发送 `act:evaluate`。
-   路由处理程序调用 Playwright 来执行 JavaScript。
-   Playwright 序列化页面命令，因此一个永不结束的 evaluate 会阻塞队列。
-   卡住的队列意味着该标签页上后续的点击/输入/等待操作可能看起来挂起。

## 提议的架构

### 1\. 截止时间传播

引入单一的预算概念，并从中推导出所有内容：

-   调用者设置 `timeoutMs`（或未来的一个截止时间）。
-   外部请求超时、路由处理程序逻辑以及页面内的执行预算都使用相同的预算，并在需要时为序列化开销留出少量余量。
-   中止作为 `AbortSignal` 到处传播，以便取消操作保持一致。

实施方向：

-   添加一个小型辅助工具（例如 `createBudget({ timeoutMs, signal })`），返回：
    -   `signal`：链接的 AbortSignal
    -   `deadlineAtMs`：绝对截止时间
    -   `remainingMs()`：用于子操作的剩余预算
-   在以下位置使用此辅助工具：
    -   `src/browser/client-fetch.ts`（HTTP 和进程内分派）
    -   `src/node-host/runner.ts`（代理路径）
    -   浏览器操作实现（Playwright 和 CDP）

### 2\. 独立的 Evaluate 引擎（CDP 路径）

添加一个基于 CDP 的 evaluate 实现，该实现不共享 Playwright 的每页面命令队列。关键特性是 evaluate 传输使用单独的 WebSocket 连接和附加到目标的单独 CDP 会话。实施方向：

-   新模块，例如 `src/browser/cdp-evaluate.ts`，该模块：
    -   连接到配置的 CDP 端点（浏览器级别套接字）。
    -   使用 `Target.attachToTarget({ targetId, flatten: true })` 获取 `sessionId`。
    -   运行以下之一：
        -   `Runtime.evaluate` 用于页面级 evaluate，或
        -   `DOM.resolveNode` 加上 `Runtime.callFunctionOn` 用于元素 evaluate。
    -   在超时或中止时：
        -   尽力为该会话发送 `Runtime.terminateExecution`。
        -   关闭 WebSocket 并返回明确的错误。

注意：

-   这仍然在页面中执行 JavaScript，因此终止可能会有副作用。好处在于它不会阻塞 Playwright 队列，并且可以通过在传输层终止 CDP 会话来取消。

### 3\. Ref 策略（无需完全重写的元素定位）

难点在于元素定位。CDP 需要一个 DOM 句柄或 `backendDOMNodeId`，而目前大多数浏览器操作使用基于快照中 refs 的 Playwright 定位器。推荐方法：保留现有的 refs，但附加一个可选的 CDP 可解析 id。

#### 3.1 扩展存储的 Ref 信息

扩展存储的角色 ref 元数据以可选地包含 CDP id：

-   当前：`{ role, name, nth }`
-   提议：`{ role, name, nth, backendDOMNodeId?: number }`

这使所有现有的基于 Playwright 的操作保持工作，并允许 CDP evaluate 在 `backendDOMNodeId` 可用时接受相同的 `ref` 值。

#### 3.2 在快照时填充 backendDOMNodeId

生成角色快照时：

1.  像今天一样生成现有的角色 ref 映射（role, name, nth）。
2.  通过 CDP 获取 AX 树（`Accessibility.getFullAXTree`），并使用相同的重复处理规则计算 `(role, name, nth) -> backendDOMNodeId` 的并行映射。
3.  将 id 合并回当前标签页的存储 ref 信息中。

如果某个 ref 的映射失败，则将 `backendDOMNodeId` 保持为未定义。这使得该功能是尽力而为的，并且可以安全地推出。

#### 3.3 带 Ref 的 Evaluate 行为

在 `act:evaluate` 中：

-   如果 `ref` 存在且具有 `backendDOMNodeId`，则通过 CDP 运行元素 evaluate。
-   如果 `ref` 存在但没有 `backendDOMNodeId`，则回退到 Playwright 路径（使用安全网）。

可选逃生舱口：

-   扩展请求形状以直接接受 `backendDOMNodeId` 供高级调用者（以及调试）使用，同时保持 `ref` 作为主要接口。

### 4\. 保留最后的后备恢复路径

即使使用 CDP evaluate，也存在其他方式导致标签页或连接卡住。保留现有的恢复机制（终止执行 + 断开 Playwright 连接）作为最后手段，用于：

-   旧版调用者
-   CDP 附加被阻止的环境
-   意外的 Playwright 边缘情况

## 实施计划（单次迭代）

### 交付成果

-   一个基于 CDP 的 evaluate 引擎，在 Playwright 每页面命令队列之外运行。
-   一个单一的端到端超时/中止预算，由调用者和处理程序一致使用。
-   可以可选携带 `backendDOMNodeId` 用于元素 evaluate 的 Ref 元数据。
-   `act:evaluate` 在可能时优先使用 CDP 引擎，否则回退到 Playwright。
-   证明卡住的 evaluate 不会阻塞后续操作的测试。
-   使故障和回退可见的日志/指标。

### 实施清单

1.  添加一个共享的“预算”辅助工具，将 `timeoutMs` + 上游 `AbortSignal` 链接到：
    -   一个单一的 `AbortSignal`
    -   一个绝对截止时间
    -   一个用于下游操作的 `remainingMs()` 辅助工具
2.  更新所有调用者路径以使用该辅助工具，使 `timeoutMs` 在所有地方含义相同：
    -   `src/browser/client-fetch.ts`（HTTP 和进程内分派）
    -   `src/node-host/runner.ts`（节点代理路径）
    -   调用 `/act` 的 CLI 包装器（向 `browser evaluate` 添加 `--timeout-ms`）
3.  实现 `src/browser/cdp-evaluate.ts`：
    -   连接到浏览器级别的 CDP 套接字
    -   `Target.attachToTarget` 以获取 `sessionId`
    -   运行 `Runtime.evaluate` 用于页面 evaluate
    -   运行 `DOM.resolveNode` + `Runtime.callFunctionOn` 用于元素 evaluate
    -   超时/中止时：尽力执行 `Runtime.terminateExecution`，然后关闭套接字
4.  扩展存储的角色 ref 元数据以可选地包含 `backendDOMNodeId`：
    -   为 Playwright 操作保留现有的 `{ role, name, nth }` 行为
    -   为 CDP 元素定位添加 `backendDOMNodeId?: number`
5.  在快照创建期间填充 `backendDOMNodeId`（尽力而为）：
    -   通过 CDP 获取 AX 树（`Accessibility.getFullAXTree`）
    -   计算 `(role, name, nth) -> backendDOMNodeId` 并合并到存储的 ref 映射中
    -   如果映射不明确或缺失，则将 id 保持为未定义
6.  更新 `act:evaluate` 路由：
    -   如果没有 `ref`：始终使用 CDP evaluate
    -   如果 `ref` 解析为 `backendDOMNodeId`：使用 CDP 元素 evaluate
    -   否则：回退到 Playwright evaluate（仍然有界且可中止）
7.  保留现有的“最后手段”恢复路径作为后备，而非默认路径。
8.  添加测试：
    -   卡住的 evaluate 在预算内超时，并且下一个点击/输入成功
    -   中止取消 evaluate（客户端断开连接或超时）并解除对后续操作的阻塞
    -   映射失败时干净地回退到 Playwright
9.  添加可观测性：
    -   evaluate 持续时间和超时计数器
    -   terminateExecution 使用情况
    -   回退率（CDP -> Playwright）及原因

### 验收标准

-   故意挂起的 `act:evaluate` 在调用者预算内返回，并且不会阻塞标签页的后续操作。
-   `timeoutMs` 在 CLI、代理工具、节点代理和进程内调用中表现一致。
-   如果 `ref` 可以映射到 `backendDOMNodeId`，则元素 evaluate 使用 CDP；否则后备路径仍然有界且可恢复。

## 测试计划

-   单元测试：
    -   角色 ref 和 AX 树节点之间的 `(role, name, nth)` 匹配逻辑。
    -   预算辅助工具行为（余量、剩余时间计算）。
-   集成测试：
    -   CDP evaluate 超时在预算内返回且不阻塞下一个操作。
    -   中止取消 evaluate 并触发尽力而为的终止。
-   契约测试：
    -   确保 `BrowserActRequest` 和 `BrowserActResponse` 保持兼容。

## 风险与缓解措施

-   映射不完美：
    -   缓解措施：尽力而为的映射，回退到 Playwright evaluate，并添加调试工具。
-   `Runtime.terminateExecution` 有副作用：
    -   缓解措施：仅在超时/中止时使用，并在错误中记录该行为。
-   额外开销：
    -   缓解措施：仅在请求快照时获取 AX 树，按目标缓存，并保持 CDP 会话短暂。
-   扩展中继限制：
    -   缓解措施：当每页面套接字不可用时，使用浏览器级别的附加 API，并保留当前的 Playwright 路径作为后备。

## 开放性问题

-   新引擎是否应配置为 `playwright`、`cdp` 或 `auto`？
-   我们是否想为高级用户公开新的“nodeRef”格式，还是只保留 `ref`？
-   框架快照和选择器作用域快照应如何参与 AX 映射？

[统一运行时流式重构计划](./acp-unified-streaming-refactor.md)[OpenResponses 网关计划](./openresponses-gateway.md)