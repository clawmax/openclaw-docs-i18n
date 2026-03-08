

  浏览器

  
# 浏览器 (OpenClaw 托管)

OpenClaw 可以运行一个**专用的 Chrome/Brave/Edge/Chromium 配置文件**，由智能体控制。它与您的个人浏览器隔离，并通过网关内部的一个小型本地控制服务（仅限环回地址）进行管理。初学者视角：

-   可以将其视为一个**独立的、仅供智能体使用的浏览器**。
-   `openclaw` 配置文件**不会**触及您的个人浏览器配置文件。
-   智能体可以在一个安全通道中**打开标签页、读取页面、点击和输入**。
-   默认的 `chrome` 配置文件通过扩展中继使用**系统默认的 Chromium 浏览器**；切换到 `openclaw` 以使用隔离的托管浏览器。

## 您将获得的功能

-   一个名为 **openclaw** 的独立浏览器配置文件（默认橙色主题）。
-   确定性的标签页控制（列出/打开/聚焦/关闭）。
-   智能体操作（点击/输入/拖拽/选择）、快照、截图、PDF 生成。
-   可选的多配置文件支持（`openclaw`、`work`、`remote`、…）。

此浏览器**并非**您的日常浏览器。它是一个用于智能体自动化和验证的安全、隔离的操作界面。

## 快速开始

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

如果遇到“浏览器已禁用”提示，请在配置中启用它（见下文）并重启网关。

## 配置文件：openclaw 与 chrome

-   `openclaw`: 托管的、隔离的浏览器（无需扩展）。
-   `chrome`: 通过扩展中继连接到您的**系统浏览器**（需要将 OpenClaw 扩展附加到标签页）。

如果您希望默认使用托管模式，请设置 `browser.defaultProfile: "openclaw"`。

## 配置

浏览器设置位于 `~/.openclaw/openclaw.json`。

```json
{
  browser: {
    enabled: true, // 默认: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // 默认信任网络模式
      // allowPrivateNetwork: true, // 旧版别名
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // 旧版单配置文件覆盖
    remoteCdpTimeoutMs: 1500, // 远程 CDP HTTP 超时（毫秒）
    remoteCdpHandshakeTimeoutMs: 3000, // 远程 CDP WebSocket 握手超时（毫秒）
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

注意事项：

-   浏览器控制服务绑定到环回地址，端口由 `gateway.port` 派生（默认：`18791`，即网关端口 + 2）。中继使用下一个端口（`18792`）。
-   如果您覆盖了网关端口（`gateway.port` 或 `OPENCLAW_GATEWAY_PORT`），派生的浏览器端口会相应调整以保持在同一“家族”中。
-   `cdpUrl` 在未设置时默认为中继端口。
-   `remoteCdpTimeoutMs` 适用于远程（非环回）CDP 可达性检查。
-   `remoteCdpHandshakeTimeoutMs` 适用于远程 CDP WebSocket 可达性检查。
-   浏览器导航/打开标签页在导航前受 SSRF 防护，并在导航后对最终的 `http(s)` URL 进行尽力而为的重新检查。
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` 默认为 `true`（信任网络模型）。将其设置为 `false` 以启用严格的仅公共网络浏览。
-   `browser.ssrfPolicy.allowPrivateNetwork` 作为旧版别名仍受支持以保持兼容性。
-   `attachOnly: true` 表示“永不启动本地浏览器；仅在其已运行时附加”。
-   `color` 和每个配置文件的 `color` 会为浏览器 UI 着色，以便您查看哪个配置文件处于活动状态。
-   默认配置文件是 `openclaw`（OpenClaw 托管的独立浏览器）。使用 `defaultProfile: "chrome"` 选择加入 Chrome 扩展中继。
-   自动检测顺序：如果系统默认浏览器基于 Chromium，则使用它；否则按 Chrome → Brave → Edge → Chromium → Chrome Canary 顺序检测。
-   本地 `openclaw` 配置文件自动分配 `cdpPort`/`cdpUrl` — 仅对远程 CDP 设置这些值。

## 使用 Brave（或其他基于 Chromium 的浏览器）

如果您的**系统默认**浏览器是基于 Chromium 的（Chrome/Brave/Edge 等），OpenClaw 会自动使用它。设置 `browser.executablePath` 以覆盖自动检测：CLI 示例：

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## 本地与远程控制

