

  远程访问

  
# 远程网关设置

OpenClaw.app 使用 SSH 隧道连接到远程网关。本指南将向您展示如何进行设置。

## 概述

## 快速设置

### 步骤 1：添加 SSH 配置

编辑 `~/.ssh/config` 并添加：

```
Host remote-gateway
    HostName <REMOTE_IP>          # e.g., 172.27.187.184
    User <REMOTE_USER>            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

将 `<REMOTE_IP>` 和 `<REMOTE_USER>` 替换为您的实际值。

### 步骤 2：复制 SSH 密钥

将您的公钥复制到远程机器（只需输入一次密码）：

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### 步骤 3：设置网关令牌

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### 步骤 4：启动 SSH 隧道

```bash
ssh -N remote-gateway &
```

### 步骤 5：重启 OpenClaw.app

```bash
# 退出 OpenClaw.app (⌘Q)，然后重新打开：
open /path/to/OpenClaw.app
```

现在，应用程序将通过 SSH 隧道连接到远程网关。

* * *

## 登录时自动启动隧道

要让 SSH 隧道在您登录时自动启动，请创建一个启动代理。

### 创建 PLIST 文件

将此文件保存为 `~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### 加载启动代理

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

现在，隧道将：

-   在您登录时自动启动
-   如果崩溃会自动重启
-   在后台持续运行

遗留说明：如果存在旧的 `com.openclaw.ssh-tunnel` LaunchAgent，请将其移除。

* * *

## 故障排除

**检查隧道是否正在运行：**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**重启隧道：**

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**停止隧道：**

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

* * *

## 工作原理

| 组件 | 功能说明 |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | 将本地端口 18789 转发到远程端口 18789 |
| `ssh -N` | SSH 不执行远程命令（仅用于端口转发） |
| `KeepAlive` | 如果隧道崩溃，自动重启 |
| `RunAtLoad` | 代理加载时启动隧道 |

OpenClaw.app 连接到您客户端机器上的 `ws://127.0.0.1:18789`。SSH 隧道将该连接转发到运行网关的远程机器的 18789 端口。

[远程访问](./remote.md)[Tailscale](./tailscale.md)