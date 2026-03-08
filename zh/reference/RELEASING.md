

  发布说明

  
# 发布清单

在仓库根目录使用 `pnpm` (Node 22+)。在打标签/发布前保持工作树干净。

## 操作员触发

当操作员说“发布”时，立即执行此预检（除非受阻，无需额外提问）：

-   阅读本文档和 `docs/platforms/mac/release.md`。
-   从 `~/.profile` 加载环境变量，并确认 `SPARKLE_PRIVATE_KEY_FILE` 和 App Store Connect 变量已设置（SPARKLE\_PRIVATE\_KEY\_FILE 应位于 `~/.profile` 中）。
-   如果需要，使用来自 `~/Library/CloudStorage/Dropbox/Backup/Sparkle` 的 Sparkle 密钥。

1.  **版本与元数据**

-   [ ] 提升 `package.json` 版本号（例如，`2026.1.29`）。
-   [ ] 运行 `pnpm plugins:sync` 以同步扩展包版本和更新日志。
-   [ ] 更新 [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) 中的 CLI/版本字符串以及 [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts) 中的 Baileys 用户代理。
-   [ ] 确认包元数据（名称、描述、仓库、关键词、许可证）以及 `bin` 映射指向 `openclaw` 的 [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)。
-   [ ] 如果依赖项有变更，运行 `pnpm install` 以确保 `pnpm-lock.yaml` 是最新的。

2.  **构建与制品**

-   [ ] 如果 A2UI 输入有变更，运行 `pnpm canvas:a2ui:bundle` 并提交任何更新后的 [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js)。
-   [ ] `pnpm run build`（重新生成 `dist/`）。
-   [ ] 验证 npm 包的 `files` 字段包含所有必需的 `dist/*` 文件夹（特别是用于无头节点 + ACP CLI 的 `dist/node-host/**` 和 `dist/acp/**`）。
-   [ ] 确认 `dist/build-info.json` 存在并包含预期的 `commit` 哈希（CLI 横幅在 npm 安装时使用此信息）。
-   [ ] 可选：构建后运行 `npm pack --pack-destination /tmp`；检查 tarball 内容并准备好用于 GitHub 发布（**不要**提交它）。

3.  **更新日志与文档**

-   [ ] 使用面向用户的亮点更新 `CHANGELOG.md`（如果文件缺失则创建）；条目严格按版本降序排列。
-   [ ] 确保 README 中的示例/标志与当前 CLI 行为匹配（特别是新命令或选项）。

4.  **验证**

-   [ ] `pnpm build`
-   [ ] `pnpm check`
-   [ ] `pnpm test`（或如果需要覆盖率输出，则运行 `pnpm test:coverage`）
-   [ ] `pnpm release:check`（验证 npm pack 内容）
-   [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke`（Docker 安装冒烟测试，快速路径；发布前必需）
    -   如果已知上一个 npm 发布版本有问题，为预安装步骤设置 `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` 或 `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1`。
-   [ ] （可选）完整安装程序冒烟测试（增加非 root + CLI 覆盖）：`pnpm test:install:smoke`
-   [ ] （可选）安装程序端到端测试（Docker，运行 `curl -fsSL https://openclaw.ai/install.sh | bash`，引导，然后运行真实工具调用）：
    -   `pnpm test:install:e2e:openai`（需要 `OPENAI_API_KEY`）
    -   `pnpm test:install:e2e:anthropic`（需要 `ANTHROPIC_API_KEY`）
    -   `pnpm test:install:e2e`（需要两个密钥；运行两个提供商）
-   [ ] （可选）如果您的更改影响发送/接收路径，请点检 Web 网关。

5.  **macOS 应用 (Sparkle)**

-   [ ] 构建 + 签名 macOS 应用，然后压缩以供分发。
-   [ ] 生成 Sparkle appcast（通过 [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) 生成 HTML 说明）并更新 `appcast.xml`。
-   [ ] 准备好应用 zip 文件（以及可选的 dSYM zip 文件）以附加到 GitHub 发布。
-   [ ] 遵循 [macOS 发布](../platforms/mac/release.md) 获取确切的命令和所需的环境变量。
    -   `APP_BUILD` 必须是数字且单调递增（不能有 `-beta`），以便 Sparkle 正确比较版本。
    -   如果要公证，请使用从 App Store Connect API 环境变量创建的 `openclaw-notary` 钥匙串配置文件（参见 [macOS 发布](../platforms/mac/release.md)）。

6.  **发布 (npm)**

-   [ ] 确认 git 状态是干净的；根据需要提交和推送。
-   [ ] 如果需要，运行 `npm login`（验证 2FA）。
-   [ ] `npm publish --access public`（预发布版本使用 `--tag beta`）。
-   [ ] 验证注册表：`npm view openclaw version`，`npm view openclaw dist-tags`，以及 `npx -y openclaw@X.Y.Z --version`（或 `--help`）。

### 故障排除（来自 2.0.0-beta2 发布的笔记）

-   **npm pack/publish 挂起或产生巨大的 tarball**：`dist/OpenClaw.app` 中的 macOS 应用包（以及发布 zip 文件）被扫入包中。通过 `package.json` 中的 `files` 字段白名单发布内容来修复（包含 dist 子目录、文档、技能；排除应用包）。使用 `npm pack --dry-run` 确认 `dist/OpenClaw.app` 未被列出。
-   **dist-tags 的 npm auth web 循环**：使用旧版身份验证以获取 OTP 提示：
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **`npx` 验证失败，提示 `ECOMPROMISED: Lock compromised`**：使用新缓存重试：
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **在后期修复后需要重新指向标签**：强制更新并推送标签，然后确保 GitHub 发布资产仍然匹配：
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **GitHub 发布 + appcast**

-   [ ] 打标签并推送：`git tag vX.Y.Z && git push origin vX.Y.Z`（或 `git push --tags`）。
-   [ ] 为 `vX.Y.Z` 创建/刷新 GitHub 发布，**标题为 `openclaw X.Y.Z`**（不仅仅是标签）；正文应包含该版本的**完整**更新日志部分（亮点 + 变更 + 修复），内联（不要只有链接），并且**不得在正文中重复标题**。
-   [ ] 附加制品：`npm pack` tarball（可选）、`OpenClaw-X.Y.Z.zip` 和 `OpenClaw-X.Y.Z.dSYM.zip`（如果已生成）。
-   [ ] 提交更新后的 `appcast.xml` 并推送它（Sparkle 从 main 分支获取）。
-   [ ] 从一个干净的临时目录（没有 `package.json`）运行 `npx -y openclaw@X.Y.Z send --help` 以确认安装/CLI 入口点正常工作。
-   [ ] 宣布/分享发布说明。

## 插件发布范围 (npm)

我们只发布 `@openclaw/*` 作用域下的**现有 npm 插件**。未在 npm 上的捆绑插件保持**仅磁盘树**（仍随 `extensions/**` 分发）。推导列表的流程：

1.  `npm search @openclaw --json` 并捕获包名。
2.  与 `extensions/*/package.json` 中的名称进行比较。
3.  仅发布**交集**（已在 npm 上的）。

当前 npm 插件列表（根据需要更新）：

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

发布说明还必须指出**新的可选捆绑插件**，这些插件**默认不启用**（例如：`tlon`）。

[致谢](./credits.md)[测试](./test.md)