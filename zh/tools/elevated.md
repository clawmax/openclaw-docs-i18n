

  内置工具

  
# 高权限模式

## 功能说明

-   `/elevated on` 在网关主机上运行，并保留执行审批（与 `/elevated ask` 相同）。
-   `/elevated full` 在网关主机上运行 **并且** 自动批准执行（跳过执行审批）。
-   `/elevated ask` 在网关主机上运行，但保留执行审批（与 `/elevated on` 相同）。
-   `on`/`ask` **不会**强制设置 `exec.security=full`；配置的安全/询问策略仍然适用。
-   仅当代理处于**沙盒**状态时改变行为（否则执行已在主机上运行）。
-   指令形式：`/elevated on|off|ask|full`，`/elev on|off|ask|full`。
-   仅接受 `on|off|ask|full`；其他任何输入将返回提示且不改变状态。

## 控制范围（及不控制的范围）

-   **可用性门控**：`tools.elevated` 是全局基线。`agents.list[].tools.elevated` 可以进一步按代理限制高权限（两者都必须允许）。
-   **每会话状态**：`/elevated on|off|ask|full` 为当前会话密钥设置高权限级别。
-   **行内指令**：消息内的 `/elevated on|ask|full` 仅适用于该条消息。
-   **群组**：在群聊中，仅当提及代理时，高权限指令才会被采纳。绕过提及要求的纯命令消息被视为已提及。
-   **主机执行**：高权限强制 `exec` 在网关主机上运行；`full` 还会设置 `security=full`。
-   **审批**：`full` 跳过执行审批；`on`/`ask` 在允许列表/询问规则要求时遵守审批。
-   **非沙盒代理**：对位置无影响；仅影响门控、日志记录和状态。
-   **工具策略仍然适用**：如果 `exec` 被工具策略拒绝，则无法使用高权限模式。
-   **与 `/exec` 分离**：`/exec` 为授权发送者调整每会话默认值，且不需要高权限。

## 解析顺序

1.  消息上的行内指令（仅适用于该消息）。
2.  会话覆盖（通过发送仅包含指令的消息设置）。
3.  全局默认值（配置中的 `agents.defaults.elevatedDefault`）。

## 设置会话默认值

-   发送一条**仅包含**指令的消息（允许有空白字符），例如 `/elevated full`。
-   会发送确认回复（`Elevated mode set to full...` / `Elevated mode disabled.`）。
-   如果高权限访问被禁用或发送者不在批准的允许列表中，指令将回复一个可操作的错误，且不会改变会话状态。
-   发送不带参数的 `/elevated`（或 `/elevated:`）以查看当前的高权限级别。

## 可用性 + 允许列表

-   功能门控：`tools.elevated.enabled`（即使代码支持，默认也可以通过配置关闭）。
-   发送者允许列表：`tools.elevated.allowFrom` 包含每个提供商的允许列表（例如 `discord`、`whatsapp`）。
-   无前缀的允许列表条目仅匹配发送者范围内的身份值（`SenderId`、`SenderE164`、`From`）；接收者路由字段从不用于高权限授权。
-   可变的发送者元数据需要显式前缀：
    -   `name:` 匹配 `SenderName`
    -   `username:` 匹配 `SenderUsername`
    -   `tag:` 匹配 `SenderTag`
    -   `id:`、`from:`、`e164:` 可用于显式身份定位
-   每代理门控：`agents.list[].tools.elevated.enabled`（可选；只能进一步限制）。
-   每代理允许列表：`agents.list[].tools.elevated.allowFrom`（可选；设置后，发送者必须匹配**全局和每代理**两个允许列表）。
-   Discord 回退：如果省略 `tools.elevated.allowFrom.discord`，则使用 `channels.discord.allowFrom` 列表作为回退（旧版：`channels.discord.dm.allowFrom`）。设置 `tools.elevated.allowFrom.discord`（即使是 `[]`）以覆盖。每代理允许列表**不**使用此回退。
-   所有门控都必须通过；否则高权限被视为不可用。

## 日志记录 + 状态

-   高权限执行调用在信息级别记录。
-   会话状态包含高权限模式（例如 `elevated=ask`、`elevated=full`）。

[PDF 工具](./pdf.md)[执行工具](./exec.md)

---