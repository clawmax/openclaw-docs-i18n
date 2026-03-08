

  CLI 命令

  
# node

运行一个**无头节点主机**，它连接到网关 WebSocket 并在此机器上暴露 `system.run` / `system.which` 功能。

## 为什么使用节点主机？

当您希望代理在您网络中的**其他机器上运行命令**，而又不想在那里安装完整的 macOS 伴侣应用时，请使用节点主机。常见用例：

-   在远程 Linux/Windows 机器（构建服务器、实验室机器、NAS）上运行命令。
-   将执行**沙盒化**在网关上，但将已批准的运行委托给其他主机。
-   为自动化或 CI 节点提供一个轻量级的、无头的执行目标。

执行仍然受到节点主机上的**执行审批**和每个代理允许列表的保护，因此您可以保持命令访问范围明确且受控。

## 浏览器代理（零配置）

如果节点上的 `browser.enabled` 未被禁用，节点主机会自动通告一个浏览器代理。这使得代理可以在该节点上使用浏览器自动化，而无需额外配置。如有需要，可以在节点上禁用它：

```json
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## 运行（前台）

```bash
openclaw node run --host <gateway-host> --port 18789
```

选项：

-   `--host `: 网关 WebSocket 主机（默认：`127.0.0.1`）
-   `--port `: 网关 WebSocket 端口（默认：`18789`）
-   `--tls`: 为网关连接使用 TLS
-   `--tls-fingerprint `: 预期的 TLS 证书指纹 (sha256)
-   `--node-id `: 覆盖节点 ID（清除配对令牌）
-   `--display-name `: 覆盖节点显示名称

## 服务（后台）

将无头节点主机安装为用户服务。

```bash
openclaw node install --host <gateway-host> --port 18789
```

选项：

-   `--host `: 网关 WebSocket 主机（默认：`127.0.0.1`）
-   `--port `: 网关 WebSocket 端口（默认：`18789`）
-   `--tls`: 为网关连接使用 TLS
-   `--tls-fingerprint `: 预期的 TLS 证书指纹 (sha256)
-   `--node-id `: 覆盖节点 ID（清除配对令牌）
-   `--display-name `: 覆盖节点显示名称
-   `--runtime `: 服务运行时（`node` 或 `bun`）
-   `--force`: 如果已安装则重新安装/覆盖

管理服务：

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

使用 `openclaw node run` 运行前台节点主机（非服务）。服务命令接受 `--json` 参数以输出机器可读格式。

## 配对

首次连接会在网关上创建一个待处理的设备配对请求（`role: node`）。通过以下方式批准：

```bash
openclaw devices list
openclaw devices approve <requestId>
```

节点主机将其节点 ID、令牌、显示名称和网关连接信息存储在 `~/.openclaw/node.json` 中。

## 执行审批

`system.run` 受本地执行审批控制：

-   `~/.openclaw/exec-approvals.json`
-   [执行审批](../tools/exec-approvals.md)
-   `openclaw approvals --node <id|name|ip>`（从网关编辑）

[models](./models.md)[nodes](./nodes.md)

---