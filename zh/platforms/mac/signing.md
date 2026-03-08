

  macOS 配套应用

  
# macOS 签名

此应用通常由 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 构建，该脚本现在：

-   设置一个稳定的调试包标识符：`ai.openclaw.mac.debug`
-   使用该包标识符写入 Info.plist（可通过 `BUNDLE_ID=...` 覆盖）
-   调用 [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) 来签名主二进制文件和应用程序包，以便 macOS 将每次重新构建视为同一个已签名的包，并保持 TCC 权限（通知、辅助功能、屏幕录制、麦克风、语音）。要获得稳定的权限，请使用真实的签名身份；临时签名是可选加入且不稳定的（参见 [macOS 权限](./permissions.md)）。
-   默认使用 `CODESIGN_TIMESTAMP=auto`；它为开发者 ID 签名启用可信时间戳。设置 `CODESIGN_TIMESTAMP=off` 以跳过时间戳（离线调试构建）。
-   将构建元数据注入 Info.plist：`OpenClawBuildTimestamp`（UTC）和 `OpenClawGitCommit`（短哈希），以便“关于”面板可以显示构建、git 和调试/发布渠道信息。
-   **打包需要 Node 22+**：脚本会运行 TS 构建和 Control UI 构建。
-   从环境中读取 `SIGN_IDENTITY`。将 `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（或您的开发者 ID 应用程序证书）添加到您的 shell rc 文件中，以便始终使用您的证书签名。临时签名需要通过 `ALLOW_ADHOC_SIGNING=1` 或 `SIGN_IDENTITY="-"` 明确选择加入（不推荐用于权限测试）。
-   签名后运行团队 ID 审计，如果应用程序包内的任何 Mach-O 文件由不同的团队 ID 签名，则失败。设置 `SKIP_TEAM_ID_CHECK=1` 以绕过。

## 使用方法

```bash
# 从仓库根目录
scripts/package-mac-app.sh               # 自动选择身份；如果未找到则报错
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # 真实证书
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # 临时签名（权限将无法保持）
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # 明确临时签名（相同注意事项）
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 仅限开发使用的 Sparkle 团队 ID 不匹配变通方案
```

### 临时签名说明

当使用 `SIGN_IDENTITY="-"`（临时）签名时，脚本会自动禁用**强化运行时**（`--options runtime`）。这是为了防止应用尝试加载不共享相同团队 ID 的嵌入式框架（如 Sparkle）时发生崩溃所必需的。临时签名也会破坏 TCC 权限的持久性；恢复步骤请参见 [macOS 权限](./permissions.md)。

## 用于“关于”的构建元数据

`package-mac-app.sh` 为包标记以下信息：

-   `OpenClawBuildTimestamp`：打包时的 ISO8601 UTC 时间
-   `OpenClawGitCommit`：简短的 git 哈希值（如果不可用则为 `unknown`）

“关于”选项卡读取这些键以显示版本、构建日期、git 提交以及是否为调试构建（通过 `#if DEBUG`）。代码更改后，运行打包程序以刷新这些值。

## 原因

TCC 权限与包标识符*和*代码签名绑定。具有变化 UUID 的未签名调试构建导致 macOS 在每次重新构建后忘记授权。对二进制文件进行签名（默认为临时签名）并保持固定的包标识符/路径（`dist/OpenClaw.app`）可以在构建之间保留授权，这与 VibeTunnel 的方法一致。

[远程控制](./remote.md)[macOS 发布](./release.md)