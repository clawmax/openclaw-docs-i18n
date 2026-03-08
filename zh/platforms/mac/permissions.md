title: "OpenClaw 稳定应用授权的 macOS 权限指南"
description: "了解如何为 OpenClaw 维持稳定的 macOS 权限。修复消失的提示并重置辅助功能、屏幕录制和文件访问的 TCC。"
keywords: ["macos 权限", "tcc 重置", "辅助功能授权", "屏幕录制", "捆绑标识符", "代码签名", "openclaw mac", "隐私安全"]
---

  macOS 配套应用

  
# macOS 权限

macOS 权限授权是脆弱的。TCC 将权限授权与应用的代码签名、捆绑标识符和磁盘路径关联。如果其中任何一项发生更改，macOS 会将应用视为新应用，并可能丢弃或隐藏提示。

## 稳定权限的要求

-   相同路径：从固定位置运行应用（对于 OpenClaw，是 `dist/OpenClaw.app`）。
-   相同的捆绑标识符：更改捆绑 ID 会创建新的权限身份。
-   已签名的应用：未签名或临时签名的构建不会持久保存权限。
-   一致的签名：使用真实的 Apple 开发或开发者 ID 证书，以便签名在多次重建中保持稳定。

临时签名每次构建都会生成新的身份。macOS 将忘记之前的授权，并且提示可能完全消失，直到清除过时的条目。

## 提示消失时的恢复清单

1.  退出应用。
2.  在系统设置 -> 隐私与安全性中移除应用条目。
3.  从相同路径重新启动应用并重新授予权限。
4.  如果提示仍未出现，使用 `tccutil` 重置 TCC 条目并重试。
5.  某些权限只有在完全重启 macOS 后才会重新出现。

重置示例（根据需要替换捆绑 ID）：

```bash
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## 文件和文件夹权限（桌面/文稿/下载）

macOS 也可能对终端/后台进程限制桌面、文稿和下载文件夹的访问。如果文件读取或目录列表挂起，请向执行文件操作的相同进程上下文授予访问权限（例如 Terminal/iTerm、LaunchAgent 启动的应用或 SSH 进程）。变通方法：如果您想避免按文件夹授权，请将文件移动到 OpenClaw 工作区 (`~/.openclaw/workspace`)。如果您正在测试权限，请始终使用真实证书签名。临时构建仅适用于权限无关紧要的快速本地运行。

[macOS 日志记录](./logging.md)[远程控制](./remote.md)

---