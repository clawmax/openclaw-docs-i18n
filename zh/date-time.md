

  技术参考

  
# 日期与时间

OpenClaw 默认使用**主机本地时间作为传输时间戳**，并且**仅在系统提示中使用用户时区**。供应商时间戳会被保留，以便工具保持其原生语义（当前时间可通过 `session_status` 获取）。

## 消息信封（默认本地时间）

传入的消息会附带一个时间戳（精确到分钟）进行包装：

```
[Provider ... 2026-01-05 16:26 PST] 消息文本
```

此信封时间戳**默认使用主机本地时间**，与供应商时区无关。您可以覆盖此行为：

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
-   `envelopeTimezone: "local"` 使用主机时区。
-   `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（回退到主机时区）。
-   使用明确的 IANA 时区（例如 `"America/Chicago"`）来指定固定时区。
-   `envelopeTimestamp: "off"` 从信封头部移除绝对时间戳。
-   `envelopeElapsed: "off"` 移除经过时间后缀（`+2m` 这种样式）。

### 示例

**本地时间（默认）：**

```
[WhatsApp +1555 2026-01-18 00:19 PST] 你好
```

**用户时区：**

```
[WhatsApp +1555 2026-01-18 00:19 CST] 你好
```

**启用经过时间：**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] 后续跟进
```

## 系统提示：当前日期与时间

如果已知用户时区，系统提示会包含一个专门的**当前日期与时间**部分，其中仅包含**时区**（不包含时钟/时间格式），以保持提示缓存的稳定性：

```bash
时区：America/Chicago
```

当代理需要当前时间时，请使用 `session_status` 工具；状态卡片包含一个时间戳行。

## 系统事件行（默认本地时间）

插入到代理上下文中的排队系统事件会带有一个时间戳前缀，其时区选择与消息信封相同（默认：主机本地时间）。

```yaml
系统：[2026-01-12 12:19:17 PST] 模型已切换。
```

### 配置用户时区 + 格式

```json
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

-   `userTimezone` 设置提示上下文中使用的**用户本地时区**。
-   `timeFormat` 控制提示中**12小时制/24小时制**的显示。`auto` 遵循操作系统偏好设置。

## 时间格式检测（自动）

当 `timeFormat: "auto"` 时，OpenClaw 会检查操作系统偏好设置（macOS/Windows）并回退到区域格式。检测到的值**按进程缓存**，以避免重复的系统调用。

## 工具负载 + 连接器（原始供应商时间 + 标准化字段）

通道工具返回**供应商原生时间戳**，并添加标准化字段以确保一致性：

-   `timestampMs`：纪元毫秒数（UTC）
-   `timestampUtc`：ISO 8601 UTC 字符串

原始供应商字段会被保留，因此不会丢失任何信息。

-   Slack：来自 API 的类纪元字符串
-   Discord：UTC ISO 时间戳
-   Telegram/WhatsApp：供应商特定的数字/ISO 时间戳

如果需要本地时间，请使用已知时区在下游进行转换。

## 相关文档

-   [系统提示](./concepts/system-prompt.md)
-   [时区](./concepts/timezone.md)
-   [消息](./concepts/messages.md)

[记录整理](./reference/transcript-hygiene.md)[TypeBox](./concepts/typebox.md)

---