-   **本地控制（默认）：** 网关启动环回控制服务，并可启动本地浏览器。
-   **远程控制（节点主机）：** 在拥有浏览器的机器上运行节点主机；网关将浏览器操作代理到该节点。
-   **远程 CDP：** 设置 `browser.profiles..cdpUrl`（或 `browser.cdpUrl`）以附加到远程的基于 Chromium 的浏览器。在这种情况下，OpenClaw 不会启动本地浏览器。

远程 CDP URL 可以包含认证信息：

-   查询令牌（例如，`https://provider.example?token=`）
-   HTTP 基本认证（例如，`https://user:pass@provider.example`）

OpenClaw 在调用 `/json/*` 端点和连接 CDP WebSocket 时会保留认证信息。建议使用环境变量或密钥管理器来存储令牌，而不是将其提交到配置文件中。

## 节点浏览器代理（零配置默认）

如果您在拥有浏览器的机器上运行**节点主机**，OpenClaw 可以自动将浏览器工具调用路由到该节点，无需任何额外的浏览器配置。这是远程网关的默认路径。注意事项：

-   节点主机通过**代理命令**暴露其本地浏览器控制服务器。
-   配置文件来自节点自身的 `browser.profiles` 配置（与本地相同）。
-   如果您不希望启用此功能，请禁用：
    -   在节点上：`nodeHost.browserProxy.enabled=false`
    -   在网关上：`gateway.nodes.browser.mode="off"`

## Browserless（托管的远程 CDP）

