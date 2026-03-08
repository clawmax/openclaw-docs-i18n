

  托管与部署

  
# GCP

## 目标

在 GCP Compute Engine 虚拟机上使用 Docker 运行一个持久化的 OpenClaw Gateway，具备持久化状态、预置的二进制文件以及安全的重启行为。如果你想要“每月约 5-12 美元的 24/7 OpenClaw 服务”，这是在 Google Cloud 上的一个可靠设置。价格因机器类型和地区而异；选择适合你工作负载的最小虚拟机，如果遇到内存不足（OOM）再升级。

## 我们要做什么（简单来说）？

-   创建一个 GCP 项目并启用计费
-   创建一个 Compute Engine 虚拟机
-   安装 Docker（隔离的应用运行时）
-   在 Docker 中启动 OpenClaw Gateway
-   在宿主机上持久化 `~/.openclaw` + `~/.openclaw/workspace`（在重启/重建后保留）
-   通过 SSH 隧道从你的笔记本电脑访问控制界面

可以通过以下方式访问 Gateway：

-   从你的笔记本电脑进行 SSH 端口转发
-   如果你自行管理防火墙和令牌，也可以直接暴露端口

本指南使用 GCP Compute Engine 上的 Debian。Ubuntu 也适用；请相应映射软件包。关于通用的 Docker 流程，请参阅 [Docker](./docker.md)。

* * *

## 快速路径（有经验的操作者）

1.  创建 GCP 项目 + 启用 Compute Engine API
2.  创建 Compute Engine 虚拟机（e2-small, Debian 12, 20GB）
3.  SSH 连接到虚拟机
4.  安装 Docker
5.  克隆 OpenClaw 仓库
6.  创建持久化的宿主机目录
7.  配置 `.env` 和 `docker-compose.yml`
8.  预置所需的二进制文件，构建并启动

* * *

## 所需条件

-   GCP 账户（e2-micro 符合免费套餐资格）
-   已安装 gcloud CLI（或使用 Cloud Console）
-   从你的笔记本电脑进行 SSH 访问
-   基本熟悉 SSH + 复制/粘贴
-   ~20-30 分钟
-   Docker 和 Docker Compose
-   模型认证凭据
-   可选的提供商凭据
    -   WhatsApp QR 码
    -   Telegram 机器人令牌
    -   Gmail OAuth

* * *

## 1) 安装 gcloud CLI（或使用 Console）

**选项 A: gcloud CLI**（推荐用于自动化） 从 [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) 安装 初始化和认证：

```bash
gcloud init
gcloud auth login
```

