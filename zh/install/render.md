

  托管与部署

  
# 在 Render 上部署

使用基础设施即代码在 Render 上部署 OpenClaw。包含的 `render.yaml` 蓝图以声明方式定义您的整个技术栈、服务、磁盘、环境变量，因此您可以一键部署，并将您的基础设施与代码一起进行版本控制。

## 先决条件

-   一个 [Render 账户](https://render.com)（提供免费套餐）
-   来自您首选的 [模型提供商](../providers.md) 的 API 密钥

## 使用 Render 蓝图部署

[部署到 Render](https://render.com/deploy?repo=https://github.com/openclaw/openclaw) 点击此链接将：

1.  根据此仓库根目录下的 `render.yaml` 蓝图创建一个新的 Render 服务。
2.  提示您设置 `SETUP_PASSWORD`
3.  构建 Docker 镜像并部署

部署完成后，您的服务 URL 遵循模式 `https://<service-name>.onrender.com`。

## 理解蓝图

Render 蓝图是定义您基础设施的 YAML 文件。此仓库中的 `render.yaml` 配置了运行 OpenClaw 所需的一切：

```yaml
services:
  - type: web
    name: openclaw
    runtime: docker
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: "8080"
      - key: SETUP_PASSWORD
        sync: false # prompts during deploy
      - key: OPENCLAW_STATE_DIR
        value: /data/.openclaw
      - key: OPENCLAW_WORKSPACE_DIR
        value: /data/workspace
      - key: OPENCLAW_GATEWAY_TOKEN
        generateValue: true # auto-generates a secure token
    disk:
      name: openclaw-data
      mountPath: /data
      sizeGB: 1
```

使用的关键蓝图特性：

| 特性 | 用途 |
| --- | --- |
| `runtime: docker` | 从仓库的 Dockerfile 构建 |
| `healthCheckPath` | Render 监控 `/health` 并重启不健康的实例 |
| `sync: false` | 在部署期间提示输入值（用于密钥） |
| `generateValue: true` | 自动生成加密安全的值 |
| `disk` | 持久化存储，可在重新部署后保留 |

## 选择套餐

| 套餐 | 休眠 | 磁盘 | 最适合 |
| --- | --- | --- | --- |
| 免费 | 空闲 15 分钟后 | 不可用 | 测试、演示 |
| 入门版 | 从不 | 1GB+ | 个人使用、小团队 |
| 标准版+ | 从不 | 1GB+ | 生产环境、多频道 |

蓝图默认为 `starter`。要使用免费套餐，请在您 fork 的 `render.yaml` 中将 `plan` 改为 `free`（但请注意：没有持久化磁盘意味着每次部署时配置都会重置）。

## 部署后

### 完成设置向导

1.  访问 `https://<your-service>.onrender.com/setup`
2.  输入您的 `SETUP_PASSWORD`
3.  选择一个模型提供商并粘贴您的 API 密钥
4.  可选配置消息频道（Telegram、Discord、Slack）
5.  点击 **运行设置**

### 访问控制界面

Web 仪表板位于 `https://<your-service>.onrender.com/openclaw`。

## Render 仪表板功能

### 日志

在 **仪表板 → 您的服务 → 日志** 中查看实时日志。可按以下类型筛选：

-   构建日志（Docker 镜像创建）
-   部署日志（服务启动）
-   运行时日志（应用程序输出）

### Shell 访问

用于调试，通过 **仪表板 → 您的服务 → Shell** 打开一个 shell 会话。持久化磁盘挂载在 `/data`。

### 环境变量

在 **仪表板 → 您的服务 → 环境** 中修改变量。更改会触发自动重新部署。

### 自动部署

如果您使用原始的 OpenClaw 仓库，Render 将不会自动部署您的 OpenClaw。要更新它，请从仪表板运行手动蓝图同步。

## 自定义域名

1.  前往 **仪表板 → 您的服务 → 设置 → 自定义域名**
2.  添加您的域名
3.  按照指示配置 DNS（CNAME 指向 `*.onrender.com`）
4.  Render 会自动配置 TLS 证书

## 扩展

Render 支持水平和垂直扩展：

-   **垂直**：更改套餐以获得更多 CPU/内存
-   **水平**：增加实例数量（标准版及以上套餐）

对于 OpenClaw，垂直扩展通常就足够了。水平扩展需要粘性会话或外部状态管理。

## 备份与迁移

随时导出您的配置和工作区：

```
https://<your-service>.onrender.com/setup/export
```

这将下载一个可移植的备份，您可以在任何 OpenClaw 主机上恢复。

## 故障排除

### 服务无法启动

检查 Render 仪表板中的部署日志。常见问题：

-   缺少 `SETUP_PASSWORD` — 蓝图会提示输入，但请确认它已设置
-   端口不匹配 — 确保 `PORT=8080` 与 Dockerfile 中暴露的端口一致

### 冷启动缓慢（免费套餐）

免费套餐服务在空闲 15 分钟后会休眠。休眠后的第一个请求需要几秒钟来启动容器。升级到入门版套餐以获得始终在线。

### 重新部署后数据丢失

这发生在免费套餐（无持久化磁盘）。升级到付费套餐，或定期通过 `/setup/export` 导出您的配置。

### 健康检查失败

Render 期望在 30 秒内从 `/health` 收到 200 响应。如果构建成功但部署失败，可能是服务启动时间过长。检查：

-   构建日志中的错误
-   容器是否能在本地通过 `docker build && docker run` 运行

[在 Railway 上部署](./railway.md)[在 Northflank 上部署](./northflank.md)