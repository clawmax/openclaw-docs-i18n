

  配置与操作

  
# 日志

关于面向用户的概述（CLI + 控制台 UI + 配置），请参阅 [/logging](../logging.md)。OpenClaw 有两个日志“表面”：

-   **终端输出**（你在终端 / 调试 UI 中看到的内容）。
-   **文件日志**（JSON 行），由网关记录器写入。

## 基于文件的记录器

-   默认的滚动日志文件位于 `/tmp/openclaw/` 下（每天一个文件）：`openclaw-YYYY-MM-DD.log`
    -   日期使用网关主机的本地时区。
-   日志文件路径和级别可以通过 `~/.openclaw/openclaw.json` 配置：
    -   `logging.file`
    -   `logging.level`

文件格式为每行一个 JSON 对象。控制台 UI 的日志选项卡通过网关 (`logs.tail`) 跟踪此文件。CLI 也可以执行相同操作：

```bash
openclaw logs --follow
```

**详细模式与日志级别**

-   **文件日志** 完全由 `logging.level` 控制。
-   `--verbose` 仅影响 **终端详细程度**（以及 WS 日志样式）；它**不会**提高文件日志级别。
-   要在文件日志中捕获仅在详细模式下显示的详细信息，请将 `logging.level` 设置为 `debug` 或 `trace`。

## 终端捕获

CLI 捕获 `console.log/info/warn/error/debug/trace` 并将其写入文件日志，同时仍打印到 stdout/stderr。你可以通过以下方式独立调整终端详细程度：

-   `logging.consoleLevel` (默认 `info`)
-   `logging.consoleStyle` (`pretty` | `compact` | `json`)

## 工具摘要脱敏

详细的工具摘要（例如 `🛠️ Exec: ...`）可以在其进入终端流之前屏蔽敏感令牌。这**仅适用于工具**，不会更改文件日志。

-   `logging.redactSensitive`: `off` | `tools` (默认: `tools`)
-   `logging.redactPatterns`: 正则表达式字符串数组（覆盖默认值）
    -   使用原始正则表达式字符串（自动添加 `gi`），或者如果需要自定义标志，则使用 `/pattern/flags`。
    -   匹配项通过保留前 6 个和后 4 个字符（长度 >= 18）进行屏蔽，否则显示为 `***`。
    -   默认值涵盖常见的键值分配、CLI 标志、JSON 字段、Bearer 头、PEM 块以及流行的令牌前缀。

## 网关 WebSocket 日志

网关以两种模式打印 WebSocket 协议日志：

-   **普通模式（无 `--verbose`）**：仅打印“有趣”的 RPC 结果：
    -   错误 (`ok=false`)
    -   慢调用（默认阈值：`>= 50ms`）
    -   解析错误
-   **详细模式（`--verbose`）**：打印所有 WS 请求/响应流量。

### WS 日志样式

`openclaw gateway` 支持每个网关的样式切换：

-   `--ws-log auto` (默认)：普通模式已优化；详细模式使用紧凑输出
-   `--ws-log compact`：详细模式下使用紧凑输出（配对的请求/响应）
-   `--ws-log full`：详细模式下使用完整每帧输出
-   `--compact`：`--ws-log compact` 的别名

示例：

```bash
# 优化模式（仅错误/慢调用）
openclaw gateway

# 显示所有 WS 流量（配对）
openclaw gateway --verbose --ws-log compact

# 显示所有 WS 流量（完整元数据）
openclaw gateway --verbose --ws-log full
```

## 终端格式化（子系统日志记录）

终端格式化器是**TTY感知的**，并打印一致的、带前缀的行。子系统记录器保持输出分组且易于扫描。行为：

-   **子系统前缀** 在每一行上（例如 `[gateway]`, `[canvas]`, `[tailscale]`）
-   **子系统颜色**（每个子系统稳定）加上级别颜色
-   **当输出是 TTY 或环境看起来像富终端时使用颜色**（`TERM`/`COLORTERM`/`TERM_PROGRAM`），尊重 `NO_COLOR`
-   **缩短的子系统前缀**：删除开头的 `gateway/` + `channels/`，保留最后 2 个部分（例如 `whatsapp/outbound`）
-   **按子系统划分的子记录器**（自动前缀 + 结构化字段 `{ subsystem }`）
-   **`logRaw()`** 用于 QR/UX 输出（无前缀，无格式化）
-   **终端样式**（例如 `pretty | compact | json`）
-   **终端日志级别** 与文件日志级别分开（当 `logging.level` 设置为 `debug`/`trace` 时，文件保留完整细节）
-   **WhatsApp 消息正文** 在 `debug` 级别记录（使用 `--verbose` 查看它们）

这保持了现有文件日志的稳定性，同时使交互式输出易于扫描。

[诊断](./doctor.md)[网关锁](./gateway-lock.md)

---