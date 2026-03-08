

  CLI 命令

  
# agent

通过网关运行一个代理回合（使用 `--local` 参数进行嵌入式运行）。使用 `--agent ` 直接指定一个已配置的代理。相关：

-   代理发送工具：[Agent send](../tools/agent-send.md)

## 示例

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## 注意

-   当此命令触发 `models.json` 重新生成时，由 SecretRef 管理的提供商凭证将作为非机密标记（例如环境变量名或 `secretref-managed`）持久化，而不是解析后的机密明文。

[acp](./acp.md)[agents](./agents.md)

---