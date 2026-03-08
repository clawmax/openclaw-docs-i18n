

  实验

  
# PTY 与进程监督计划

## 1\. 问题与目标

我们需要为长时间运行的命令执行建立一个可靠的生命周期，涵盖：

-   `exec` 前台运行
-   `exec` 后台运行
-   `process` 后续操作 (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
-   CLI 代理运行器子进程

目标不仅仅是支持 PTY。目标是实现可预测的所有权、取消、超时和清理，无需不安全的进程匹配启发式方法。

## 2\. 范围与边界

-   将实现保持在 `src/process/supervisor` 内部。
-   不要为此创建新的包。
-   在可行的情况下保持当前行为兼容性。
-   不要将范围扩大到终端回放或 tmux 风格的会话持久化。

## 3\. 本分支已实现的内容

### 监督器基线已存在

-   监督器模块已就位，位于 `src/process/supervisor/*`。
-   Exec 运行时和 CLI 运行器已通过监督器的 spawn 和 wait 进行路由。
-   注册表终结是幂等的。

### 本次已完成

1.  明确的 PTY 命令契约

-   `SpawnInput` 现在是 `src/process/supervisor/types.ts` 中的一个可辨识联合类型。
-   PTY 运行需要 `ptyCommand`，而不是重用通用的 `argv`。
-   监督器不再在 `src/process/supervisor/supervisor.ts` 中通过 argv 连接重建 PTY 命令字符串。
-   Exec 运行时现在在 `src/agents/bash-tools.exec-runtime.ts` 中直接传递 `ptyCommand`。

2.  进程层类型解耦

-   监督器类型不再从 agents 导入 `SessionStdin`。
-   进程本地 stdin 契约位于 `src/process/supervisor/types.ts` (`ManagedRunStdin`)。
-   适配器现在仅依赖于进程层类型：
    -   `src/process/supervisor/adapters/child.ts`
    -   `src/process/supervisor/adapters/pty.ts`

3.  进程工具生命周期所有权改进

-   `src/agents/bash-tools.process.ts` 现在首先通过监督器请求取消。
-   `process kill/remove` 现在在监督器查找失败时使用进程树回退终止。
-   `remove` 通过在请求终止后立即删除正在运行的会话条目，保持确定性的移除行为。

4.  单一来源的看门狗默认值

-   在 `src/agents/cli-watchdog-defaults.ts` 中添加了共享默认值。
-   `src/agents/cli-backends.ts` 使用共享默认值。
-   `src/agents/cli-runner/reliability.ts` 使用相同的共享默认值。

5.  已死辅助代码清理

-   从 `src/agents/bash-tools.shared.ts` 中移除了未使用的 `killSession` 辅助路径。

6.  添加了直接监督器路径测试

-   添加了 `src/agents/bash-tools.process.supervisor.test.ts` 以覆盖通过监督器取消进行 kill 和 remove 路由的情况。

7.  可靠性缺口修复完成

-   `src/agents/bash-tools.process.ts` 现在在监督器查找失败时回退到真正的操作系统级进程终止。
-   `src/process/supervisor/adapters/child.ts` 现在对默认的取消/超时终止路径使用进程树终止语义。
-   在 `src/process/kill-tree.ts` 中添加了共享的进程树工具。

8.  添加了 PTY 契约边缘情况覆盖

-   添加了 `src/process/supervisor/supervisor.pty-command.test.ts` 用于逐字 PTY 命令转发和空命令拒绝。
-   添加了 `src/process/supervisor/adapters/child.test.ts` 用于子适配器取消中的进程树终止行为。

## 4\. 剩余缺口与决策

### 可靠性状态

本次所需的两个可靠性缺口现已关闭：

-   `process kill/remove` 现在在监督器查找失败时具有真正的操作系统终止回退。
-   子进程取消/超时现在对默认终止路径使用进程树终止语义。
-   为这两种行为添加了回归测试。

### 持久性与启动协调

重启行为现在明确定义为仅限内存生命周期。

-   `reconcileOrphans()` 在 `src/process/supervisor/supervisor.ts` 中按设计保持无操作。
-   进程重启后不会恢复正在运行的进程。
-   此边界是本次实现有意为之，以避免部分持久化风险。

### 可维护性后续工作

1.  `src/agents/bash-tools.exec-runtime.ts` 中的 `runExecProcess` 仍然处理多个职责，可以在后续工作中拆分为专注的辅助函数。

## 5\. 实施计划

所需可靠性和契约项目的实施阶段已完成。已完成：

-   `process kill/remove` 回退真实终止
-   子适配器默认终止路径的进程树取消
-   回退终止和子适配器终止路径的回归测试
-   明确 `ptyCommand` 下的 PTY 命令边缘情况测试
-   明确的内存重启边界，`reconcileOrphans()` 按设计无操作

可选后续工作：

-   将 `runExecProcess` 拆分为专注的辅助函数，无行为漂移

## 6\. 文件映射

### 进程监督器

-   `src/process/supervisor/types.ts` 更新为包含可辨识的 spawn 输入和进程本地 stdin 契约。
-   `src/process/supervisor/supervisor.ts` 更新为使用明确的 `ptyCommand`。
-   `src/process/supervisor/adapters/child.ts` 和 `src/process/supervisor/adapters/pty.ts` 与代理类型解耦。
-   `src/process/supervisor/registry.ts` 幂等终结保持不变并保留。

### Exec 与进程集成

-   `src/agents/bash-tools.exec-runtime.ts` 更新为显式传递 PTY 命令并保留回退路径。
-   `src/agents/bash-tools.process.ts` 更新为通过监督器取消，并具有真实的进程树回退终止。
-   `src/agents/bash-tools.shared.ts` 移除了直接的终止辅助路径。

### CLI 可靠性

-   `src/agents/cli-watchdog-defaults.ts` 作为共享基线添加。
-   `src/agents/cli-backends.ts` 和 `src/agents/cli-runner/reliability.ts` 现在使用相同的默认值。

## 7\. 本次验证运行

单元测试：

-   `pnpm vitest src/process/supervisor/registry.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
-   `pnpm vitest src/process/supervisor/adapters/child.test.ts`
-   `pnpm vitest src/agents/cli-backends.test.ts`
-   `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
-   `pnpm vitest src/process/exec.test.ts`

E2E 目标：

-   `pnpm vitest src/agents/cli-runner.test.ts`
-   `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

类型检查说明：

-   在此仓库中使用 `pnpm build`（以及 `pnpm check` 进行完整的 lint/docs 检查）。旧笔记中提到的 `pnpm tsgo` 已过时。

## 8\. 保持的操作保证

-   Exec 环境加固行为不变。
-   审批和允许列表流程不变。
-   输出清理和输出上限不变。
-   PTY 适配器仍然保证在强制终止和监听器处置时等待结算。

## 9\. 完成定义

1.  监督器是托管运行的生命周期所有者。
2.  PTY spawn 使用明确的命令契约，无需 argv 重建。
3.  进程层对于监督器 stdin 契约没有对代理层的类型依赖。
4.  看门狗默认值单一来源。
5.  目标单元和 e2e 测试保持通过。
6.  重启持久性边界已明确记录或完全实现。

## 10\. 总结

该分支现在具有一个更连贯、更安全的监督结构：

-   明确的 PTY 契约
-   更清晰的进程分层
-   进程操作由监督器驱动的取消路径
-   监督器查找失败时的真实回退终止
-   子进程运行默认终止路径的进程树取消
-   统一的看门狗默认值
-   明确的内存重启边界（本次不跨重启协调孤儿进程）

[OpenResponses 网关计划](./openresponses-gateway.md)[会话绑定通道无关计划](./session-binding-channel-agnostic.md)

---