**选项 B: Cloud Console** 所有步骤都可以通过 [https://console.cloud.google.com](https://console.cloud.google.com) 的 Web 界面完成。

* * *

## 2) 创建一个 GCP 项目

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

在 [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) 启用计费（Compute Engine 必需）。启用 Compute Engine API：

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1.  转到 IAM & Admin > 创建项目
2.  命名并创建
3.  为项目启用计费
4.  导航到 APIs & Services > 启用 API > 搜索 “Compute Engine API” > 启用

* * *

## 3) 创建虚拟机

**机器类型：**

| 类型 | 规格 | 成本 | 备注 |
| --- | --- | --- | --- |
| e2-medium | 2 vCPU, 4GB RAM | ~$25/月 | 本地 Docker 构建最可靠 |
| e2-small | 2 vCPU, 2GB RAM | ~$12/月 | Docker 构建的最低推荐配置 |
| e2-micro | 2 vCPU (共享), 1GB RAM | 免费套餐资格 | 常因 Docker 构建 OOM 失败（退出码 137） |

**CLI:**

```
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1.  转到 Compute Engine > VM 实例 > 创建实例
2.  名称：`openclaw-gateway`
3.  区域：`us-central1`，可用区：`us-central1-a`
4.  机器类型：`e2-small`
5.  启动磁盘：Debian 12, 20GB
6.  创建

* * *

## 4) SSH 连接到虚拟机

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:** 在 Compute Engine 仪表板中点击你的虚拟机旁边的“SSH”按钮。注意：SSH 密钥传播可能在虚拟机创建后需要 1-2 分钟。如果连接被拒绝，请等待并重试。

* * *

## 5) 安装 Docker（在虚拟机上）

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

注销并重新登录以使组更改生效：

```
exit
```

然后重新 SSH 连接：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

验证：

```bash
docker --version
docker compose version
```

* * *

## 6) 克隆 OpenClaw 仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

本指南假设你将构建一个自定义镜像以保证二进制文件的持久性。

* * *

## 7) 创建持久化的宿主机目录

Docker 容器是临时的。所有长期存在的状态必须保存在宿主机上。

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

* * *

## 8) 配置环境变量

在仓库根目录创建 `.env`。

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

生成强密钥：

```bash
openssl rand -hex 32
```

**不要提交此文件。**

* * *

## 9) Docker Compose 配置

创建或更新 `docker-compose.yml`。

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # 推荐：在虚拟机上保持 Gateway 仅限回环访问；通过 SSH 隧道访问。
      # 要公开暴露，请移除 `127.0.0.1:` 前缀并相应配置防火墙。
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

* * *

## 10) 将所需二进制文件预置到镜像中（关键）

在运行的容器内安装二进制文件是一个陷阱。任何在运行时安装的内容都将在重启时丢失。技能所需的所有外部二进制文件必须在镜像构建时安装。下面的示例仅展示了三个常见的二进制文件：

-   `gog` 用于 Gmail 访问
-   `goplaces` 用于 Google Places
-   `wacli` 用于 WhatsApp

这些是示例，并非完整列表。你可以使用相同的模式安装任意数量的二进制文件。如果你以后添加了依赖额外二进制文件的新技能，你必须：

1.  更新 Dockerfile
2.  重新构建镜像
3.  重启容器

**示例 Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 示例二进制文件 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 示例二进制文件 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 示例二进制文件 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 使用相同模式在下方添加更多二进制文件

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 11) 构建并启动

```bash
docker compose build
docker compose up -d openclaw-gateway
```

如果在 `pnpm install --frozen-lockfile` 期间构建失败并出现 `Killed` / `exit code 137`，说明虚拟机内存不足。至少使用 `e2-small`，或使用 `e2-medium` 以获得更可靠的首次构建。当绑定到局域网（`OPENCLAW_GATEWAY_BIND=lan`）时，在继续之前配置一个受信任的浏览器来源：

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

如果你更改了网关端口，请将 `18789` 替换为你配置的端口。验证二进制文件：

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

预期输出：

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 12) 验证 Gateway

```bash
docker compose logs -f openclaw-gateway
```

成功：

```ini
[gateway] listening on ws://0.0.0.0:18789
```

* * *

## 13) 从你的笔记本电脑访问

创建一个 SSH 隧道来转发 Gateway 端口：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

在浏览器中打开：`http://127.0.0.1:18789/` 获取一个新的带令牌的仪表板链接：

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

粘贴该 URL 中的令牌。如果控制界面显示 `unauthorized` 或 `disconnected (1008): pairing required`，请批准浏览器设备：

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

* * *

## 哪些内容持久化在哪里（事实来源）

OpenClaw 在 Docker 中运行，但 Docker 不是事实来源。所有长期存在的状态必须在重启、重建和重新启动后保留。

| 组件 | 位置 | 持久化机制 | 备注 |
| --- | --- | --- | --- |
| Gateway 配置 | `/home/node/.openclaw/` | 宿主机卷挂载 | 包括 `openclaw.json`、令牌 |
| 模型认证配置文件 | `/home/node/.openclaw/` | 宿主机卷挂载 | OAuth 令牌、API 密钥 |
| 技能配置 | `/home/node/.openclaw/skills/` | 宿主机卷挂载 | 技能级别的状态 |
| 代理工作区 | `/home/node/.openclaw/workspace/` | 宿主机卷挂载 | 代码和代理产物 |
| WhatsApp 会话 | `/home/node/.openclaw/` | 宿主机卷挂载 | 保留 QR 码登录 |
| Gmail 密钥环 | `/home/node/.openclaw/` | 宿主机卷 + 密码 | 需要 `GOG_KEYRING_PASSWORD` |
| 外部二进制文件 | `/usr/local/bin/` | Docker 镜像 | 必须在构建时预置 |
| Node 运行时 | 容器文件系统 | Docker 镜像 | 每次镜像构建时重建 |
| 操作系统包 | 容器文件系统 | Docker 镜像 | 不要在运行时安装 |
| Docker 容器 | 临时性 | 可重启 | 销毁是安全的 |

* * *

## 更新

要在虚拟机上更新 OpenClaw：

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

* * *

## 故障排除

**SSH 连接被拒绝** SSH 密钥传播可能在虚拟机创建后需要 1-2 分钟。请等待并重试。**OS Login 问题** 检查你的 OS Login 配置文件：

```bash
gcloud compute os-login describe-profile
```

确保你的账户具有所需的 IAM 权限（Compute OS Login 或 Compute OS Admin Login）。**内存不足（OOM）** 如果 Docker 构建失败并出现 `Killed` 和 `exit code 137`，说明虚拟机被 OOM 杀死。升级到 e2-small（最低要求）或 e2-medium（推荐用于可靠的本地构建）：

```bash
# 先停止虚拟机
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# 更改机器类型
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# 启动虚拟机
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## 服务账户（安全最佳实践）

对于个人使用，你的默认用户账户即可。对于自动化或 CI/CD 流水线，创建一个具有最小权限的专用服务账户：

1.  创建服务账户：
    
    Copy
    
    ```
    gcloud iam service-accounts create openclaw-deploy \
      --display-name="OpenClaw Deployment"
    ```
    
2.  授予 Compute Instance Admin 角色（或更窄的自定义角色）：
    
    Copy
    
    ```
    gcloud projects add-iam-policy-binding my-openclaw-project \
      --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
      --role="roles/compute.instanceAdmin.v1"
    ```
    

避免对自动化使用 Owner 角色。遵循最小权限原则。有关 IAM 角色的详细信息，请参阅 [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles)。

* * *

## 后续步骤

-   设置消息通道：[通道](../channels.md)
-   将本地设备配对为节点：[节点](../nodes.md)
-   配置 Gateway：[Gateway 配置](../gateway/configuration.md)

[Hetzner](./hetzner.md)[macOS 虚拟机](./macos-vm.md)