[Browserless](https://browserless.io) 是一项托管的 Chromium 服务，通过 HTTPS 暴露 CDP 端点。您可以将 OpenClaw 浏览器配置文件指向 Browserless 区域端点，并使用您的 API 密钥进行身份验证。示例：

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

注意事项：

-   将 `<BROWSERLESS_API_KEY>` 替换为您真实的 Browserless 令牌。
-   选择与您的 Browserless 账户匹配的区域端点（参见其文档）。

## 安全性

关键要点：

-   浏览器控制仅限于环回地址；访问流经网关的身份验证或节点配对。
-   如果启用了浏览器控制且未配置身份验证，OpenClaw 会在启动时自动生成 `gateway.auth.token` 并持久化到配置中。
-   将网关和任何节点主机保持在私有网络（如 Tailscale）中；避免公开暴露。
-   将远程 CDP URL/令牌视为机密；建议使用环境变量或密钥管理器。

远程 CDP 提示：

-   尽可能使用 HTTPS 端点和短期令牌。
-   避免将长期令牌直接嵌入配置文件中。

## 配置文件（多浏览器）

OpenClaw 支持多个命名的配置文件（路由配置）。配置文件可以是：

-   **openclaw 托管**：一个专用的基于 Chromium 的浏览器实例，拥有自己的用户数据目录 + CDP 端口
-   **远程**：一个显式的 CDP URL（在其他地方运行的基于 Chromium 的浏览器）
-   **扩展中继**：通过本地中继 + Chrome 扩展连接到您现有的 Chrome 标签页

默认值：

-   如果缺少 `openclaw` 配置文件，则会自动创建。
-   `chrome` 配置文件是内置的，用于 Chrome 扩展中继（默认指向 `http://127.0.0.1:18792`）。
-   本地 CDP 端口默认从 **18800–18899** 分配。
-   删除配置文件会将其本地数据目录移至回收站。

所有控制端点都接受 `?profile=`；CLI 使用 `--browser-profile`。

## Chrome 扩展中继（使用您现有的 Chrome）

OpenClaw 也可以通过本地 CDP 中继 + Chrome 扩展来驱动**您现有的 Chrome 标签页**（无需单独的“openclaw” Chrome 实例）。完整指南：[Chrome 扩展](./chrome-extension.md) 流程：

-   网关在本地（同一台机器）运行，或者节点主机在浏览器机器上运行。
-   一个本地**中继服务器**在环回地址 `cdpUrl`（默认：`http://127.0.0.1:18792`）上监听。
-   您在标签页上点击 **OpenClaw Browser Relay** 扩展图标以附加（它不会自动附加）。
-   智能体通过正常的 `browser` 工具，通过选择正确的配置文件来控制该标签页。

如果网关在其他地方运行，请在浏览器机器上运行节点主机，以便网关可以代理浏览器操作。

### 沙盒会话

如果智能体会话处于沙盒中，`browser` 工具可能默认使用 `target="sandbox"`（沙盒浏览器）。Chrome 扩展中继接管需要主机浏览器控制，因此要么：

-   在非沙盒环境中运行会话，或者
-   设置 `agents.defaults.sandbox.browser.allowHostControl: true` 并在调用工具时使用 `target="host"`。

### 设置

1.  加载扩展（开发/解压模式）：

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → 启用“开发者模式”
-   “加载已解压的扩展程序” → 选择 `openclaw browser extension path` 打印的目录
-   固定扩展，然后在您想要控制的标签页上点击它（徽章显示 `ON`）。

2.  使用它：

-   CLI：`openclaw browser --browser-profile chrome tabs`
-   智能体工具：`browser` 并设置 `profile="chrome"`

可选：如果您想要不同的名称或中继端口，请创建您自己的配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

注意事项：

-   此模式依赖 Playwright-on-CDP 进行大多数操作（截图/快照/操作）。
-   再次点击扩展图标以分离。

## 隔离保证

-   **专用的用户数据目录**：绝不触及您的个人浏览器配置文件。
-   **专用端口**：避免使用 `9222`，以防止与开发工作流冲突。
-   **确定性的标签页控制**：通过 `targetId` 定位标签页，而不是“最后一个标签页”。

## 浏览器选择

在本地启动时，OpenClaw 选择第一个可用的：

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

您可以使用 `browser.executablePath` 覆盖。平台：

-   macOS：检查 `/Applications` 和 `~/Applications`。
-   Linux：查找 `google-chrome`、`brave`、`microsoft-edge`、`chromium` 等。
-   Windows：检查常见的安装位置。

## 控制 API（可选）

仅用于本地集成，网关暴露一个小型的环回 HTTP API：

-   状态/启动/停止：`GET /`、`POST /start`、`POST /stop`
-   标签页：`GET /tabs`、`POST /tabs/open`、`POST /tabs/focus`、`DELETE /tabs/:targetId`
-   快照/截图：`GET /snapshot`、`POST /screenshot`
-   操作：`POST /navigate`、`POST /act`
-   钩子：`POST /hooks/file-chooser`、`POST /hooks/dialog`
-   下载：`POST /download`、`POST /wait/download`
-   调试：`GET /console`、`POST /pdf`
-   调试：`GET /errors`、`GET /requests`、`POST /trace/start`、`POST /trace/stop`、`POST /highlight`
-   网络：`POST /response/body`
-   状态：`GET /cookies`、`POST /cookies/set`、`POST /cookies/clear`
-   状态：`GET /storage/:kind`、`POST /storage/:kind/set`、`POST /storage/:kind/clear`
-   设置：`POST /set/offline`、`POST /set/headers`、`POST /set/credentials`、`POST /set/geolocation`、`POST /set/media`、`POST /set/timezone`、`POST /set/locale`、`POST /set/device`

所有端点都接受 `?profile=`。如果配置了网关身份验证，浏览器 HTTP 路由也需要身份验证：

-   `Authorization: Bearer `
-   `x-openclaw-password: ` 或使用该密码的 HTTP 基本认证

### Playwright 要求

某些功能（导航/操作/AI 快照/角色快照、元素截图、PDF）需要 Playwright。如果未安装 Playwright，这些端点将返回清晰的 501 错误。ARIA 快照和基本截图对于 openclaw 托管的 Chrome 仍然有效。对于 Chrome 扩展中继驱动程序，ARIA 快照和截图需要 Playwright。如果您看到 `Playwright is not available in this gateway build`，请安装完整的 Playwright 包（不是 `playwright-core`）并重启网关，或者重新安装带有浏览器支持的 OpenClaw。

#### Docker 中安装 Playwright

如果您的网关在 Docker 中运行，请避免使用 `npx playwright`（npm 覆盖冲突）。请改用捆绑的 CLI：

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

要持久化浏览器下载，请设置 `PLAYWRIGHT_BROWSERS_PATH`（例如，`/home/node/.cache/ms-playwright`）并确保通过 `OPENCLAW_HOME_VOLUME` 或绑定挂载持久化 `/home/node`。参见 [Docker](../install/docker.md)。

## 工作原理（内部）

高层流程：

-   一个小的**控制服务器**接受 HTTP 请求。
-   它通过 **CDP** 连接到基于 Chromium 的浏览器（Chrome/Brave/Edge/Chromium）。
-   对于高级操作（点击/输入/快照/PDF），它在 CDP 之上使用 **Playwright**。
-   当缺少 Playwright 时，仅非 Playwright 操作可用。

这种设计使智能体保持在稳定、确定的接口上，同时允许您交换本地/远程浏览器和配置文件。

## CLI 快速参考

所有命令都接受 `--browser-profile ` 以定位特定配置文件。所有命令也接受 `--json` 以获取机器可读的输出（稳定的负载）。基础命令：

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

检查命令：

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

操作命令：

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

状态命令：

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

注意事项：

-   `upload` 和 `dialog` 是**预置**调用；在触发文件选择器/对话框的点击/按键操作之前运行它们。
-   下载和跟踪输出路径被限制在 OpenClaw 临时根目录下：
    -   跟踪：`/tmp/openclaw`（回退：`${os.tmpdir()}/openclaw`）
    -   下载：`/tmp/openclaw/downloads`（回退：`${os.tmpdir()}/openclaw/downloads`）
-   上传路径被限制在 OpenClaw 临时上传根目录下：
    -   上传：`/tmp/openclaw/uploads`（回退：`${os.tmpdir()}/openclaw/uploads`）
-   `upload` 也可以通过 `--input-ref` 或 `--element` 直接设置文件输入框。
-   `snapshot`：
    -   `--format ai`（安装 Playwright 时的默认值）：返回带有数字引用（`aria-ref=""`）的 AI 快照。
    -   `--format aria`：返回无障碍功能树（无引用；仅用于检查）。
    -   `--efficient`（或 `--mode efficient`）：紧凑的角色快照预设（交互式 + 紧凑 + 深度 + 较低的 maxChars）。
    -   配置默认值（仅限工具/CLI）：设置 `browser.snapshotDefaults.mode: "efficient"`，以便在调用者未传递模式时使用高效快照（参见[网关配置](../gateway/configuration.md#browser-openclaw-managed-browser)）。
    -   角色快照选项（`--interactive`、`--compact`、`--depth`、`--selector`）强制生成带有类似 `ref=e12` 引用的基于角色的快照。
    -   `--frame ""` 将角色快照范围限定在 iframe 内（与 `e12` 等角色引用配对使用）。
    -   `--interactive` 输出一个扁平的、易于选择的交互元素列表（最适合驱动操作）。
    -   `--labels` 添加一个仅包含视口的截图，并叠加引用标签（打印 `MEDIA:`）。
-   `click`/`type`/等操作需要来自 `snapshot` 的 `ref`（数字 `12` 或角色引用 `e12`）。CSS 选择器故意不支持用于操作。

## 快照与引用

OpenClaw 支持两种“快照”样式：

-   **AI 快照（数字引用）**：`openclaw browser snapshot`（默认；`--format ai`）
    -   输出：包含数字引用的文本快照。
    -   操作：`openclaw browser click 12`、`openclaw browser type 23 "hello"`。
    -   内部实现：引用通过 Playwright 的 `aria-ref` 解析。
-   **角色快照（类似 `e12` 的角色引用）**：`openclaw browser snapshot --interactive`（或 `--compact`、`--depth`、`--selector`、`--frame`）
    -   输出：基于角色的列表/树，带有 `[ref=e12]`（以及可选的 `[nth=1]`）。
    -   操作：`openclaw browser click e12`、`openclaw browser highlight e12`。
    -   内部实现：引用通过 `getByRole(...)` 解析（对于重复项加上 `nth()`）。
    -   添加 `--labels` 以包含带有叠加 `e12` 标签的视口截图。

引用行为：

-   引用**在导航之间不稳定**；如果操作失败，请重新运行 `snapshot` 并使用新的引用。
-   如果角色快照是使用 `--frame` 拍摄的，则角色引用将限定在该 iframe 内，直到下一次角色快照。

## 等待功能增强

您可以等待的不仅仅是时间/文本：

-   等待 URL（支持 Playwright 的通配符）：
    -   `openclaw browser wait --url "**/dash"`
-   等待加载状态：
    -   `openclaw browser wait --load networkidle`
-   等待 JS 谓词：
    -   `openclaw browser wait --fn "window.ready===true"`
-   等待选择器变为可见：
    -   `openclaw browser wait "#main"`

这些可以组合使用：

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## 调试工作流

当操作失败时（例如“不可见”、“严格模式违规”、“被覆盖”）：

1.  `openclaw browser snapshot --interactive`
2.  使用 `click ` / `type `（在交互模式下优先使用角色引用）
3.  如果仍然失败：`openclaw browser highlight ` 以查看 Playwright 正在定位什么
4.  如果页面行为异常：
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  进行深度调试：记录跟踪：
    -   `openclaw browser trace start`
    -   重现问题
    -   `openclaw browser trace stop`（打印 `TRACE:`）

## JSON 输出

`--json` 用于脚本编写和结构化工具。示例：

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

JSON 格式的角色快照包含 `refs` 以及一个小的 `stats` 块（行数/字符数/引用数/交互元素数），以便工具可以推断负载大小和密度。

## 状态和环境调节

这些对于“使网站表现得像 X”的工作流很有用：

-   Cookies：`cookies`、`cookies set`、`cookies clear`
-   存储：`storage local|session get|set|clear`
-   离线：`set offline on|off`
-   请求头：`set headers --headers-json '{"X-Debug":"1"}'`（旧版 `set headers --json '{"X-Debug":"1"}'` 仍受支持）
-   HTTP 基本认证：`set credentials user pass`（或 `--clear`）
-   地理位置：`set geo   --origin "https://example.com"`（或 `--clear`）
-   媒体：`set media dark|light|no-preference|none`
-   时区 / 区域设置：`set timezone ...`、`set locale ...`
-   设备 / 视口：
    -   `set device "iPhone 14"`（Playwright 设备预设）
    -   `set viewport 1280 720`

## 安全与隐私

-   openclaw 浏览器配置文件可能包含已登录的会话；请将其视为敏感信息。
-   `browser act kind=evaluate` / `openclaw browser evaluate` 和 `wait --fn` 在页面上下文中执行任意 JavaScript。提示注入可以引导此操作。如果您不需要它，请通过 `browser.evaluateEnabled=false` 禁用它。
-   关于登录和反机器人注意事项（X/Twitter 等），请参见[浏览器登录 + X/Twitter 发帖](./browser-login.md)。
-   保持网关/节点主机私有（仅限环回或 tailnet）。
-   远程 CDP 端点功能强大；请对其进行隧道传输和保护。

严格模式示例（默认阻止私有/内部目标）：

```json
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // 可选精确允许
    },
  },
}
```

## 故障排除

关于 Linux 特定问题（尤其是 snap Chromium），请参见[浏览器故障排除](./browser-linux-troubleshooting.md)。

## 智能体工具 + 控制工作原理

智能体获得**一个工具**用于浏览器自动化：

-   `browser` — 状态/启动/停止/标签页/打开/聚焦/关闭/快照/截图/导航/操作

映射方式：

-   `browser snapshot` 返回一个稳定的 UI 树（AI 或 ARIA）。
-   `browser act` 使用快照 `ref` ID 进行点击/输入/拖拽/选择。
-   `browser screenshot` 捕获像素（整个页面或元素）。
-   `browser` 接受：
    -   `profile` 以选择命名的浏览器配置文件（openclaw、chrome 或远程 CDP）。
    -   `target`（`sandbox` | `host` | `node`）以选择浏览器所在位置。
    -   在沙盒会话中，`target: "host"` 需要 `agents.defaults.sandbox.browser.allowHostControl=true`。
    -   如果省略 `target`：沙盒会话默认为 `sandbox`，非沙盒会话默认为 `host`。
    -   如果连接了具有浏览器能力的节点，除非您固定 `target="host"` 或 `target="node"`，否则工具可能会自动路由到该节点。

这使智能体保持确定性，并避免使用脆弱的 CSS 选择器。

[Web 工具](./web.md)[浏览器登录](./browser-login.md)