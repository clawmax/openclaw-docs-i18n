

  macOS 配套应用

  
# macOS 日志记录

## 滚动诊断文件日志（调试面板）

OpenClaw 通过 swift-log（默认使用统一日志记录）路由 macOS 应用日志，并可在需要持久捕获时，将本地滚动文件日志写入磁盘。

-   详细级别：**调试面板 → 日志 → 应用日志记录 → 详细级别**
-   启用：**调试面板 → 日志 → 应用日志记录 → “写入滚动诊断日志 (JSONL)”**
-   位置：`~/Library/Logs/OpenClaw/diagnostics.jsonl`（自动滚动；旧文件后缀为 `.1`、`.2`……）
-   清除：**调试面板 → 日志 → 应用日志记录 → “清除”**

注意事项：

-   此功能**默认关闭**。仅在主动调试时启用。
-   请将此文件视为敏感文件；未经审查请勿分享。

## macOS 统一日志记录中的私有数据

统一日志记录会隐藏大多数负载，除非子系统选择加入 `privacy -off`。根据 Peter 关于 macOS [日志记录隐私问题](https://steipete.me/posts/2025/logging-privacy-shenanigans)（2025 年）的文章，这由 `/Library/Preferences/Logging/Subsystems/` 目录中按子系统名称键控的 plist 文件控制。只有新的日志条目会获取该标志，因此请在复现问题前启用它。

## 为 OpenClaw (ai.openclaw) 启用

-   首先将 plist 写入临时文件，然后以 root 身份原子化安装：

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

-   无需重启；logd 会快速注意到该文件，但只有新的日志行才会包含私有负载。
-   使用现有的辅助工具查看更丰富的输出，例如 `./scripts/clawlog.sh --category WebChat --last 5m`。

## 调试后禁用

-   移除覆盖文件：`sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`。
-   可选运行 `sudo log config --reload` 以强制 logd 立即丢弃覆盖。
-   请注意，此记录表面可能包含电话号码和消息正文；仅在您确实需要额外详细信息时保留该 plist 文件。

[菜单栏图标](./icon.md)[macOS 权限](./permissions.md)