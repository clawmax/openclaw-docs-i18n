

  配置与操作

  
# 多网关

大多数设置应使用一个网关，因为单个网关可以处理多个消息连接和代理。如果您需要更强的隔离性或冗余性（例如，救援机器人），请使用隔离的配置文件/端口运行单独的网关。

## 隔离检查清单（必需）

-   `OPENCLAW_CONFIG_PATH` — 每个实例的配置文件
-   `OPENCLAW_STATE_DIR` — 每个实例的会话、凭据、缓存
-   `agents.defaults.workspace` — 每个实例的工作区根目录
-   `gateway.port`（或 `--port`）— 每个实例唯一
-   派生端口（浏览器/画布）不得重叠

如果这些是共享的，您将遇到配置竞争和端口冲突。

## 推荐：配置文件 (--profile)

配置文件自动限定 `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` 的范围，并为服务名称添加后缀。

```bash
# 主机器人
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# 救援机器人
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

每个配置文件的服务：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## 救援机器人指南

在同一主机上运行第二个网关，它拥有自己的：

-   配置文件/配置
-   状态目录
-   工作区
-   基础端口（加上派生端口）

这使救援机器人与主机器人隔离，以便在主机器人宕机时，救援机器人可以进行调试或应用配置更改。端口间隔：基础端口之间至少留出 20 个端口，以确保派生的浏览器/画布/CDP 端口永远不会冲突。

### 如何安装（救援机器人）

```bash
# 主机器人（现有的或全新的，不带 --profile 参数）
# 运行在端口 18789 + Chrome CDC/Canvas/... 端口
openclaw onboard
openclaw gateway install

# 救援机器人（隔离的配置文件 + 端口）
openclaw --profile rescue onboard
# 注意：
# - 工作区名称默认会加上 -rescue 后缀
# - 端口应至少为 18789 + 20 个端口，
#   最好选择完全不同的基础端口，例如 19789，
# - 其余的上线步骤与正常情况相同

# 安装服务（如果在上线期间未自动发生）
openclaw --profile rescue gateway install
```

## 端口映射（派生）

基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）。

-   浏览器控制服务端口 = 基础端口 + 2（仅限环回）
-   画布主机在网关 HTTP 服务器上提供服务（与 `gateway.port` 端口相同）
-   浏览器配置文件 CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配

如果您在配置或环境变量中覆盖了这些端口中的任何一个，则必须确保每个实例的端口是唯一的。

## 浏览器/CDP 注意事项（常见陷阱）

-   **不要**在多个实例中将 `browser.cdpUrl` 固定为相同的值。
-   每个实例都需要自己的浏览器控制端口和 CDP 范围（从其网关端口派生）。
-   如果您需要明确的 CDP 端口，请为每个实例设置 `browser.profiles..cdpPort`。
-   远程 Chrome：使用 `browser.profiles..cdpUrl`（每个配置文件，每个实例）。

## 手动环境变量示例

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## 快速检查

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

[后台执行与进程工具](./background-process.md)[故障排除](./troubleshooting.md)