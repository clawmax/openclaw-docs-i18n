

  平台概览

  
# 树莓派

## 目标

在树莓派上运行一个持久化、始终在线的 OpenClaw 网关，一次性成本约为 **35-80 美元**（无月费）。非常适合：

-   24/7 个人 AI 助手
-   家庭自动化中心
-   低功耗、始终可用的 Telegram/WhatsApp 机器人

## 硬件要求

| 树莓派型号 | 内存 | 是否可行？ | 备注 |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ 最佳 | 最快，推荐 |
| **Pi 4** | 4GB | ✅ 良好 | 大多数用户的最佳选择 |
| **Pi 4** | 2GB | ✅ 尚可 | 可工作，需添加交换空间 |
| **Pi 4** | 1GB | ⚠️ 紧张 | 使用交换空间可能可行，需最小化配置 |
| **Pi 3B+** | 1GB | ⚠️ 缓慢 | 可工作但反应迟钝 |
| **Pi Zero 2 W** | 512MB | ❌ | 不推荐 |

**最低配置：** 1GB 内存，1 核心，500MB 磁盘空间
**推荐配置：** 2GB+ 内存，64 位操作系统，16GB+ SD 卡（或 USB SSD）

## 所需物品

-   树莓派 4 或 5（推荐 2GB+ 内存）
-   MicroSD 卡（16GB+）或 USB SSD（性能更佳）
-   电源适配器（推荐官方树莓派电源）
-   网络连接（以太网或 WiFi）
-   ~30 分钟时间

## 1) 刷写操作系统

使用 **Raspberry Pi OS Lite (64 位)** — 无头服务器无需桌面环境。

1.  下载 [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2.  选择操作系统：**Raspberry Pi OS Lite (64 位)**
3.  点击齿轮图标 (⚙️) 进行预配置：
    -   设置主机名：`gateway-host`
    -   启用 SSH
    -   设置用户名/密码
    -   配置 WiFi（如果不使用以太网）
4.  刷写到您的 SD 卡 / USB 驱动器
5.  插入并启动树莓派

## 2) 通过 SSH 连接

```bash
ssh user@gateway-host
# 或使用 IP 地址
ssh user@192.168.x.x
```

## 3) 系统设置

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基础软件包
sudo apt install -y git curl build-essential

# 设置时区（对 cron/提醒很重要）
sudo timedatectl set-timezone America/Chicago  # 更改为您的时区
```

## 4) 安装 Node.js 22 (ARM64)

```bash
# 通过 NodeSource 安装 Node.js
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 验证
node --version  # 应显示 v22.x.x
npm --version
```

## 5) 添加交换空间（对 2GB 或更少内存很重要）

交换空间可防止内存不足崩溃：

```bash
# 创建 2GB 交换文件
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 设为永久
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 针对低内存优化（降低交换倾向）
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) 安装 OpenClaw

### 选项 A：标准安装（推荐）

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### 选项 B：可修改安装（用于调试）

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

可修改安装让您可以直接访问日志和代码 — 对调试 ARM 特定问题很有用。

## 7) 运行初始化向导

```bash
openclaw onboard --install-daemon
```

按照向导操作：

1.  **网关模式：** 本地
2.  **认证：** 推荐使用 API 密钥（无头 Pi 上 OAuth 可能不稳定）
3.  **通道：** Telegram 最容易上手
4.  **守护进程：** 是 (systemd)

## 8) 验证安装

```bash
# 检查状态
openclaw status

# 检查服务
sudo systemctl status openclaw

# 查看日志
journalctl -u openclaw -f
```

## 9) 访问仪表板

由于 Pi 是无头的，请使用 SSH 隧道：

```bash
# 从您的笔记本电脑/台式机执行
ssh -L 18789:localhost:18789 user@gateway-host

# 然后在浏览器中打开
open http://localhost:18789
```

或者使用 Tailscale 实现始终在线访问：

```bash
# 在 Pi 上执行
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# 更新配置
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## 性能优化

### 使用 USB SSD（巨大提升）

SD 卡速度慢且易磨损。USB SSD 能显著提升性能：

```bash
# 检查是否从 USB 启动
lsblk
```

设置方法请参阅 [Pi USB 启动指南](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot)。

### 加速 CLI 启动（模块编译缓存）

在低功耗 Pi 主机上，启用 Node 的模块编译缓存，以便重复的 CLI 运行更快：

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

说明：

-   `NODE_COMPILE_CACHE` 可加速后续运行（`status`、`health`、`--help`）。
-   `/var/tmp` 比 `/tmp` 更能经受重启。
-   `OPENCLAW_NO_RESPAWN=1` 避免 CLI 自我重启带来的额外启动开销。
-   首次运行会预热缓存；后续运行受益最大。

### systemd 启动调优（可选）

如果此 Pi 主要运行 OpenClaw，可以添加一个服务覆盖文件以减少重启抖动并保持启动环境稳定：

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

然后应用：

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

如果可能，将 OpenClaw 状态/缓存保存在 SSD 存储上，以避免冷启动期间 SD 卡随机 I/O 的瓶颈。`Restart=` 策略如何帮助自动恢复：[systemd 可以自动化服务恢复](https://www.redhat.com/en/blog/systemd-automate-recovery)。

### 减少内存使用

```bash
# 禁用 GPU 内存分配（无头模式）
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# 禁用不需要的蓝牙
sudo systemctl disable bluetooth
```

### 监控资源

```bash
# 检查内存
free -h

