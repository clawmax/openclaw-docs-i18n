

  CLI 命令

  
# tui

打开连接到网关的终端用户界面。相关：

-   TUI 指南：[TUI](../web/tui.md)

注意：

-   `tui` 在可能的情况下会解析配置的网关身份验证 SecretRefs 以进行令牌/密码身份验证（`env`/`file`/`exec` 提供程序）。

## 示例

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)

---