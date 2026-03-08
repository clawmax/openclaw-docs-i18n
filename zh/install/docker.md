

  其他安装方式

  
# Docker

Docker 是**可选的**。仅在你需要容器化网关或验证 Docker 流程时使用。

## Docker 适合我吗？

-   **适合**：你想要一个隔离的、可丢弃的网关环境，或者想在无需本地安装的主机上运行 OpenClaw。
-   **不适合**：你在自己的机器上运行，只想要最快的开发循环。请改用普通安装流程。
-   **沙箱化说明**：代理沙箱化也使用 Docker，但它**不**要求整个网关在 Docker 中运行。参见[沙箱化](../gateway/sandboxing.md)。

本指南涵盖：

-   容器化网关（完整的 OpenClaw 在 Docker 中）
-   按会话代理沙箱（主机网关 + Docker 隔离的代理工具）

沙箱化详情：[沙箱化](../gateway/sandboxing.md)

## 要求

-   Docker Desktop（或 Docker Engine）+ Docker Compose v2
-   至少 2 GB 内存用于镜像构建（在 1 GB 内存的主机上，`pnpm install` 可能因 OOM 被终止，退出码 137）
-   足够的磁盘空间用于镜像 + 日志
-   如果在 VPS/公共主机上运行，请查看[网络暴露的安全加固](../gateway/security.md#04-network-exposure-bind--port--firewall)，尤其是 Docker `DOCKER-USER` 防火墙策略。

## 容器化网关（Docker Compose）

### 快速开始（推荐）

> **ℹ️** 此处的 Docker 默认值假设绑定模式（`lan`/`loopback`），而非主机别名。在 `gateway.bind` 中使用绑定模式值（例如 `lan` 或 `loopback`），而不是主机别名如 `0.0.0.0` 或 `localhost`。

从仓库根目录运行：

```
./docker-setup.sh
```

此脚本：

-   在本地构建网关镜像（如果设置了 `OPENCLAW_IMAGE` 则拉取远程镜像）
-   运行入门向导
-   打印可选的提供商设置提示
-   通过 Docker Compose 启动网关
-   生成网关令牌并写入 `.env`

可选环境变量：

-   `OPENCLAW_IMAGE` — 使用远程镜像而非本地构建（例如 `ghcr.io/openclaw/openclaw:latest`）
-   `OPENCLAW_DOCKER_APT_PACKAGES` — 在构建期间安装额外的 apt 包
-   `OPENCLAW_EXTENSIONS` — 在构建时预安装扩展依赖项（空格分隔的扩展名，例如 `diagnostics-otel matrix`）
-   `OPENCLAW_EXTRA_MOUNTS` — 添加额外的主机绑定挂载
-   `OPENCLAW_HOME_VOLUME` — 在命名卷中持久化 `/home/node`
-   `OPENCLAW_SANDBOX` — 选择启用 Docker 网关沙箱引导。仅明确的真值会启用它：`1`、`true`、`yes`、`on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — 本地镜像构建的构建参数透传（`1` 会在镜像中安装 Docker CLI）。`docker-setup.sh` 在 `OPENCLAW_SANDBOX=1` 时自动设置此值用于本地构建。
-   `OPENCLAW_DOCKER_SOCKET` — 覆盖 Docker 套接字路径（默认：`DOCKER_HOST=unix://...` 路径，否则为 `/var/run/docker.sock`）
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — 紧急措施：允许受信任的私有网络 `ws://` 目标用于 CLI/入门客户端路径（默认仅限环回）
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — 当你需要 WebGL/3D 兼容性时，禁用容器浏览器加固标志 `--disable-3d-apis`、`--disable-software-rasterizer`、`--disable-gpu`。
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — 当浏览器流程需要扩展时保持扩展启用（默认在沙箱浏览器中禁用扩展）。
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — 设置 Chromium 渲染器进程限制；设置为 `0` 以跳过该标志并使用 Chromium 默认行为。

完成后：

-   在浏览器中打开 `http://127.0.0.1:18789/`。
-   将令牌粘贴到控制界面（设置 → 令牌）。
-   需要再次获取 URL？运行 `docker compose run --rm openclaw-cli dashboard --no-open`。

### 为 Docker 网关启用代理沙箱（选择启用）

`docker-setup.sh` 也可以为 Docker 部署引导 `agents.defaults.sandbox.*`。通过以下方式启用：

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

自定义套接字路径（例如 rootless Docker）：

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

注意：

-   该脚本仅在沙箱先决条件通过后才挂载 `docker.sock`。
-   如果沙箱设置无法完成，脚本会将 `agents.defaults.sandbox.mode` 重置为 `off`，以避免在重新运行时出现陈旧/损坏的沙箱配置。
-   如果 `Dockerfile.sandbox` 缺失，脚本会打印警告并继续；如果需要，使用 `scripts/sandbox-setup.sh` 构建 `openclaw-sandbox:bookworm-slim`。
-   对于非本地的 `OPENCLAW_IMAGE` 值，镜像必须已包含用于沙箱执行的 Docker CLI 支持。

### 自动化/CI（非交互式，无 TTY 输出）

对于脚本和 CI，使用 `-T` 禁用 Compose 伪 TTY 分配：

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

如果你的自动化没有导出 Claude 会话变量，现在将它们留空不设置，在 `docker-compose.yml` 中默认解析为空值，以避免重复的“变量未设置”警告。

### 共享网络安全说明（CLI + 网关）

`openclaw-cli` 使用 `network_mode: "service:openclaw-gateway"`，以便 CLI 命令在 Docker 中可以通过 `127.0.0.1` 可靠地访问网关。将此视为共享信任边界：环回绑定并不是这两个容器之间的隔离。如果你需要更强的隔离，请从单独的容器/主机网络路径运行命令，而不是使用捆绑的 `openclaw-cli` 服务。为了减少 CLI 进程被攻破时的影响，compose 配置删除了 `NET_RAW`/`NET_ADMIN` 并在 `openclaw-cli` 上启用了 `no-new-privileges`。它在主机上写入配置/工作空间：

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

在 VPS 上运行？参见 [Hetzner (Docker VPS)](./hetzner.md)。

### 使用远程镜像（跳过本地构建）

官方预构建镜像发布在：

-   [GitHub Container Registry 包](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

使用镜像名称 `ghcr.io/openclaw/openclaw`（不是 Docker Hub 上名称类似的镜像）。常见标签：

-   `main` — 来自 `main` 分支的最新构建
-   `` — 发布标签构建（例如 `2026.2.26`）
-   `latest` — 最新的稳定发布标签

### 基础镜像元数据

主 Docker 镜像当前使用：

-   `node:22-bookworm`

Docker 镜像现在发布 OCI 基础镜像注解（sha256 是示例）：

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=OpenClaw 网关和 CLI 运行时容器镜像`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

参考：[OCI 镜像注解](https://github.com/opencontainers/image-spec/blob/main/annotations.md) 发布上下文：此仓库的标签历史在 `v2026.2.22` 及更早的 2026 标签（例如 `v2026.2.21`、`v2026.2.9`）中已经使用 Bookworm。默认情况下，设置脚本从源代码构建镜像。要拉取预构建的镜像，请在运行脚本前设置 `OPENCLAW_IMAGE`：

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

脚本检测到 `OPENCLAW_IMAGE` 不是默认的 `openclaw:local`，并运行 `docker pull` 而不是 `docker build`。其他所有操作（入门、网关启动、令牌生成）的工作方式相同。`docker-setup.sh` 仍然从仓库根目录运行，因为它使用本地的 `docker-compose.yml` 和辅助文件。`OPENCLAW_IMAGE` 跳过了本地镜像构建时间；它不会替换 compose/设置工作流。

### Shell 助手（可选）

为了更轻松的日常 Docker 管理，安装 `ClawDock`：

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**添加到你的 shell 配置（zsh）：**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

然后使用 `clawdock-start`、`clawdock-stop`、`clawdock-dashboard` 等命令。运行 `clawdock-help` 查看所有命令。详情参见 [`ClawDock` 助手 README](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md)。

### 手动流程（compose）

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

注意：从仓库根目录运行 `docker compose ...`。如果你启用了 `OPENCLAW_EXTRA_MOUNTS` 或 `OPENCLAW_HOME_VOLUME`，设置脚本会写入 `docker-compose.extra.yml`；在其他地方运行 Compose 时包含它：

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### 控制界面令牌 + 配对（Docker）

如果你看到“未授权”或“已断开连接 (1008)：需要配对”，获取新的仪表板链接并批准浏览器设备：

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

更多详情：[仪表板](../web/dashboard.md)、[设备](../cli/devices.md)。

### 额外挂载（可选）

如果你想将额外的主机目录挂载到容器中，在运行 `docker-setup.sh` 前设置 `OPENCLAW_EXTRA_MOUNTS`。这接受一个逗号分隔的 Docker 绑定挂载列表，并通过生成 `docker-compose.extra.yml` 将它们应用到 `openclaw-gateway` 和 `openclaw-cli`。示例：

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意：

-   路径必须在 macOS/Windows 上与 Docker Desktop 共享。
-   每个条目必须是 `source:target[:options]` 格式，没有空格、制表符或换行符。
-   如果你编辑了 `OPENCLAW_EXTRA_MOUNTS`，重新运行 `docker-setup.sh` 以重新生成额外的 compose 文件。
-   `docker-compose.extra.yml` 是生成的。不要手动编辑它。

### 持久化整个容器主目录（可选）

如果你希望 `/home/node` 在容器重新创建后持久存在，通过 `OPENCLAW_HOME_VOLUME` 设置一个命名卷。这会创建一个 Docker 卷并将其挂载到 `/home/node`，同时保留标准的配置/工作空间绑定挂载。在此处使用命名卷（不是绑定路径）；对于绑定挂载，请使用 `OPENCLAW_EXTRA_MOUNTS`。示例：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

你可以将此与额外挂载结合：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意：

-   命名卷必须匹配 `^[A-Za-z0-9][A-Za-z0-9_.-]*$`。
-   如果你更改了 `OPENCLAW_HOME_VOLUME`，重新运行 `docker-setup.sh` 以重新生成额外的 compose 文件。
-   命名卷会一直存在，直到使用 `docker volume rm ` 删除。

### 安装额外的 apt 包（可选）

如果你需要在镜像内安装系统包（例如，构建工具或媒体库），在运行 `docker-setup.sh` 前设置 `OPENCLAW_DOCKER_APT_PACKAGES`。这会在镜像构建期间安装这些包，因此即使容器被删除，它们也会持久存在。示例：

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

注意：

-   这接受一个空格分隔的 apt 包名列表。
-   如果你更改了 `OPENCLAW_DOCKER_APT_PACKAGES`，重新运行 `docker-setup.sh` 以重新构建镜像。

### 预安装扩展依赖项（可选）

具有自己的 `package.json` 的扩展（例如 `diagnostics-otel`、`matrix`、`msteams`）在首次加载时安装其 npm 依赖项。要将这些依赖项烘焙到镜像中，请在运行 `docker-setup.sh` 前设置 `OPENCLAW_EXTENSIONS`：

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

或者直接构建时：

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

注意：

-   这接受一个空格分隔的扩展目录名列表（位于 `extensions/` 下）。
-   仅影响具有 `package.json` 的扩展；没有 `package.json` 的轻量级插件会被忽略。
-   如果你更改了 `OPENCLAW_EXTENSIONS`，重新运行 `docker-setup.sh` 以重新构建镜像。

### 高级用户 / 全功能容器（选择启用）

默认的 Docker 镜像**安全第一**，并以非 root 用户 `node` 运行。这保持了较小的攻击面，但也意味着：

-   运行时无法安装系统包
-   默认没有 Homebrew
-   没有捆绑的 Chromium/Playwright 浏览器

如果你想要一个功能更全的容器，请使用这些选择启用的开关：

1.  **持久化 `/home/node`** 以便浏览器下载和工具缓存得以保留：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **将系统依赖项烘焙到镜像中**（可重复 + 持久）：

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **无需 `npx` 安装 Playwright 浏览器**（避免 npm 覆盖冲突）：

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

如果你需要 Playwright 安装系统依赖项，请使用 `OPENCLAW_DOCKER_APT_PACKAGES` 重新构建镜像，而不是在运行时使用 `--with-deps`。

4.  **持久化 Playwright 浏览器下载**：

-   在 `docker-compose.yml` 中设置 `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright`。
-   通过 `OPENCLAW_HOME_VOLUME` 确保 `/home/node` 持久化，或者通过 `OPENCLAW_EXTRA_MOUNTS` 挂载 `/home/node/.cache/ms-playwright`。

### 权限 + EACCES

镜像以 `node`（uid 1000）运行。如果你在 `/home/node/.openclaw` 上看到权限错误，请确保你的主机绑定挂载归 uid 1000 所有。示例（Linux 主机）：

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

如果你为了方便选择以 root 身份运行，你接受了安全权衡。

### 更快的重建（推荐）

为了加速重建，请对你的 Dockerfile 排序，以便依赖层被缓存。这避免了除非锁定文件更改，否则重新运行 `pnpm install`：

```dockerfile
FROM node:22-bookworm

# 安装 Bun（构建脚本所需）
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# 除非包元数据更改，否则缓存依赖项
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### 频道设置（可选）

使用 CLI 容器配置频道，然后根据需要重启网关。WhatsApp（二维码）：

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram（机器人令牌）：

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord（机器人令牌）：

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

文档：[WhatsApp](../channels/whatsapp.md)、[Telegram](../channels/telegram.md)、[Discord](../channels/discord.md)

### OpenAI Codex OAuth（无头 Docker）

如果你在向导中选择 OpenAI Codex OAuth，它会打开一个浏览器 URL 并尝试在 `http://127.0.0.1:1455/auth/callback` 捕获回调。在 Docker 或无头设置中，该回调可能会显示浏览器错误。复制你最终到达的完整重定向 URL 并将其粘贴回向导以完成身份验证。

### 健康检查

容器探测端点（无需身份验证）：

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

别名：`/health` 和 `/ready`。`/healthz` 是一个浅层存活探针，用于检查“网关进程是否启动”。`/readyz` 在启动宽限期内保持就绪状态，然后仅在所需的托管频道在宽限期后仍然断开连接或稍后断开连接时才变为 `503`。Docker 镜像包含一个内置的 `HEALTHCHECK`，它在后台 ping `/healthz`。简单来说：Docker 持续检查 OpenClaw 是否仍然响应。如果检查持续失败，Docker 将容器标记为 `unhealthy`，编排系统（Docker Compose 重启策略、Swarm、Kubernetes 等）可以自动重启或替换它。经过身份验证的深度健康快照（网关 + 频道）：

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### E2E 冒烟测试（Docker）

```
scripts/e2e/onboard-docker.sh
```

### 二维码导入冒烟测试（Docker）

```bash
pnpm test:docker:qr
```

### LAN 与环回（Docker Compose）

`docker-setup.sh` 默认设置 `OPENCLAW_GATEWAY_BIND=lan`，以便主机访问 `http://127.0.0.1:18789` 能与 Docker 端口发布一起工作。

-   `lan`（默认）：主机浏览器 + 主机 CLI 可以访问已发布的网关端口。
-   `loopback`：只有容器网络命名空间内的进程可以直接访问网关；主机发布的端口访问可能会失败。

设置脚本还在入门后将 `gateway.mode=local` 固定，以便 Docker CLI 命令默认以本地环回为目标。遗留配置说明：在 `gateway.bind` 中使用绑定模式值（`lan` / `loopback` / `custom` / `tailnet` / `auto`），而不是主机别名（`0.0.0.0`、`127.0.0.1`、`localhost`、`::`、`::1`）。如果你从 Docker CLI 命令看到 `Gateway target: ws://172.x.x.x:18789` 或重复的 `pairing required` 错误，请运行：

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### 注意事项

-   网关绑定默认为 `lan` 以供容器使用（`OPENCLAW_GATEWAY_BIND`）。
-   Dockerfile CMD 使用 `--allow-unconfigured`；即使挂载的配置中 `gateway.mode` 不是 `local`，仍会启动。覆盖 CMD 以强制执行此防护。
-   网关容器是会话（`~/.openclaw/agents//sessions/`）的单一事实来源。

### 存储模型

-   **持久化主机数据：** Docker Compose 将 `OPENCLAW_CONFIG_DIR` 绑定挂载到 `/home/node/.openclaw`，将 `OPENCLAW_WORKSPACE_DIR` 绑定挂载到 `/home/node/.openclaw/workspace`，因此这些路径在容器替换后得以保留。
-   **临时沙箱 tmpfs：** 当启用 `agents.defaults.sandbox` 时，沙箱容器对 `/tmp`、`/var/tmp` 和 `/run` 使用 `tmpfs`。这些挂载与顶层的 Compose 堆栈是分开的，并随沙箱容器一起消失。
-   **磁盘增长热点：** 注意 `media/`、`agents//sessions/sessions.json`、转录 JSONL 文件、`cron/runs/*.jsonl` 以及 `/tmp/openclaw/` 下的滚动文件日志（或你配置的 `logging.file`）。如果你还在 Docker 外运行 macOS 应用，其服务日志又是分开的：`~/.openclaw/logs/gateway.log`、`~/.openclaw/logs/gateway.err.log` 和 `/tmp/openclaw/openclaw-gateway.log`。

## 代理沙箱（主机网关 + Docker 工具）

深入探讨：[沙箱化](../gateway/sandboxing.md)

### 功能说明

当启用 `agents.defaults.sandbox` 时，**非主会话**在 Docker 容器内运行工具。网关保留在你的主机上，但工具执行是隔离的：

-   作用域：默认为 `"agent"`（每个代理一个容器 + 工作空间）
-   作用域：`"session"` 用于按会话隔离
-   每个作用域的工作空间文件夹挂载在 `/workspace`
-   可选的代理工作空间访问（`agents.defaults.sandbox.workspaceAccess`）
-   允许/拒绝工具策略（拒绝优先）
-   入站媒体被复制到活动沙箱工作空间（`media/inbound/*`）中，以便工具可以读取它（使用 `workspaceAccess: "rw"` 时，这会进入代理工作空间）

警告：`scope: "shared"` 禁用了跨会话隔离。所有会话共享一个容器和一个工作空间。

### 按代理沙箱配置文件（多代理）

如果你使用多代理路由，每个代理可以覆盖沙箱 + 工具设置：`agents.list[].sandbox` 和 `agents.list[].tools`（以及 `agents.list[].tools.sandbox.tools`）。这让你可以在一个网关中运行混合访问级别：

-   完全访问（个人代理）
-   只读工具 + 只读工作空间（家庭/工作代理）
-   无文件系统/Shell 工具（公共代理）

示例、优先级和故障排除参见[多代理沙箱与工具](../tools/multi-agent-sandbox-tools.md)。

### 默认行为

-   镜像：`openclaw-sandbox:bookworm-slim`
-   每个代理一个容器
-   代理工作空间访问：`workspaceAccess: "none"`（默认）使用 `~/.openclaw/sandboxes`
    -   `"ro"` 将沙箱工作空间保留在 `/workspace`，并以只读方式挂载代理工作空间到 `/agent`（禁用 `write`/`edit`/`apply_patch`）
    -   `"rw"` 以读写方式挂载代理工作空间到 `/workspace`
-   自动清理：空闲 > 24 小时 或 存在时间 > 7 天
-   网络：默认为 `none`（如果你需要出口流量，请明确选择启用）
    -   `host` 被阻止。
    -   `container:` 默认被阻止（命名空间加入风险）。
-   默认允许：`exec`、`process`、`read`、`write`、`edit`、`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
-   默认拒绝：`browser`、`canvas`、`nodes`、`cron`、`discord`、`gateway`

### 启用沙箱化

如果你计划在 `setupCommand` 中安装包，请注意：

-   默认 `docker.network` 是 `"none"`（无出口流量）。
-   `docker.network: "host"` 被阻止。
-   `docker.network: "container:"` 默认被阻止。
-   紧急覆盖：`agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`。
-   `readOnlyRoot: true` 会阻止包安装。
-   `user` 必须是 root 才能使用 `apt-get`（省略 `user` 或设置 `user: "0:0"`）。OpenClaw 会在 `setupCommand`（或 docker 配置）更改时自动重新创建容器，除非容器**最近被使用过**（约 5 分钟内）。热容器会记录警告，并显示确切的 `openclaw sandbox recreate ...` 命令。

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent 是默认值)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 禁用空闲清理
          maxAgeDays: 7, // 0 禁用最长存在时间清理
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

加固开关位于 `agents.defaults.sandbox.docker` 下：`network`、`user`、`pidsLimit`、`memory`、`memorySwap`、`cpus`、`ulimits`、`seccompProfile`、`apparmorProfile`、`dns`、`extraHosts`、`dangerouslyAllowContainerNamespaceJoin`（仅限紧急情况）。多代理：通过 `agents.list[].sandbox.{docker,browser,prune}.*` 覆盖每个代理的 `agents.defaults.sandbox.{docker,browser,prune}.*`（当 `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` 为 `"shared"` 时忽略）。

### 构建默认沙箱镜像

```
scripts/sandbox-setup.sh
```

这会使用 `Dockerfile.sandbox` 构建 `openclaw-sandbox:bookworm-slim`。

### 沙箱通用镜像（可选）

如果你想要一个包含常见构建工具（Node、Go、Rust 等）的沙箱镜像，请构建通用镜像：

```
scripts/sandbox-common-setup.sh
```

这会构建 `openclaw-sandbox-common:bookworm-slim`。要使用它：

```json
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } },
    },
  },
}
```

### 沙箱浏览器镜像

要在沙箱内运行浏览器工具，请构建浏览器镜像：

```
scripts/sandbox-browser-setup.sh
```

这会使用 `Dockerfile.sandbox-browser` 构建 `openclaw-sandbox-browser:bookworm-slim`。容器运行启用了 CDP 的 Chromium 和可选的 noVNC 观察器（通过 Xvfb 实现有头模式）。注意：

-   有头模式（Xvfb）相比无头模式减少了机器人拦截。
-   通过设置 `agents.defaults.sandbox.browser.headless=true` 仍可使用无头模式。
-   不需要完整的桌面环境（GNOME）；Xvfb 提供显示。
-   浏览器容器默认使用专用的 Docker 网络（`openclaw-sandbox-browser`），而不是全局的 `bridge`。
-   可选的 `agents.defaults.sandbox.browser.cdpSourceRange` 通过 CIDR 限制容器边缘的 CDP 入口（例如 `172.21.0.1/32`）。
-   noVNC 观察器访问默认受密码保护；OpenClaw 提供一个短期的观察器令牌 URL，该 URL 提供一个本地引导页面，并将密码保留在 URL 片段中（而不是 URL 查询参数）。
-   浏览器容器启动默认值对于共享/容器工作负载是保守的，包括：
    -   `--remote-debugging-address=127.0.0.1`
    -   `--remote-debugging-port=`
    -   `--user-data-dir=${HOME}/.chrome`
    -   `--no-first-run`
    -   `--no-default-browser-check`
    -   `--disable-3d-apis`
    -   `--disable-software-rasterizer`
    -   `--disable-gpu`
    -   `--disable-dev-shm-usage`
    -   `--disable-background-networking`
    -   `--disable-features=TranslateUI`
    -   `--disable-breakpad`
    -   `--disable-crash-reporter`
    -   `--metrics-recording-only`
    -   `--renderer-process-limit=2`
    -   `--no-zygote`
    -   `--disable-extensions`
    -   如果设置了 `agents.defaults.sandbox.browser.noSandbox`，还会追加 `--no-sandbox` 和 `--disable-setuid-sandbox`。
    -   上述三个图形加固标志是可选的。如果你的工作负载需要 WebGL/3D，请设置 `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` 以在没有 `--disable-3d-apis`、`--disable-software-rasterizer` 和 `--disable-gpu` 的情况下运行。
    -   扩展行为由 `--disable-extensions` 控制，可以通过 `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` 禁用（启用扩展）以支持依赖扩展的页面或扩展密集型工作流。
    -   `--renderer-process-limit=2` 也可以通过 `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT` 配置；设置为 `0` 以让 Chromium 在需要调整浏览器并发性时选择其默认的进程限制。

默认值在捆绑的镜像中默认应用。如果你需要不同的 Chromium 标志，请使用自定义浏览器镜像并提供你自己的入口点。使用配置：

```json
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true },
      },
    },
  },
}
```

自定义浏览器镜像：

```json
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" }