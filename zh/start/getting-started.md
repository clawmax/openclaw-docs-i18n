

  初始步骤

  
# 入门指南

目标：从零开始，以最小设置完成首次聊天。

> **ℹ️** 最快聊天方式：打开控制界面（无需设置频道）。运行 `openclaw dashboard` 并在浏览器中聊天，或在网关主机上打开 `http://127.0.0.1:18789/`。文档：[仪表板](../web/dashboard.md) 和 [控制界面](../web/control-ui.md)。

## 前提条件

-   Node 22 或更新版本

> **💡** 如果不确定，请使用 `node --version` 检查你的 Node 版本。

## 快速设置 (CLI)

### 步骤 1：安装 OpenClaw (推荐)

> **ℹ️** 其他安装方法和要求：[安装](../install.md)。

### 步骤 2：运行入门向导

```bash
openclaw onboard --install-daemon
```

该向导将配置认证、网关设置和可选频道。详情请参阅 [入门向导](./wizard.md)。

### 步骤 3：检查网关

如果你安装了服务，它应该已经在运行：

```bash
openclaw gateway status
```

### 步骤 4：打开控制界面

```bash
openclaw dashboard
```

 

> **✅** 如果控制界面加载成功，你的网关已准备就绪。

## 可选检查与额外操作

适用于快速测试或故障排除。

```bash
openclaw gateway --port 18789
```

需要已配置的频道。

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## 有用的环境变量

如果你以服务账户运行 OpenClaw 或需要自定义配置/状态存储位置：

-   `OPENCLAW_HOME` 设置用于内部路径解析的主目录。
-   `OPENCLAW_STATE_DIR` 覆盖状态目录。
-   `OPENCLAW_CONFIG_PATH` 覆盖配置文件路径。

完整环境变量参考：[环境变量](../help/environment.md)。

## 深入了解

## 你将获得

-   一个正在运行的网关
-   已配置的认证
-   控制界面访问权限或一个已连接的频道

## 后续步骤

-   私信安全与审批：[配对](../channels/pairing.md)
-   连接更多频道：[频道](../channels.md)
-   高级工作流和从源码安装：[设置](./setup.md)

[功能特性](../concepts/features.md)[初始设置概览](./onboarding-overview.md)

---