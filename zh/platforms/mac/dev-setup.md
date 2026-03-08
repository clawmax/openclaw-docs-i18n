

  macOS 配套应用

  
# macOS 开发环境设置

本指南涵盖了从源码构建和运行 OpenClaw macOS 应用程序的必要步骤。

## 先决条件

在构建应用之前，请确保已安装以下内容：

1.  **Xcode 26.2+**：Swift 开发所必需。
2.  **Node.js 22+ 和 pnpm**：网关、CLI 和打包脚本所必需。

## 1\. 安装依赖项

安装项目范围的依赖项：

```bash
pnpm install
```

## 2\. 构建并打包应用

要构建 macOS 应用并将其打包到 `dist/OpenClaw.app`，请运行：

```
./scripts/package-mac-app.sh
```

如果您没有 Apple 开发者 ID 证书，脚本将自动使用**临时签名** (`-`)。有关开发运行模式、签名标志和 Team ID 故障排除，请参阅 macOS 应用 README：[https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **注意**：临时签名的应用可能会触发安全提示。如果应用立即崩溃并显示“Abort trap 6”，请参阅[故障排除](#troubleshooting)部分。

## 3\. 安装 CLI

macOS 应用需要一个全局安装的 `openclaw` CLI 来管理后台任务。**安装方法（推荐）：**

1.  打开 OpenClaw 应用。
2.  转到 **通用** 设置选项卡。
3.  点击 **“安装 CLI”**。

或者，手动安装：

```bash
npm install -g openclaw@<version>
```

## 故障排除

### 构建失败：工具链或 SDK 不匹配

macOS 应用构建需要最新的 macOS SDK 和 Swift 6.2 工具链。**系统依赖项（必需）：**

-   **软件更新中可用的最新 macOS 版本**（Xcode 26.2 SDK 所要求）
-   **Xcode 26.2**（Swift 6.2 工具链）

**检查：**

```bash
xcodebuild -version
xcrun swift --version
```

如果版本不匹配，请更新 macOS/Xcode 并重新运行构建。

### 授予权限时应用崩溃

如果您在尝试允许**语音识别**或**麦克风**访问时应用崩溃，可能是由于 TCC 缓存损坏或签名不匹配。**修复方法：**

1.  重置 TCC 权限：
    
    复制
    
    ```bash
    tccutil reset All ai.openclaw.mac.debug
    ```
    
2.  如果上述方法失败，请临时更改 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 中的 `BUNDLE_ID`，以强制 macOS 从“干净状态”开始。

### 网关无限期显示“正在启动…”

如果网关状态一直停留在“正在启动…”，请检查是否有僵尸进程占用了端口：

```bash
openclaw gateway status
openclaw gateway stop

# 如果您没有使用 LaunchAgent（开发模式/手动运行），请查找监听器：
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

如果是手动运行占用了端口，请停止该进程 (Ctrl+C)。作为最后的手段，终止上面找到的 PID。

[Raspberry Pi](../raspberry-pi.md)[菜单栏](./menu-bar.md)