

  Web 界面

  
# 仪表板

网关仪表板是默认在 `/` 路径提供的浏览器控制界面（可通过 `gateway.controlUi.basePath` 覆盖）。快速打开（本地网关）：

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (或 [http://localhost:18789/](http://localhost:18789/))

关键参考：

-   [控制界面](./control-ui.md) 了解使用方法和界面功能。
-   [Tailscale](../gateway/tailscale.md) 了解 Serve/Funnel 自动化。
-   [Web 界面](../web.md) 了解绑定模式和安全性说明。

身份验证在 WebSocket 握手阶段通过 `connect.params.auth`（令牌或密码）强制执行。请参阅 [网关配置](../gateway/configuration.md) 中的 `gateway.auth`。安全提示：控制界面是一个**管理界面**（聊天、配置、执行审批）。请勿将其公开暴露。界面会将仪表板 URL 令牌保留在当前标签页的内存中，并在加载后从 URL 中移除。建议使用本地主机、Tailscale Serve 或 SSH 隧道。

## 快速路径（推荐）

-   完成初始设置后，CLI 会自动打开仪表板并打印一个干净的（不含令牌的）链接。
-   随时重新打开：`openclaw dashboard`（复制链接，如果可能则打开浏览器，如果是无头环境则显示 SSH 提示）。
-   如果界面提示进行身份验证，请将 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）中的令牌粘贴到控制界面设置中。

## 令牌基础（本地与远程）

-   **本地主机**：打开 `http://127.0.0.1:18789/`。
-   **令牌来源**：`gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）；`openclaw dashboard` 可以通过 URL 片段传递令牌以进行一次性引导，但控制界面不会将网关令牌持久化存储在 localStorage 中。
-   如果 `gateway.auth.token` 由 SecretRef 管理，`openclaw dashboard` 会设计为打印/复制/打开一个不含令牌的 URL。这可以避免在 shell 日志、剪贴板历史记录或浏览器启动参数中暴露外部管理的令牌。
-   如果 `gateway.auth.token` 配置为 SecretRef 且在当前 shell 中未解析，`openclaw dashboard` 仍会打印一个不含令牌的 URL 以及可操作的身份验证设置指导。
-   **非本地主机**：使用 Tailscale Serve（如果 `gateway.auth.allowTailscale: true`，则控制界面/WebSocket 无需令牌，前提是网关主机可信；HTTP API 仍需要令牌/密码）、带有令牌的 tailnet 绑定或 SSH 隧道。请参阅 [Web 界面](../web.md)。

## 如果看到“未授权”/ 1008 错误

-   确保网关可达（本地：`openclaw status`；远程：SSH 隧道 `ssh -N -L 18789:127.0.0.1:18789 user@host`，然后打开 `http://127.0.0.1:18789/`）。
-   从网关主机检索或提供令牌：
    -   明文配置：`openclaw config get gateway.auth.token`
    -   SecretRef 管理的配置：解析外部密钥提供程序或在此 shell 中导出 `OPENCLAW_GATEWAY_TOKEN`，然后重新运行 `openclaw dashboard`
    -   未配置令牌：`openclaw doctor --generate-gateway-token`
-   在仪表板设置中，将令牌粘贴到身份验证字段，然后连接。

[控制界面](./control-ui.md)[WebChat](./webchat.md)

---