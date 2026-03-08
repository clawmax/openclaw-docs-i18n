

  平台概览

  
# 平台

OpenClaw 核心使用 TypeScript 编写。**Node 是推荐的运行时**。不建议将 Bun 用于网关（WhatsApp/Telegram 存在错误）。macOS（菜单栏应用）和移动节点（iOS/Android）有伴侣应用。Windows 和 Linux 的伴侣应用正在规划中，但网关目前已完全支持。Windows 的原生伴侣应用也在规划中；建议通过 WSL2 使用网关。

## 选择您的操作系统

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS 与托管

-   VPS 中心: [VPS 托管](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + HTTPS 代理): [exe.dev](./install/exe-dev.md)

## 常用链接

-   安装指南: [入门指南](./start/getting-started.md)
-   网关运行手册: [网关](./gateway.md)
-   网关配置: [配置](./gateway/configuration.md)
-   服务状态: `openclaw gateway status`

## 网关服务安装 (CLI)

使用以下方法之一（均支持）：

-   向导（推荐）: `openclaw onboard --install-daemon`
-   直接安装: `openclaw gateway install`
-   配置流程: `openclaw configure` → 选择 **网关服务**
-   修复/迁移: `openclaw doctor`（提供安装或修复服务的选项）

服务目标取决于操作系统：

-   macOS: LaunchAgent (`ai.openclaw.gateway` 或 `ai.openclaw.`；旧版 `com.openclaw.*`)
-   Linux/WSL2: systemd 用户服务 (`openclaw-gateway[-].service`)

[macOS 应用](./platforms/macos.md)

---