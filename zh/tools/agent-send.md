

  智能体协调

  
# Agent Send

`openclaw agent` 无需传入聊天消息即可运行单次智能体轮次。默认情况下，它会**通过网关**运行；添加 `--local` 参数可强制在当前机器上使用嵌入式运行时。

## 行为

-   必需：`--message <文本>`
-   会话选择：
    -   `--to <目标>` 派生会话密钥（群组/频道目标保持隔离；直接聊天合并为 `main`），**或**
    -   `--session-id ` 按 id 重用现有会话，**或**
    -   `--agent ` 直接定位到已配置的智能体（使用该智能体的 `main` 会话密钥）
-   运行与正常传入回复相同的嵌入式智能体运行时。
-   思考/详细标志会持久化到会话存储中。
-   输出：
    -   默认：打印回复文本（加上 `MEDIA:` 行）
    -   `--json`：打印结构化负载 + 元数据
-   可选地使用 `--deliver` + `--channel` 将回复交付回某个渠道（目标格式与 `openclaw message --target` 匹配）。
-   使用 `--reply-channel`/`--reply-to`/`--reply-account` 来覆盖交付目标，而无需更改会话。

如果网关无法访问，CLI 将**回退**到嵌入式本地运行。

## 示例

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## 标志

-   `--local`：本地运行（需要 shell 中的模型提供商 API 密钥）
-   `--deliver`：将回复发送到所选渠道
-   `--channel`：交付渠道 (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`，默认：`whatsapp`)
-   `--reply-to`：交付目标覆盖
-   `--reply-channel`：交付渠道覆盖
-   `--reply-account`：交付账户 ID 覆盖
-   `--thinking <off|minimal|low|medium|high|xhigh>`：持久化思考级别（仅限 GPT-5.2 + Codex 模型）
-   `--verbose <on|full|off>`：持久化详细级别
-   `--timeout <秒>`：覆盖智能体超时时间
-   `--json`：输出结构化 JSON

[浏览器故障排除](./browser-linux-troubleshooting.md)[子智能体](./subagents.md)

---