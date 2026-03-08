

  其他安装方式

  
# Ansible

部署 OpenClaw 到生产服务器的推荐方式是通过 **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** —— 一个采用安全优先架构的自动化安装程序。

## 快速开始

一键安装命令：

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 完整指南：[github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** openclaw-ansible 仓库是 Ansible 部署的权威来源。本页仅提供快速概览。

## 你将获得

-   🔒 **防火墙优先安全**：UFW + Docker 隔离（仅 SSH + Tailscale 可访问）
-   🔐 **Tailscale VPN**：安全的远程访问，无需公开暴露服务
-   🐳 **Docker**：隔离的沙箱容器，仅绑定到 localhost
-   🛡️ **深度防御**：4 层安全架构
-   🚀 **一键设置**：几分钟内完成完整部署
-   🔧 **Systemd 集成**：开机自启并具备安全加固

## 要求

-   **操作系统**：Debian 11+ 或 Ubuntu 20.04+
-   **访问权限**：Root 或 sudo 权限
-   **网络**：用于软件包安装的互联网连接
-   **Ansible**：2.14+（快速启动脚本会自动安装）

## 安装内容

Ansible 剧本将安装并配置：

1.  **Tailscale**（用于安全远程访问的网状 VPN）
2.  **UFW 防火墙**（仅开放 SSH + Tailscale 端口）
3.  **Docker CE + Compose V2**（用于代理沙箱）
4.  **Node.js 22.x + pnpm**（运行时依赖）
5.  **OpenClaw**（基于主机运行，非容器化）
6.  **Systemd 服务**（自动启动并具备安全加固）

注意：网关**直接在主机上运行**（不在 Docker 中），但代理沙箱使用 Docker 进行隔离。详情请参阅[沙箱化](../gateway/sandboxing.md)。

## 安装后设置

安装完成后，切换到 openclaw 用户：

```bash
sudo -i -u openclaw
```

安装后脚本将引导你完成：

1.  **入门向导**：配置 OpenClaw 设置
2.  **提供商登录**：连接 WhatsApp/Telegram/Discord/Signal
3.  **网关测试**：验证安装
4.  **Tailscale 设置**：连接到你的 VPN 网状网络

### 快速命令

```bash
# 检查服务状态
sudo systemctl status openclaw

# 查看实时日志
sudo journalctl -u openclaw -f

# 重启网关
sudo systemctl restart openclaw

# 提供商登录（以 openclaw 用户运行）
sudo -i -u openclaw
openclaw channels login
```

## 安全架构

### 4 层防御

1.  **防火墙 (UFW)**：仅 SSH (22) + Tailscale (41641/udp) 端口公开暴露
2.  **VPN (Tailscale)**：网关仅可通过 VPN 网状网络访问
3.  **Docker 隔离**：DOCKER-USER iptables 链阻止外部端口暴露
4.  **Systemd 加固**：NoNewPrivileges, PrivateTmp, 非特权用户

### 验证

测试外部攻击面：

```bash
nmap -p- YOUR_SERVER_IP
```

应显示**仅端口 22**（SSH）开放。所有其他服务（网关、Docker）均被锁定。

### Docker 可用性

安装 Docker 是为了**代理沙箱**（隔离的工具执行），而不是为了运行网关本身。网关仅绑定到 localhost，并且只能通过 Tailscale VPN 访问。有关沙箱配置，请参阅[多代理沙箱与工具](../tools/multi-agent-sandbox-tools.md)。

## 手动安装

如果你更喜欢手动控制而非自动化：

```bash
# 1. 安装先决条件
sudo apt update && sudo apt install -y ansible git

# 2. 克隆仓库
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. 安装 Ansible 集合
ansible-galaxy collection install -r requirements.yml

# 4. 运行剧本
./run-playbook.sh

# 或者直接运行（然后手动执行 /tmp/openclaw-setup.sh）
# ansible-playbook playbook.yml --ask-become-pass
```

## 更新 OpenClaw

Ansible 安装程序为 OpenClaw 设置了手动更新流程。请参阅[更新](./updating.md)了解标准更新流程。要重新运行 Ansible 剧本（例如，进行配置更改）：

```bash
cd openclaw-ansible
./run-playbook.sh
```

注意：这是幂等的，可以安全地多次运行。

## 故障排除

### 防火墙阻止了我的连接

如果你被锁定在外：

-   确保首先可以通过 Tailscale VPN 访问
-   SSH 访问（端口 22）始终被允许
-   网关**仅**可通过 Tailscale 访问，这是设计使然

### 服务无法启动

```bash
# 检查日志
sudo journalctl -u openclaw -n 100

# 验证权限
sudo ls -la /opt/openclaw

# 测试手动启动
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker 沙箱问题

```bash
# 验证 Docker 是否正在运行
sudo systemctl status docker

# 检查沙箱镜像
sudo docker images | grep openclaw-sandbox

# 如果缺少则构建沙箱镜像
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### 提供商登录失败

确保以 `openclaw` 用户身份运行：

```bash
sudo -i -u openclaw
openclaw channels login
```

## 高级配置

有关详细的安全架构和故障排除：

-   [安全架构](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [技术细节](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [故障排除指南](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## 相关链接

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 完整部署指南
-   [Docker](./docker.md) — 容器化网关设置
-   [沙箱化](../gateway/sandboxing.md) — 代理沙箱配置
-   [多代理沙箱与工具](../tools/multi-agent-sandbox-tools.md) — 每个代理的隔离

[Nix](./nix.md)[Bun (实验性)](./bun.md)