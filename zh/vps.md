

  托管与部署

  
# VPS 托管

本中心链接到支持的 VPS/托管指南，并从高层次解释云部署的工作原理。

## 选择提供商

-   **Railway** (一键 + 浏览器设置): [Railway](./install/railway.md)
-   **Northflank** (一键 + 浏览器设置): [Northflank](./install/northflank.md)
-   **Oracle Cloud (始终免费)**: [Oracle](./platforms/oracle.md) — $0/月 (始终免费，ARM；容量/注册可能有些棘手)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + HTTPS 代理): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/免费套餐)**: 同样运行良好。视频指南: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## 云设置如何工作

-   **网关运行在 VPS 上**，并拥有状态和工作区。
-   您通过 **控制 UI** 或 **Tailscale/SSH** 从您的笔记本电脑/手机连接。
-   将 VPS 视为单一事实来源，并**备份**状态和工作区。
-   安全默认设置：将网关保持在环回地址上，并通过 SSH 隧道或 Tailscale Serve 访问。如果您绑定到 `lan`/`tailnet`，则需要 `gateway.auth.token` 或 `gateway.auth.password`。

远程访问: [网关远程](./gateway/remote.md)  
平台中心: [平台](./platforms.md)

## 在 VPS 上共享公司代理

当用户处于一个信任边界内（例如一个公司团队）且代理仅用于业务时，这是一种有效的设置。

-   将其保持在专用运行时（VPS/VM/容器 + 专用操作系统用户/账户）上。
-   不要将该运行时登录到个人 Apple/Google 账户或个人浏览器/密码管理器配置文件。
-   如果用户彼此对立，请按网关/主机/操作系统用户进行拆分。

安全模型详情: [安全](./gateway/security.md)

## 在 VPS 上使用节点

您可以将网关保留在云端，并在本地设备（Mac/iOS/Android/无头设备）上配对**节点**。节点提供本地屏幕/摄像头/画布和 `system.run` 功能，而网关则保持在云端。文档: [节点](./nodes.md), [节点 CLI](./cli/nodes.md)

## 针对小型 VM 和 ARM 主机的启动调优

如果 CLI 命令在低功耗 VM（或 ARM 主机）上感觉缓慢，请启用 Node 的模块编译缓存：

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` 提高了重复命令的启动时间。
-   `OPENCLAW_NO_RESPAWN=1` 避免了自重启路径带来的额外启动开销。
-   首次命令运行会预热缓存；后续运行会更快。
-   有关 Raspberry Pi 的具体信息，请参阅 [Raspberry Pi](./platforms/raspberry-pi.md)。

### systemd 调优清单（可选）

对于使用 `systemd` 的 VM 主机，请考虑：

-   为服务添加环境变量以实现稳定的启动路径：
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   保持重启行为明确：
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   对于状态/缓存路径，优先使用 SSD 支持的磁盘，以减少随机 I/O 冷启动的惩罚。

示例：

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

`Restart=` 策略如何帮助自动恢复: [systemd 可以自动恢复服务](https://www.redhat.com/en/blog/systemd-automate-recovery)。

[卸载](./install/uninstall.md)[Fly.io](./install/fly.md)