

  概念内部

  
# 时区

OpenClaw 标准化时间戳，使模型看到**单一的参考时间**。

## 消息信封（默认为本地时间）

传入的消息被包装在一个信封中，例如：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

信封中的时间戳**默认是主机本地时间**，精确到分钟。您可以通过以下方式覆盖此设置：

```json
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA 时区
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

-   `envelopeTimezone: "utc"` 使用 UTC。
-   `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（回退到主机时区）。
-   使用明确的 IANA 时区（例如 `"Europe/Vienna"`）来设置固定偏移量。
-   `envelopeTimestamp: "off"` 从信封头部移除绝对时间戳。
-   `envelopeElapsed: "off"` 移除经过时间后缀（`+2m` 这种样式）。

### 示例

**本地时间（默认）：**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**固定时区：**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**经过时间：**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

## 工具负载（原始提供商数据 + 标准化字段）

工具调用（`channels.discord.readMessages`、`channels.slack.readMessages` 等）返回**原始提供商时间戳**。我们还附加了标准化字段以确保一致性：

-   `timestampMs`（UTC 纪元毫秒数）
-   `timestampUtc`（ISO 8601 UTC 字符串）

原始提供商字段会被保留。

## 用于系统提示的用户时区

设置 `agents.defaults.userTimezone` 以告知模型用户的本地时区。如果未设置，OpenClaw 会在运行时解析**主机时区**（无需写入配置）。

```json
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

系统提示包含：

-   `当前日期和时间` 部分，包含本地时间和时区
-   `时间格式：12小时制` 或 `24小时制`

您可以通过 `agents.defaults.timeFormat`（`auto` | `12` | `24`）控制提示格式。有关完整行为和示例，请参阅[日期与时间](../date-time.md)。

[使用情况跟踪](./usage-tracking.md)[积分](../reference/credits.md)