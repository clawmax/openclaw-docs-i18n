

  托管与部署

  
# macOS 虚拟机

## 推荐的默认方案（适合大多数用户）

-   **小型 Linux VPS**，用于保持网关始终在线且成本低廉。请参阅 [VPS 托管](../vps.md)。
-   **专用硬件**（Mac mini 或 Linux 主机），如果你想要完全控制以及用于浏览器自动化的**住宅 IP**。许多网站会屏蔽数据中心 IP，因此本地浏览通常效果更好。
-   **混合方案：** 将网关保留在廉价的 VPS 上，当你需要浏览器/UI 自动化时，将你的 Mac 作为**节点**连接。请参阅 [节点](../nodes.md) 和 [远程网关](../gateway/remote.md)。

当你特别需要仅限 macOS 的功能（iMessage/BlueBubbles）或希望与你的日常 Mac 严格隔离时，请使用 macOS 虚拟机。

## macOS 虚拟机选项

### 在你的 Apple Silicon Mac 上运行本地虚拟机（Lume）

使用 [Lume](https://cua.ai/docs/lume) 在你现有的 Apple Silicon Mac 上运行沙盒化的 macOS 虚拟机。这为你提供：

-   完全隔离的 macOS 环境（你的主机保持清洁）
-   通过 BlueBubbles 支持 iMessage（在 Linux/Windows 上无法实现）
-   通过克隆虚拟机实现即时重置
-   无需额外的硬件或云成本

### 托管 Mac 提供商（云端）

如果你想要云端的 macOS，托管 Mac 提供商也可以：

-   [MacStadium](https://www.macstadium.com/)（托管 Mac）
-   其他托管 Mac 供应商也适用；请遵循他们的虚拟机 + SSH 文档

一旦你获得对 macOS 虚拟机的 SSH 访问权限，请继续下面的步骤 6。

* * *

## 快速路径（Lume，有经验的用户）

1.  安装 Lume
2.  `lume create openclaw --os macos --ipsw latest`
3.  完成设置助理，启用远程登录（SSH）
4.  `lume run openclaw --no-display`
5.  SSH 进入，安装 OpenClaw，配置频道
6.  完成

* * *

## 所需条件（Lume）

-   Apple Silicon Mac（M1/M2/M3/M4）
-   主机上运行 macOS Sequoia 或更高版本
-   每个虚拟机约 60 GB 可用磁盘空间
-   约 20 分钟

* * *

## 1) 安装 Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

如果 `~/.local/bin` 不在你的 PATH 中：

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

验证：

```bash
lume --version
```

文档：[Lume 安装](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) 创建 macOS 虚拟机

```bash
lume create openclaw --os macos --ipsw latest
```

这将下载 macOS 并创建虚拟机。VNC 窗口会自动打开。注意：下载时间可能因你的网络连接而异。

* * *

## 3) 完成设置助理

在 VNC 窗口中：

1.  选择语言和地区
2.  跳过 Apple ID（或者如果你想稍后使用 iMessage 可以登录）
3.  创建一个用户帐户（记住用户名和密码）
4.  跳过所有可选功能

设置完成后，启用 SSH：

1.  打开系统设置 → 通用 → 共享
2.  启用“远程登录”

* * *

## 4) 获取虚拟机的 IP 地址

```bash
lume get openclaw
```

查找 IP 地址（通常是 `192.168.64.x`）。

* * *

## 5) SSH 连接到虚拟机

```bash
ssh youruser@192.168.64.X
```

将 `youruser` 替换为你创建的帐户，并将 IP 替换为你的虚拟机 IP。

* * *

## 6) 安装 OpenClaw

在虚拟机内部：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

按照引导提示设置你的模型提供商（Anthropic、OpenAI 等）。

* * *

## 7) 配置频道

编辑配置文件：

```bash
nano ~/.openclaw/openclaw.json
```

添加你的频道：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

然后登录 WhatsApp（扫描二维码）：

```bash
openclaw channels login
```

* * *

## 8) 无头运行虚拟机

停止虚拟机并无显示重启：

```bash
lume stop openclaw
lume run openclaw --no-display
```

虚拟机在后台运行。OpenClaw 的守护进程保持网关运行。检查状态：

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## 额外功能：iMessage 集成

这是在 macOS 上运行的杀手级功能。使用 [BlueBubbles](https://bluebubbles.app) 将 iMessage 添加到 OpenClaw。在虚拟机内部：

1.  从 bluebubbles.app 下载 BlueBubbles
2.  使用你的 Apple ID 登录
3.  启用 Web API 并设置密码
4.  将 BlueBubbles webhook 指向你的网关（例如：`https://your-gateway-host:3000/bluebubbles-webhook?password=`）

添加到你的 OpenClaw 配置中：

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

重启网关。现在你的代理可以发送和接收 iMessage 了。完整设置详情：[BlueBubbles 频道](../channels/bluebubbles.md)

* * *

## 保存一个黄金镜像

在进一步自定义之前，快照你的干净状态：

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

随时重置：

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## 24/7 运行

通过以下方式保持虚拟机运行：

-   保持你的 Mac 接通电源
-   在系统设置 → 节能中禁用睡眠
-   如果需要，使用 `caffeinate`

对于真正的始终在线，请考虑使用专用的 Mac mini 或小型 VPS。请参阅 [VPS 托管](../vps.md)。

* * *

## 故障排除

| 问题 | 解决方案 |
| --- | --- |
| 无法 SSH 连接到虚拟机 | 检查虚拟机系统设置中是否启用了“远程登录” |
| 虚拟机 IP 未显示 | 等待虚拟机完全启动，再次运行 `lume get openclaw` |
| 找不到 Lume 命令 | 将 `~/.local/bin` 添加到你的 PATH |
| WhatsApp 二维码无法扫描 | 确保运行 `openclaw channels login` 时已登录到虚拟机（而非主机） |

* * *

## 相关文档

-   [VPS 托管](../vps.md)
-   [节点](../nodes.md)
-   [远程网关](../gateway/remote.md)
-   [BlueBubbles 频道](../channels/bluebubbles.md)
-   [Lume 快速入门](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [Lume CLI 参考](https://cua.ai/docs/lume/reference/cli-reference)
-   [无人值守虚拟机设置](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup)（高级）
-   [Docker 沙盒化](./docker.md)（替代隔离方案）

[GCP](./gcp.md)[exe.dev](./exe-dev.md)