

  CLI 命令

  
# pairing

批准或检查 DM 配对请求（适用于支持配对的频道）。相关：

-   配对流程：[配对](../channels/pairing.md)

## 命令

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## 说明

-   频道输入：可以按位置传递 (`pairing list telegram`) 或使用 `--channel `。
-   `pairing list` 支持 `--account ` 用于多账户频道。
-   `pairing approve` 支持 `--account ` 和 `--notify`。
-   如果只配置了一个支持配对的频道，则允许使用 `pairing approve `。

[onboard](./onboard.md)[plugins](./plugins.md)

---