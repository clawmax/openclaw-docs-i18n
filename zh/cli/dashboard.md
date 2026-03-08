

  CLI 命令

  
# dashboard

使用您当前的认证信息打开控制 UI。

```bash
openclaw dashboard
openclaw dashboard --no-open
```

注意事项：

-   `dashboard` 命令会尽可能解析配置的 `gateway.auth.token` SecretRefs。
-   对于由 SecretRef 管理的令牌（无论是否已解析），`dashboard` 会打印/复制/打开一个非令牌化的 URL，以避免在终端输出、剪贴板历史记录或浏览器启动参数中暴露外部密钥。
-   如果 `gateway.auth.token` 由 SecretRef 管理但在此命令路径中未解析，该命令将打印一个非令牌化的 URL 和明确的修复指导，而不是嵌入一个无效的令牌占位符。

[daemon](./daemon.md)[devices](./devices.md)