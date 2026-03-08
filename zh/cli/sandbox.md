

  CLI 命令

  
# 沙盒 CLI

管理基于 Docker 的沙盒容器，用于隔离代理执行。

## 概述

OpenClaw 可以在隔离的 Docker 容器中运行代理以确保安全。`sandbox` 命令可帮助您管理这些容器，特别是在更新或配置更改之后。

## 命令

### openclaw sandbox explain

检查**生效的**沙盒模式/范围/工作空间访问权限、沙盒工具策略以及提升的权限门（附带修复配置键路径）。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

列出所有沙盒容器及其状态和配置。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # 仅列出浏览器容器
openclaw sandbox list --json     # JSON 输出
```

**输出包括：**

-   容器名称和状态（运行中/已停止）
-   Docker 镜像及其是否与配置匹配
-   存在时间（自创建以来的时间）
-   空闲时间（自上次使用以来的时间）
-   关联的会话/代理

### openclaw sandbox recreate

移除沙盒容器以强制使用更新的镜像/配置重新创建。

```bash
openclaw sandbox recreate --all                # 重新创建所有容器
openclaw sandbox recreate --session main       # 特定会话
openclaw sandbox recreate --agent mybot        # 特定代理
openclaw sandbox recreate --browser            # 仅浏览器容器
openclaw sandbox recreate --all --force        # 跳过确认
```

**选项：**

-   `--all`: 重新创建所有沙盒容器
-   `--session `: 为特定会话重新创建容器
-   `--agent `: 为特定代理重新创建容器
-   `--browser`: 仅重新创建浏览器容器
-   `--force`: 跳过确认提示

**重要提示：** 容器将在代理下次使用时自动重新创建。

## 使用场景

### 更新 Docker 镜像后

```bash
# 拉取新镜像
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 更新配置以使用新镜像
# 编辑配置：agents.defaults.sandbox.docker.image (或 agents.list[].sandbox.docker.image)

# 重新创建容器
openclaw sandbox recreate --all
```

### 更改沙盒配置后

```bash
# 编辑配置：agents.defaults.sandbox.* (或 agents.list[].sandbox.*)

# 重新创建以应用新配置
openclaw sandbox recreate --all
```

### 更改 setupCommand 后

```bash
openclaw sandbox recreate --all
# 或仅针对一个代理：
openclaw sandbox recreate --agent family
```

### 仅针对特定代理

```bash
# 仅更新一个代理的容器
openclaw sandbox recreate --agent alfred
```

## 为什么需要这样做？

**问题：** 当您更新沙盒 Docker 镜像或配置时：

-   现有容器继续使用旧设置运行
-   容器仅在闲置 24 小时后才会被清理
-   经常使用的代理会无限期地保持旧容器运行

**解决方案：** 使用 `openclaw sandbox recreate` 强制移除旧容器。它们将在下次需要时自动使用当前设置重新创建。提示：优先使用 `openclaw sandbox recreate` 而非手动 `docker rm`。它使用网关的容器命名方式，并避免在范围/会话键更改时出现不匹配。

## 配置

沙盒设置位于 `~/.openclaw/openclaw.json` 中的 `agents.defaults.sandbox` 下（每个代理的覆盖设置在 `agents.list[].sandbox` 中）：

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... 更多 Docker 选项
        },
        "prune": {
          "idleHours": 24, // 闲置 24 小时后自动清理
          "maxAgeDays": 7, // 7 天后自动清理
        },
      },
    },
  },
}
```

## 另请参阅

-   [沙盒文档](../gateway/sandboxing.md)
-   [代理配置](../concepts/agent-workspace.md)
-   [Doctor 命令](../gateway/doctor.md) - 检查沙盒设置

[reset](./reset.md)[secrets](./secrets.md)

---