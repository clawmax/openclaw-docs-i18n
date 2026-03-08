

  浏览器

  
# Chrome 扩展

OpenClaw Chrome 扩展允许代理控制您**现有的 Chrome 标签页**（您的常规 Chrome 窗口），而不是启动一个独立的 openclaw 管理的 Chrome 配置文件。附加/分离操作通过**一个 Chrome 工具栏按钮**进行。

## 它是什么（概念）

包含三个部分：

-   **浏览器控制服务**（网关或节点）：代理/工具调用的 API（通过网关）
-   **本地中继服务器**（环回 CDP）：在控制服务器和扩展之间建立桥梁（默认为 `http://127.0.0.1:18792`）
-   **Chrome MV3 扩展**：使用 `chrome.debugger` 附加到活动标签页，并将 CDP 消息传输到中继

然后，OpenClaw 通过正常的 `browser` 工具界面（选择正确的配置文件）来控制附加的标签页。

## 安装 / 加载（解压版）

1.  将扩展安装到稳定的本地路径：

```bash
openclaw browser extension install
```

2.  打印已安装扩展的目录路径：

```bash
openclaw browser extension path
```

3.  Chrome → `chrome://extensions`
    -   启用“开发者模式”
    -   “加载已解压的扩展程序” → 选择上面打印的目录

4.  固定该扩展。

## 更新（无需构建步骤）

扩展作为静态文件随 OpenClaw 发行版（npm 包）一起提供。没有单独的“构建”步骤。升级 OpenClaw 后：

-   重新运行 `openclaw browser extension install` 以刷新 OpenClaw 状态目录下的已安装文件。
-   Chrome → `chrome://extensions` → 在扩展上点击“重新加载”。

## 使用它（设置一次网关令牌）

OpenClaw 附带一个名为 `chrome` 的内置浏览器配置文件，该文件指向默认端口上的扩展中继。在首次附加之前，打开扩展的选项页面并设置：

-   `端口`（默认 `18792`）
-   `网关令牌`（必须与 `gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN` 匹配）

使用它：

-   CLI：`openclaw browser --browser-profile chrome tabs`
-   代理工具：`browser` 并指定 `profile="chrome"`

如果您想要不同的名称或不同的中继端口，请创建自己的配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### 自定义网关端口

如果您使用自定义网关端口，扩展中继端口会自动派生：**扩展中继端口 = 网关端口 + 3** 示例：如果 `gateway.port: 19001`，那么：

-   扩展中继端口：`19004`（网关端口 + 3）

在扩展的选项页面中配置扩展以使用派生的中继端口。

## 附加 / 分离（工具栏按钮）

-   打开您希望 OpenClaw 控制的标签页。
-   点击扩展图标。
    -   附加时，徽章显示 `ON`。
-   再次点击以分离。

## 它控制哪个标签页？

-   它**不会**自动控制“您正在查看的任何标签页”。
-   它**仅控制您通过点击工具栏按钮明确附加的**标签页。
-   要切换：打开另一个标签页并在那里点击扩展图标。

## 徽章 + 常见错误

-   `ON`：已附加；OpenClaw 可以驱动该标签页。
-   `…`：正在连接到本地中继。
-   `!`：中继无法访问/未认证（最常见：中继服务器未运行，或网关令牌缺失/错误）。

如果您看到 `!`：

-   确保网关在本地运行（默认设置），或者如果网关运行在其他机器上，请在此机器上运行一个节点主机。
-   打开扩展的选项页面；它会验证中继的可达性 + 网关令牌认证。

## 远程网关（使用节点主机）

### 本地网关（与 Chrome 在同一台机器上）—— 通常无需额外步骤

如果网关与 Chrome 运行在同一台机器上，它会在环回地址上启动浏览器控制服务并自动启动中继服务器。扩展与本地中继通信；CLI/工具调用则发送到网关。

### 远程网关（网关运行在其他地方）—— 运行一个节点主机

如果您的网关运行在另一台机器上，请在运行 Chrome 的机器上启动一个节点主机。网关会将浏览器操作代理到该节点；扩展 + 中继则保留在浏览器机器本地。如果连接了多个节点，请使用 `gateway.nodes.browser.node` 固定一个，或设置 `gateway.nodes.browser.mode`。

## 沙盒化（工具容器）

如果您的代理会话处于沙盒中（`agents.defaults.sandbox.mode != "off"`），`browser` 工具可能会受到限制：

-   默认情况下，沙盒会话通常以**沙盒浏览器**（`target="sandbox"`）为目标，而不是您的主机 Chrome。
-   Chrome 扩展中继接管需要控制**主机**浏览器控制服务器。

选项：

-   最简单的方法：在**非沙盒**会话/代理中使用扩展。
-   或者允许沙盒会话控制主机浏览器：

```json
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

然后确保工具策略没有拒绝该工具，并且（如果需要）使用 `target="host"` 调用 `browser`。调试：`openclaw sandbox explain`

## 远程访问提示

-   将网关和节点主机保持在同一个 tailnet 中；避免将中继端口暴露给局域网或公共互联网。
-   有意配对节点；如果您不希望远程控制，请禁用浏览器代理路由（`gateway.nodes.browser.mode="off"`）。

## “扩展路径”如何工作

`openclaw browser extension path` 打印包含扩展文件的**已安装**磁盘目录。CLI 特意**不**打印 `node_modules` 路径。始终先运行 `openclaw browser extension install`，将扩展复制到 OpenClaw 状态目录下的稳定位置。如果您移动或删除该安装目录，Chrome 会将扩展标记为损坏，直到您从有效路径重新加载它。

## 安全影响（请阅读）

这功能强大且具有风险。请将其视为赋予模型“操作您浏览器的权限”。

-   该扩展使用 Chrome 的调试器 API（`chrome.debugger`）。当附加时，模型可以：
    -   在该标签页中点击/输入/导航
    -   读取页面内容
    -   访问该标签页已登录会话可以访问的任何内容
-   **这不是隔离的**，不像专用的 openclaw 管理的配置文件。
    -   如果您附加到日常使用的配置文件/标签页，您就是在授予访问该账户状态的权限。

建议：

-   对于扩展中继使用，优先使用专用的 Chrome 配置文件（与您的个人浏览分开）。
-   将网关和任何节点主机保持在仅限 tailnet；依赖网关认证 + 节点配对。
-   避免通过局域网（`0.0.0.0`）暴露中继端口，并避免使用 Funnel（公开）。
-   中继会阻止非扩展来源，并要求 `/cdp` 和 `/extension` 都进行网关令牌认证。

相关内容：

-   浏览器工具概述：[浏览器](./browser.md)
-   安全审计：[安全](../gateway/security.md)
-   Tailscale 设置：[Tailscale](../gateway/tailscale.md)

[浏览器登录](./browser-login.md)[浏览器故障排除](./browser-linux-troubleshooting.md)