

  实验

  
# 会话绑定通道无关计划

## 概述

本文档定义了长期的通道无关会话绑定模型以及下一次实施迭代的具体范围。目标：

-   使子智能体绑定会话路由成为核心能力
-   将通道特定行为保留在适配器中
-   避免正常 Discord 行为出现回归

## 存在原因

当前行为混合了：

-   完成结果内容策略
-   目标路由策略
-   Discord 特定细节

这导致了以下边缘情况：

-   并发运行时主频道和线程重复交付
-   重用绑定管理器时使用过期的令牌
-   基于 Webhook 发送时缺少活动记录

## 迭代 1 范围

本次迭代范围有意限制。

### 1. 添加通道无关核心接口

为绑定和路由添加核心类型和服务接口。建议的核心类型：

```bash
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

核心服务契约：

```bash
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. 为子智能体完成结果添加一个核心交付路由器

为完成事件添加单一的目标解析路径。路由器契约：

```bash
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

对于本次迭代：

-   只有 `task_completion` 通过此新路径路由
-   其他事件类型的现有路径保持不变

### 3. 保持 Discord 作为适配器

Discord 仍然是第一个适配器实现。适配器职责：

-   创建/重用线程对话
-   通过 Webhook 或频道发送发送绑定消息
-   验证线程状态（已归档/已删除）
-   映射适配器元数据（Webhook 身份、线程 ID）

### 4. 修复当前已知的正确性问题

本次迭代中需要：

-   重用现有线程绑定管理器时刷新令牌使用
-   记录基于 Webhook 的 Discord 发送的出站活动
-   当为会话模式完成结果选择了绑定的线程目标时，停止隐式回退到主频道

### 5. 保持当前运行时安全默认值

对于禁用了线程绑定生成的用户，行为不变。默认值保持：

-   `channels.discord.threadBindings.spawnSubagentSessions = false`

结果：

-   普通 Discord 用户保持当前行为
-   新的核心路径仅影响启用了绑定会话完成路由的情况

## 不在迭代 1 中

明确推迟：

-   ACP 绑定目标 (`targetKind: "acp"`)
-   Discord 之外的新通道适配器
-   全局替换所有交付路径 (`spawn_ack`, 未来的 `subagent_message`)
-   协议级别变更
-   为所有绑定持久化进行存储迁移/版本重新设计

关于 ACP 的说明：

-   接口设计为 ACP 留出空间
-   ACP 实现不在本次迭代中开始

## 路由不变量

这些不变量在迭代 1 中是强制性的。

-   目标选择和内容生成是独立的步骤
-   如果会话模式完成结果解析到一个活动的绑定目标，交付必须针对该目标
-   没有从绑定目标到主频道的隐藏重路由
-   回退行为必须是明确且可观察的

## 兼容性与发布

兼容性目标：

-   对于禁用了线程绑定生成的用户无回归
-   本次迭代中非 Discord 通道无变化

发布步骤：

1.  在当前功能开关后落地接口和路由器。
2.  通过路由器路由 Discord 完成模式绑定交付。
3.  为非绑定流保留旧路径。
4.  通过针对性测试和金丝雀运行时日志进行验证。

## 迭代 1 中所需的测试

所需的单元和集成测试覆盖：

-   管理器令牌轮换在管理器重用后使用最新令牌
-   Webhook 发送更新频道活动时间戳
-   同一请求者频道中的两个活动绑定会话不会重复发送到主频道
-   绑定会话模式运行的完成结果仅解析到线程目标
-   禁用的生成标志保持旧行为不变

## 建议的实施文件

核心：

-   `src/infra/outbound/session-binding-service.ts` (新)
-   `src/infra/outbound/bound-delivery-router.ts` (新)
-   `src/agents/subagent-announce.ts` (完成结果目标解析集成)

Discord 适配器和运行时：

-   `src/discord/monitor/thread-bindings.manager.ts`
-   `src/discord/monitor/reply-delivery.ts`
-   `src/discord/send.outbound.ts`

测试：

-   `src/discord/monitor/provider*.test.ts`
-   `src/discord/monitor/reply-delivery.test.ts`
-   `src/agents/subagent-announce.format.test.ts`

## 迭代 1 的完成标准

-   核心接口存在并已为完成结果路由连接
-   上述正确性修复已通过测试合并
-   会话模式绑定运行中无主频道和线程重复完成结果交付
-   对于禁用了绑定生成的部署，行为无变化
-   ACP 保持明确推迟

[PTY 与进程监督计划](./pty-process-supervision.md)[工作空间记忆研究](../research/memory.md)