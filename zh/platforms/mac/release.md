

  macOS 配套应用

  
# macOS 发布

此应用现已支持 Sparkle 自动更新。发布版本必须使用开发者 ID 签名、压缩，并随附签名的 appcast 条目一同发布。

## 先决条件

-   已安装开发者 ID 应用程序证书（例如：`Developer ID Application:  ()`）。
-   Sparkle 私钥路径已在环境中设置为 `SPARKLE_PRIVATE_KEY_FILE`（指向你的 Sparkle ed25519 私钥的路径；公钥已嵌入 Info.plist）。如果缺失，请检查 `~/.profile`。
-   用于 `xcrun notarytool` 的公证凭据（钥匙串配置文件或 API 密钥），以便进行 Gatekeeper 安全的 DMG/zip 分发。
    -   我们使用名为 `openclaw-notary` 的钥匙串配置文件，该文件根据你的 shell 配置文件中的 App Store Connect API 密钥环境变量创建：
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   已安装 `pnpm` 依赖项 (`pnpm install --config.node-linker=hoisted`)。
-   Sparkle 工具已通过 SwiftPM 自动获取到 `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` 目录下（包含 `sign_update`, `generate_appcast` 等）。

## 构建与打包

注意事项：

-   `APP_BUILD` 映射到 `CFBundleVersion`/`sparkle:version`；保持其为数字且单调递增（不要包含 `-beta`），否则 Sparkle 会将其视为相等版本。
-   如果省略 `APP_BUILD`，`scripts/package-mac-app.sh` 会从 `APP_VERSION` 派生一个 Sparkle 安全的默认值（`YYYYMMDDNN`：稳定版默认为 `90`，预发布版本使用后缀派生的通道），并使用该值与 git 提交计数中较高的一个。
-   当发布工程需要特定的单调递增值时，你仍然可以显式地覆盖 `APP_BUILD`。
-   默认为当前架构 (`$(uname -m)`)。对于发布/通用构建，请设置 `BUILD_ARCHS="arm64 x86_64"`（或 `BUILD_ARCHS=all`）。
-   使用 `scripts/package-mac-dist.sh` 生成发布工件（zip + DMG + 公证）。使用 `scripts/package-mac-app.sh` 进行本地/开发打包。

```bash
# 从仓库根目录执行；设置发布 ID 以便启用 Sparkle 源。
# APP_BUILD 必须是数字且单调递增，以便 Sparkle 进行比较。
# 省略时，默认值从 APP_VERSION 自动派生。
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# 为分发创建压缩包（包含资源分支以支持 Sparkle 增量更新）
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# 可选：同时为普通用户构建一个带样式的 DMG（拖拽到 /Applications）
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# 推荐：构建 + 公证/装订 zip + DMG
# 首先，创建一次钥匙串配置文件：
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# 可选：随发布版本一同提供 dSYM 文件
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## Appcast 条目

使用发布说明生成器，以便 Sparkle 渲染格式化的 HTML 说明：

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

从 `CHANGELOG.md` 生成 HTML 发布说明（通过 [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)）并将其嵌入 appcast 条目。发布时，将更新后的 `appcast.xml` 与发布资产（zip + dSYM）一同提交。

## 发布与验证

-   将 `OpenClaw-2026.3.7.zip`（以及 `OpenClaw-2026.3.7.dSYM.zip`）上传到标签为 `v2026.3.7` 的 GitHub 发布页面。
-   确保原始 appcast URL 与嵌入的源匹配：`https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`。
-   完整性检查：
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` 返回 200。
    -   资产上传后，`curl -I ` 返回 200。
    -   在之前的公开构建版本上，从“关于”选项卡运行“检查更新…”，并验证 Sparkle 能干净地安装新构建。

完成定义：已发布签名的应用和 appcast，从已安装的旧版本可以正常进行更新流程，并且发布资产已附加到 GitHub 发布页面。

[macOS 签名](./signing.md)[macOS 上的网关](./bundled-gateway.md)