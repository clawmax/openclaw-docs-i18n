

  CLI 命令

  
# logs

通过 RPC 实时跟踪网关文件日志（在远程模式下工作）。相关链接：

-   日志记录概述：[日志记录](../logging.md)

## 示例

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

使用 `--local-time` 以在您的本地时区呈现时间戳。

[hooks](./hooks.md)[memory](./memory.md)

---