

  平台概览

  
# Oracle Cloud

## 目标

在 Oracle Cloud 的**永久免费** ARM 层上运行一个持久的 OpenClaw Gateway。Oracle 的免费层可能非常适合 OpenClaw（特别是如果你已经有一个 OCI 账户），但它有一些权衡：

-   ARM 架构（大多数东西都能工作，但有些二进制文件可能仅限 x86）
-   容量和注册可能比较棘手

## 成本对比 (2026)

| 提供商 | 方案 | 规格 | 月价格 | 备注 |
| --- | --- | --- | --- | --- |
| Oracle Cloud | 永久免费 ARM | 最多 4 OCPU, 24GB 内存 | $0 | ARM，容量有限 |
| Hetzner | CX22 | 2 vCPU, 4GB 内存 | ~ $4 | 最便宜的付费选项 |
| DigitalOcean | 基础型 | 1 vCPU, 1GB 内存 | $6 | 易于使用的 UI，文档完善 |
| Vultr | 云计算 | 1 vCPU, 1GB 内存 | $6 | 多个地点 |
| Linode | Nanode | 1 vCPU, 1GB 内存 | $5 | 现为 Akamai 的一部分 |

* * *

## 先决条件

-   Oracle Cloud 账户 ([注册](https://www.oracle.com/cloud/free/)) — 如果遇到问题，请参阅[社区注册指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)
-   Tailscale 账户 (在 [tailscale.com](https://tailscale.com) 免费注册)
-   ~30 分钟

## 1) 创建 OCI 实例

1.  登录 [Oracle Cloud 控制台](https://cloud.oracle.com/)
2.  导航到 **计算 → 实例 → 创建实例**
3.  配置：
    -   **名称：** `openclaw`
    -   **镜像：** Ubuntu 24.04 (aarch64)
    -   **形状：** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPU：** 2 (或最多 4)
    -   **内存：** 12 GB (或最多 24 GB)
    -   **启动卷：** 50 GB (最多 200 GB 免费)
    -   **SSH 密钥：** 添加你的公钥
4.  点击 **创建**
5.  记下公共 IP 地址

**提示：** 如果实例创建失败并显示“容量不足”，请尝试不同的可用性域或稍后重试。免费层容量有限。

## 2) 连接并更新

```bash
# 通过公共 IP 连接
ssh ubuntu@YOUR_PUBLIC_IP

# 更新系统
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**注意：** `build-essential` 是 ARM 编译某些依赖项所必需的。

## 3) 配置用户和主机名

```bash
# 设置主机名
sudo hostnamectl set-hostname openclaw

# 为 ubuntu 用户设置密码
sudo passwd ubuntu

# 启用 lingering (在注销后保持用户服务运行)
sudo loginctl enable-linger ubuntu
```

## 4) 安装 Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

这将启用 Tailscale SSH，因此你可以从 tailnet 上的任何设备通过 `ssh openclaw` 连接 — 无需公共 IP。验证：

```bash
tailscale status
```

**从现在开始，通过 Tailscale 连接：** `ssh ubuntu@openclaw` (或使用 Tailscale IP)。

## 5) 安装 OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

当提示“你希望如何孵化你的机器人？”时，选择 **“稍后再说”**。

> 注意：如果你遇到 ARM 原生构建问题，请先使用系统包（例如 `sudo apt install -y build-essential`），然后再尝试 Homebrew。

## 6) 配置网关（环回 + 令牌认证）并启用 Tailscale Serve

使用令牌认证作为默认方式。它是可预测的，并且避免了需要任何“不安全认证”的控制 UI 标志。

```bash
# 将网关保持在 VM 上私有
openclaw config set gateway.bind loopback

# 为网关 + 控制 UI 要求认证
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# 通过 Tailscale Serve 暴露 (HTTPS + tailnet 访问)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) 验证

```bash
# 检查版本
openclaw --version

# 检查守护进程状态
systemctl --user status openclaw-gateway

# 检查 Tailscale Serve
tailscale serve status

# 测试本地响应
curl http://localhost:18789
```

## 8) 锁定 VCN 安全

现在一切正常，锁定 VCN 以阻止除 Tailscale 之外的所有流量。OCI 的虚拟云网络在网络边缘充当防火墙 — 流量在到达你的实例之前就被阻止。

1.  在 OCI 控制台中转到 **网络 → 虚拟云网络**
2.  点击你的 VCN → **安全列表** → 默认安全列表
3.  **删除**所有入口规则，除了：
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  保留默认出口规则（允许所有出站）

这将在网络边缘阻止端口 22 上的 SSH、HTTP、HTTPS 和所有其他内容。从现在开始，你只能通过 Tailscale 连接。

* * *

## 访问控制 UI

从你的 Tailscale 网络上的任何设备：

```
https://openclaw.<tailnet-name>.ts.net/
```

将 `<tailnet-name>` 替换为你的 tailnet 名称（在 `tailscale status` 中可见）。无需 SSH 隧道。Tailscale 提供：

-   HTTPS 加密（自动证书）
-   通过 Tailscale 身份认证
-   从 tailnet 上的任何设备（笔记本电脑、手机等）访问

* * *

## 安全：VCN + Tailscale（推荐基线）

通过锁定 VCN（仅开放 UDP 41641）并将网关绑定到环回，你获得了强大的纵深防御：公共流量在网络边缘被阻止，管理访问通过你的 tailnet 进行。这种设置通常*无需*额外的基于主机的防火墙规则来阻止全网 SSH 暴力破解 — 但你仍然应该保持操作系统更新，运行 `openclaw security audit`，并验证你没有意外监听公共接口。

### 已受保护的内容

| 传统步骤 | 需要吗？ | 原因 |
| --- | --- | --- |
| UFW 防火墙 | 否 | VCN 在流量到达实例前就阻止了 |
| fail2ban | 否 | 如果端口 22 在 VCN 被阻止，则无暴力破解 |
| sshd 加固 | 否 | Tailscale SSH 不使用 sshd |
| 禁用 root 登录 | 否 | Tailscale 使用 Tailscale 身份，而非系统用户 |
| 仅 SSH 密钥认证 | 否 | Tailscale 通过你的 tailnet 认证 |
| IPv6 加固 | 通常不需要 | 取决于你的 VCN/子网设置；验证实际分配/暴露的内容 |

### 仍然推荐

-   **凭证权限：** `chmod 700 ~/.openclaw`
-   **安全审计：** `openclaw security audit`
-   **系统更新：** 定期运行 `sudo apt update && sudo apt upgrade`
-   **监控 Tailscale：** 在 [Tailscale 管理控制台](https://login.tailscale.com/admin) 中查看设备

### 验证安全态势

```bash
# 确认没有监听公共端口
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# 验证 Tailscale SSH 是否激活
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH 已激活"

# 可选：完全禁用 sshd
sudo systemctl disable --now ssh
```

* * *

## 备用方案：SSH 隧道

如果 Tailscale Serve 不工作，使用 SSH 隧道：

```bash
# 从你的本地机器（通过 Tailscale）
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

然后打开 `http://localhost:18789`。

* * *

## 故障排除

### 实例创建失败（“容量不足”）

免费层 ARM 实例很受欢迎。尝试：

-   不同的可用性域
-   在非高峰时段重试（清晨）
-   选择形状时使用“永久免费”过滤器

### Tailscale 无法连接

```bash
# 检查状态
sudo tailscale status

# 重新认证
sudo tailscale up --ssh --hostname=openclaw --reset
```

### 网关无法启动

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### 无法访问控制 UI

```bash
# 验证 Tailscale Serve 是否正在运行
tailscale serve status

# 检查网关是否在监听
curl http://localhost:18789

# 如果需要，重启
systemctl --user restart openclaw-gateway
```

### ARM 二进制文件问题

某些工具可能没有 ARM 构建。检查：

```bash
uname -m  # 应显示 aarch64
```

大多数 npm 包工作正常。对于二进制文件，请查找 `linux-arm64` 或 `aarch64` 版本。

* * *

## 持久性

所有状态保存在：

-   `~/.openclaw/` — 配置、凭证、会话数据
-   `~/.openclaw/workspace/` — 工作空间 (SOUL.md, 记忆, 工件)

定期备份：

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## 另请参阅

-   [网关远程访问](../gateway/remote.md) — 其他远程访问模式
-   [Tailscale 集成](../gateway/tailscale.md) — 完整的 Tailscale 文档
-   [网关配置](../gateway/configuration.md) — 所有配置选项
-   [DigitalOcean 指南](./digitalocean.md) — 如果你想要付费 + 更容易注册
-   [Hetzner 指南](../install/hetzner.md) — 基于 Docker 的替代方案

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)