# 检查 CPU 温度
vcgencmd measure_temp

# 实时监控
htop
```

* * *

## ARM 特定说明

### 二进制兼容性

大多数 OpenClaw 功能在 ARM64 上都能工作，但某些外部二进制文件可能需要 ARM 构建：

| 工具 | ARM64 状态 | 备注 |
| --- | --- | --- |
| Node.js | ✅ | 运行良好 |
| WhatsApp (Baileys) | ✅ | 纯 JS，无问题 |
| Telegram | ✅ | 纯 JS，无问题 |
| gog (Gmail CLI) | ⚠️ | 检查是否有 ARM 版本 |
| Chromium (浏览器) | ✅ | `sudo apt install chromium-browser` |

如果某个技能失败，请检查其二进制文件是否有 ARM 构建。许多 Go/Rust 工具有；有些没有。

### 32 位 vs 64 位

**始终使用 64 位操作系统。** Node.js 和许多现代工具需要它。使用以下命令检查：

```bash
uname -m
# 应显示：aarch64 (64 位) 而不是 armv7l (32 位)
```

* * *

## 推荐的模型设置

由于 Pi 仅作为网关（模型在云端运行），请使用基于 API 的模型：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**不要尝试在 Pi 上运行本地 LLM** — 即使小模型也太慢。让 Claude/GPT 处理繁重任务。

* * *

## 开机自启动

初始化向导会设置此项，但可以验证：

```bash
# 检查服务是否已启用
sudo systemctl is-enabled openclaw

# 如果未启用则启用
sudo systemctl enable openclaw

# 开机启动
sudo systemctl start openclaw
```

* * *

## 故障排除

### 内存不足 (OOM)

```bash
# 检查内存
free -h

# 添加更多交换空间（见步骤 5）
# 或减少 Pi 上运行的服务
```

### 性能缓慢

-   使用 USB SSD 代替 SD 卡
-   禁用未使用的服务：`sudo systemctl disable cups bluetooth avahi-daemon`
-   检查 CPU 降频：`vcgencmd get_throttled`（应返回 `0x0`）

### 服务无法启动

```bash
# 检查日志
journalctl -u openclaw --no-pager -n 100

# 常见修复：重新构建
cd ~/openclaw  # 如果使用可修改安装
npm run build
sudo systemctl restart openclaw
```

### ARM 二进制文件问题

如果技能失败并显示“exec format error”：

1.  检查二进制文件是否有 ARM64 构建
2.  尝试从源代码构建
3.  或者使用支持 ARM 的 Docker 容器

### WiFi 断连

对于使用 WiFi 的无头 Pi：

```bash
# 禁用 WiFi 电源管理
sudo iwconfig wlan0 power off

# 设为永久
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## 成本对比

| 设置 | 一次性成本 | 月度成本 | 备注 |
| --- | --- | --- | --- |
| **Pi 4 (2GB)** | ~$45 | $0 | \+ 电费 (~$5/年) |
| **Pi 4 (4GB)** | ~$55 | $0 | 推荐 |
| **Pi 5 (4GB)** | ~$60 | $0 | 最佳性能 |
| **Pi 5 (8GB)** | ~$80 | $0 | 性能过剩但面向未来 |
| DigitalOcean | $0 | $6/月 | $72/年 |
| Hetzner | $0 | €3.79/月 | ~$50/年 |

**盈亏平衡点：** 与云 VPS 相比，Pi 在约 6-12 个月内收回成本。

* * *

## 另请参阅

-   [Linux 指南](./linux.md) — 通用 Linux 设置
-   [DigitalOcean 指南](./digitalocean.md) — 云替代方案
-   [Hetzner 指南](../install/hetzner.md) — Docker 设置
-   [Tailscale](../gateway/tailscale.md) — 远程访问
-   [节点](../nodes.md) — 将您的笔记本电脑/手机与 Pi 网关配对

[Oracle Cloud](./oracle.md)[macOS 开发设置](./mac/dev-setup.md)