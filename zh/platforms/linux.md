

  平台概览

  
# Linux 应用

网关在 Linux 上得到完全支持。**Node 是推荐的运行时**。不建议将 Bun 用于网关（WhatsApp/Telegram 存在错误）。原生的 Linux 伴侣应用已在规划中。如果您想帮助构建一个，欢迎贡献。

## 新手快速路径 (VPS)

1.  安装 Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  从您的笔记本电脑执行：`ssh -N -L 18789:127.0.0.1:18789 @`
5.  打开 `http://127.0.0.1:18789/` 并粘贴您的令牌

分步 VPS 指南：[exe.dev](../install/exe-dev.md)

## 安装

-   [入门指南](../start/getting-started.md)
-   [安装与更新](../install/updating.md)
-   可选流程：[Bun (实验性)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## 网关

-   [网关运行手册](../gateway.md)
-   [配置](../gateway/configuration.md)

## 网关服务安装 (CLI)

使用以下命令之一：

```bash
openclaw onboard --install-daemon
```

或者：

```bash
openclaw gateway install
```

或者：

```bash
openclaw configure
```

当提示时选择 **网关服务**。修复/迁移：

```bash
openclaw doctor
```

## 系统控制 (systemd 用户单元)

OpenClaw 默认安装一个 systemd **用户**服务。对于共享或始终在线的服务器，请使用 **系统**服务。完整的单元示例和指导请参阅[网关运行手册](../gateway.md)。最小化设置：创建 `~/.config/systemd/user/openclaw-gateway[-].service`：

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

启用它：

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[macOS 应用](./macos.md)[Windows (WSL2)](./windows.md)