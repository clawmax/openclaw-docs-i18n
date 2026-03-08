

  平台概览

  
# Windows (WSL2)

在 Windows 上运行 OpenClaw 推荐**通过 WSL2**（推荐 Ubuntu）。CLI + 网关在 Linux 内部运行，这保持了运行时环境的一致性，并使工具链（Node/Bun/pnpm、Linux 二进制文件、技能）兼容性更好。原生 Windows 可能更棘手。WSL2 为您提供完整的 Linux 体验 —— 只需一条命令即可安装：`wsl --install`。原生 Windows 配套应用已在规划中。

## 安装 (WSL2)

-   [入门指南](../start/getting-started.md)（在 WSL 内使用）
-   [安装与更新](../install/updating.md)
-   官方 WSL2 指南 (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## 网关

-   [网关运行手册](../gateway.md)
-   [配置](../gateway/configuration.md)

## 网关服务安装 (CLI)

在 WSL2 内：

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

提示时选择 **Gateway service**。修复/迁移：

```bash
openclaw doctor
```

## 在 Windows 登录前自动启动网关

对于无头设置，确保即使无人登录 Windows，完整的启动链也能运行。

### 1) 保持用户服务在未登录状态下运行

在 WSL 内：

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) 安装 OpenClaw 网关用户服务

在 WSL 内：

```bash
openclaw gateway install
```

### 3) 在 Windows 启动时自动启动 WSL

在 PowerShell 中以管理员身份运行：

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

将 `Ubuntu` 替换为您的发行版名称，可通过以下命令查看：

```bash
wsl --list --verbose
```

### 验证启动链

重启后（在 Windows 登录之前），从 WSL 内检查：

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## 高级：通过局域网暴露 WSL 服务 (portproxy)

WSL 有自己的虚拟网络。如果另一台机器需要访问**在 WSL 内**运行的服务（SSH、本地 TTS 服务器或网关），您必须将 Windows 的一个端口转发到当前 WSL 的 IP 地址。WSL 的 IP 在重启后会改变，因此您可能需要刷新转发规则。示例（在 PowerShell 中**以管理员身份**运行）：

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

允许端口通过 Windows 防火墙（一次性操作）：

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL 重启后刷新 portproxy：

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

注意事项：

-   从另一台机器 SSH 时，目标是 **Windows 主机的 IP**（例如：`ssh user@windows-host -p 2222`）。
-   远程节点必须指向一个**可访问的**网关 URL（不是 `127.0.0.1`）；使用 `openclaw status --all` 来确认。
-   使用 `listenaddress=0.0.0.0` 允许局域网访问；`127.0.0.1` 则仅限本地访问。
-   如果您希望此过程自动化，可以注册一个计划任务，在登录时运行刷新步骤。

## 分步 WSL2 安装

### 1) 安装 WSL2 + Ubuntu

打开 PowerShell（管理员）：

```bash
wsl --install
# 或者显式选择一个发行版：
wsl --list --online
wsl --install -d Ubuntu-24.04
```

如果 Windows 提示，请重启。

### 2) 启用 systemd（网关安装所需）

在您的 WSL 终端中：

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

然后在 PowerShell 中：

```bash
wsl --shutdown
```

重新打开 Ubuntu，然后验证：

```bash
systemctl --user status
```

### 3) 安装 OpenClaw（在 WSL 内）

在 WSL 内遵循 Linux 入门流程：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时会自动安装 UI 依赖
pnpm build
openclaw onboard
```

完整指南：[入门指南](../start/getting-started.md)

## Windows 配套应用

我们目前还没有 Windows 配套应用。如果您希望贡献代码使其实现，欢迎贡献。

[Linux 应用](./linux.md)[Android 应用](./android.md)