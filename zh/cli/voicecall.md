

  CLI 命令

  
# voicecall

`voicecall` 是一个由插件提供的命令。仅当语音通话插件已安装并启用时才会出现。主要文档：

-   语音通话插件：[语音通话](../plugins/voice-call.md)

## 常用命令

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## 暴露 Webhook (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

安全提示：仅将 Webhook 端点暴露给您信任的网络。在可能的情况下，优先使用 Tailscale Serve 模式而非 Funnel 模式。

[更新](./update.md)[webhooks](./webhooks.md)

---