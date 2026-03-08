

  内置工具

  
# 工具循环检测

OpenClaw 可以防止智能体陷入重复的工具调用模式。此防护功能**默认处于禁用状态**。仅在需要时启用它，因为严格的设置可能会阻止合法的重复调用。

## 存在原因

-   检测无进展的重复序列。
-   检测高频无结果循环（相同工具、相同输入、重复错误）。
-   检测已知轮询工具的特定重复调用模式。

## 配置块

全局默认值：

```json
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

按智能体覆盖（可选）：

```json
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            warningThreshold: 8,
            criticalThreshold: 16,
          },
        },
      },
    ],
  },
}
```

### 字段行为

-   `enabled`: 主开关。`false` 表示不执行循环检测。
-   `historySize`: 为分析而保留的最近工具调用数量。
-   `warningThreshold`: 将模式归类为仅警告的阈值。
-   `criticalThreshold`: 阻止重复循环模式的阈值。
-   `globalCircuitBreakerThreshold`: 全局无进展断路器的阈值。
-   `detectors.genericRepeat`: 检测重复的相同工具 + 相同参数模式。
-   `detectors.knownPollNoProgress`: 检测已知的无状态变化的轮询类模式。
-   `detectors.pingPong`: 检测交替的乒乓模式。

## 推荐设置

-   从 `enabled: true` 开始，默认值不变。
-   保持阈值顺序为 `warningThreshold < criticalThreshold < globalCircuitBreakerThreshold`。
-   如果出现误报：
    -   提高 `warningThreshold` 和/或 `criticalThreshold`
    -   （可选）提高 `globalCircuitBreakerThreshold`
    -   仅禁用导致问题的检测器
    -   减小 `historySize` 以降低历史上下文的严格程度

## 日志与预期行为

当检测到循环时，OpenClaw 会报告一个循环事件，并根据严重程度阻止或抑制下一个工具调用周期。这可以保护用户免受令牌消耗失控和系统锁定的影响，同时保留正常的工具访问权限。

-   优先进行警告和临时抑制。
-   仅在重复证据累积时升级处理。

## 注意事项

-   `tools.loopDetection` 会与智能体级别的覆盖配置合并。
-   每个智能体的配置会完全覆盖或扩展全局值。
-   如果不存在配置，则防护栏保持关闭状态。

[Lobster](./lobster.md)[Reactions](./reactions.md)