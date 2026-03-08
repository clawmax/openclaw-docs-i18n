

  平台概览

  
# DigitalOcean

## 目标

在 DigitalOcean 上运行一个持久的 OpenClaw Gateway，每月费用为 **6 美元**（或使用预留定价每月 4 美元）。如果你想要每月 0 美元的选项并且不介意 ARM 架构和特定于提供商的设置，请参阅 [Oracle Cloud 指南](./oracle.md)。

## 成本对比 (2026)

| 提供商 | 方案 | 规格 | 月价格 | 备注 |
| --- | --- | --- | --- | --- |
| Oracle Cloud | 始终免费 ARM | 最多 4 OCPU, 24GB 内存 | $0 | ARM，容量有限/注册流程特殊 |
| Hetzner | CX22 | 2 vCPU, 4GB 内存 | €3.79 (~$4) | 最便宜的付费选项 |
| DigitalOcean | 基础型 | 1 vCPU, 1GB 内存 | $6 | 界面简单，文档完善 |
| Vultr | 云计算 | 1 vCPU, 1GB 内存 | $6 | 多个地区可选 |
| Linode | Nanode | 1 vCPU, 1GB 内存 | $5 | 现为 Akamai 的一部分 |

**选择提供商：**

-   DigitalOcean：最简单的用户体验 + 可预测的设置（本指南）
-   Hetzner：良好的性价比（参见 [Hetzner 指南](../install/hetzner.md)）
-   Oracle Cloud：可以做到每月 0 美元，但更挑剔且仅限 ARM（参见 [Oracle 指南](./oracle.md)）

* * *

## 先决条件

-   DigitalOcean 账户（[注册可获得 200 美元免费额度](https://m.do.co/c/signup)）
-   SSH 密钥对（或愿意使用密码认证）
-   ~20 分钟

## 1) 创建 Droplet

> **⚠️** 使用干净的基础镜像（Ubuntu 24.04 LTS）。避免使用第三方 Marketplace 一键镜像，除非你已审查过它们的启动脚本和防火墙默认设置。

1.  登录 [DigitalOcean](https://cloud.digitalocean.com/)
2.  点击 **创建 → Droplets**
3.  选择：
    -   **地区：** 离你（或你的用户）最近的
    -   **镜像：** Ubuntu 24.04 LTS
    -   **规格：** 基础型 → 常规型 → **$6/月** (1 vCPU, 1GB 内存, 25GB SSD)
    -   **认证：** SSH 密钥（推荐）或密码
4.  点击 **创建 Droplet**
5.  记下 IP 地址

## 2) 通过 SSH 连接

```bash
ssh root@你的_DROPLET_IP
```

## 3) 安装 OpenClaw

```bash
# 更新系统
apt update && apt upgrade -y

# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 验证
openclaw --version
```

## 4) 运行初始化设置

```bash
openclaw onboard --install-daemon
```

向导将引导你完成：

-   模型认证（API 密钥或 OAuth）
-   频道设置（Telegram, WhatsApp, Discord 等）
-   网关令牌（自动生成）
-   守护进程安装（systemd）

## 5) 验证网关

```bash
# 检查状态
openclaw status

# 检查服务
systemctl --user status openclaw-gateway.service

# 查看日志
journalctl --user -u openclaw-gateway.service -f
```

## 6) 访问控制面板

网关默认绑定到本地回环地址。要访问控制界面：
**选项 A: SSH 隧道（推荐）**

```bash
# 在你的本地机器上执行
ssh -L 18789:localhost:18789 root@你的_DROPLET_IP

# 然后打开：http://localhost:18789
```

**选项 B: Tailscale Serve (HTTPS, 仅限回环)**

```bash
# 在 Droplet 上执行
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# 配置网关使用 Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

打开：`https:///` 注意：

-   Serve 模式保持网关仅限回环访问，并通过 Tailscale 身份验证头来认证控制界面/WebSocket 流量（无令牌认证假设网关主机是可信的；HTTP API 仍需要令牌/密码）。
-   如果需要令牌/密码认证，请设置 `gateway.auth.allowTailscale: false` 或使用 `gateway.auth.mode: "password"`。

**选项 C: Tailnet 绑定（不使用 Serve）**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

打开：`http://<tailscale-ip>:18789`（需要令牌）。

## 7) 连接你的频道

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# 扫描二维码
```

其他提供商请参见 [频道](../channels.md)。

* * *

## 针对 1GB 内存的优化

6 美元的 Droplet 只有 1GB 内存。为了保持运行顺畅：

### 添加交换空间（推荐）

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### 使用更轻量的模型

如果你遇到内存不足（OOM）问题，可以考虑：

-   使用基于 API 的模型（Claude, GPT）而不是本地模型
-   将 `agents.defaults.model.primary` 设置为更小的模型

### 监控内存

```
free -h
htop
```

* * *

## 持久性

所有状态保存在：

-   `~/.openclaw/` — 配置、凭证、会话数据
-   `~/.openclaw/workspace/` — 工作空间（SOUL.md, 记忆等）

这些数据在重启后仍然存在。定期备份它们：

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Oracle Cloud 免费替代方案

Oracle Cloud 提供 **始终免费** 的 ARM 实例，其性能远超此处任何付费选项 — 每月 0 美元。

| 你能获得什么 | 规格 |
| --- | --- |
| **4 OCPU** | ARM Ampere A1 |
| **24GB 内存** | 绰绰有余 |
| **200GB 存储** | 块存储卷 |
| **永久免费** | 无信用卡扣费 |

**注意事项：**

-   注册过程可能比较挑剔（如果失败请重试）
-   ARM 架构 — 大多数东西都能运行，但有些二进制文件需要 ARM 版本

完整的设置指南，请参阅 [Oracle Cloud](./oracle.md)。关于注册技巧和解决注册流程问题的信息，请参阅此 [社区指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)。

* * *

## 故障排除

### 网关无法启动

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### 端口已被占用

```bash
lsof -i :18789
kill <PID>
```

### 内存不足

```bash
# 检查内存
free -h

# 添加更多交换空间
# 或者升级到 $12/月的 Droplet (2GB 内存)
```

* * *

## 另请参阅

-   [Hetzner 指南](../install/hetzner.md) — 更便宜，性能更强
-   [Docker 安装](../install/docker.md) — 容器化设置
-   [Tailscale](../gateway/tailscale.md) — 安全的远程访问
-   [配置](../gateway/configuration.md) — 完整的配置参考

[iOS 应用](./ios.md)[Oracle Cloud](./oracle.md)

---