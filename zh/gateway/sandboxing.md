

  安全与沙盒化

  
# 沙盒化

OpenClaw 可以在 **Docker 容器内运行工具** 以减少爆炸半径。这是**可选的**，由配置控制 (`agents.defaults.sandbox` 或 `agents.list[].sandbox`)。如果沙盒化关闭，工具将在主机上运行。网关保持在主机上；启用沙盒化时，工具执行在隔离的沙盒中运行。这不是一个完美的安全边界，但当模型执行某些愚蠢操作时，它能实质性地限制文件系统和进程访问。

## 哪些内容会被沙盒化

-   工具执行 (`exec`, `read`, `write`, `edit`, `apply_patch`, `process` 等)。
-   可选的沙盒化浏览器 (`agents.defaults.sandbox.browser`)。
    -   默认情况下，当浏览器工具需要时，沙盒浏览器会自动启动（确保 CDP 可达）。通过 `agents.defaults.sandbox.browser.autoStart` 和 `agents.defaults.sandbox.browser.autoStartTimeoutMs` 进行配置。
    -   默认情况下，沙盒浏览器容器使用专用的 Docker 网络 (`openclaw-sandbox-browser`) 而不是全局的 `bridge` 网络。使用 `agents.defaults.sandbox.browser.network` 进行配置。
    -   可选的 `agents.defaults.sandbox.browser.cdpSourceRange` 使用 CIDR 允许列表限制容器边缘的 CDP 入口（例如 `172.21.0.1/32`）。
    -   noVNC 观察者访问默认受密码保护；OpenClaw 会生成一个短期有效的令牌 URL，该 URL 提供一个本地引导页面，并在 URL 片段（而非查询/头日志）中打开带有密码的 noVNC。
    -   `agents.defaults.sandbox.browser.allowHostControl` 允许沙盒会话显式地以主机浏览器为目标。
    -   可选的允许列表控制 `target: "custom"`：`allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`。

不被沙盒化的内容：

-   网关进程本身。
-   任何明确允许在主机上运行的工具（例如 `tools.elevated`）。
    -   **提升的 exec 在主机上运行并绕过沙盒化。**
    -   如果沙盒化关闭，`tools.elevated` 不会改变执行方式（已在主机上）。参见[提升模式](../tools/elevated.md)。

## 模式

`agents.defaults.sandbox.mode` 控制**何时**使用沙盒化：

-   `"off"`：无沙盒化。
-   `"non-main"`：仅沙盒化**非主**会话（如果您希望正常聊天在主机上运行，这是默认设置）。
-   `"all"`：每个会话都在沙盒中运行。注意：`"non-main"` 基于 `session.mainKey`（默认为 `"main"`），而非代理 ID。群组/频道会话使用自己的密钥，因此它们被视为非主会话并将被沙盒化。

## 范围

`agents.defaults.sandbox.scope` 控制**创建多少个容器**：

-   `"session"`（默认）：每个会话一个容器。
-   `"agent"`：每个代理一个容器。
-   `"shared"`：一个容器由所有沙盒会话共享。

## 工作区访问

`agents.defaults.sandbox.workspaceAccess` 控制**沙盒可以看到什么**：

-   `"none"`（默认）：工具看到 `~/.openclaw/sandboxes` 下的沙盒工作区。
-   `"ro"`：以只读方式将代理工作区挂载到 `/agent`（禁用 `write`/`edit`/`apply_patch`）。
-   `"rw"`：以读写方式将代理工作区挂载到 `/workspace`。

传入的媒体文件被复制到活动的沙盒工作区 (`media/inbound/*`)。技能说明：`read` 工具是基于沙盒根目录的。当 `workspaceAccess: "none"` 时，OpenClaw 会将符合条件的技能镜像到沙盒工作区 (`.../skills`) 以便读取。当 `"rw"` 时，工作区技能可以从 `/workspace/skills` 读取。

## 自定义绑定挂载

`agents.defaults.sandbox.docker.binds` 将额外的主机目录挂载到容器中。格式：`host:container:mode`（例如，`"/home/user/source:/source:rw"`）。全局和每个代理的绑定会被**合并**（而非替换）。在 `scope: "shared"` 下，每个代理的绑定会被忽略。`agents.defaults.sandbox.browser.binds` 仅将额外的主机目录挂载到**沙盒浏览器**容器中。

-   当设置时（包括 `[]`），它会替换浏览器容器的 `agents.defaults.sandbox.docker.binds`。
-   当省略时，浏览器容器回退到 `agents.defaults.sandbox.docker.binds`（向后兼容）。

示例（只读源代码 + 额外数据目录）：

```json
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

安全说明：

-   绑定绕过沙盒文件系统：它们以您设置的任何模式（`:ro` 或 `:rw`）暴露主机路径。
-   OpenClaw 会阻止危险的绑定源（例如：`docker.sock`, `/etc`, `/proc`, `/sys`, `/dev` 以及会暴露它们的父挂载）。
-   敏感挂载（密钥、SSH 密钥、服务凭证）应设为 `:ro`，除非绝对必要。
-   如果您只需要对工作区的读取访问权限，请与 `workspaceAccess: "ro"` 结合使用；绑定模式保持独立。
-   有关绑定如何与工具策略和提升的 exec 交互，请参见[沙盒 vs 工具策略 vs 提升](./sandbox-vs-tool-policy-vs-elevated.md)。

## 镜像 + 设置

默认镜像：`openclaw-sandbox:bookworm-slim` 构建一次：

```
scripts/sandbox-setup.sh
```

注意：默认镜像**不**包含 Node。如果技能需要 Node（或其他运行时），要么构建自定义镜像，要么通过 `sandbox.docker.setupCommand` 安装（需要网络出口 + 可写的根目录 + root 用户）。如果您想要一个功能更全的沙盒镜像，包含常用工具（例如 `curl`, `jq`, `nodejs`, `python3`, `git`），请构建：

```
scripts/sandbox-common-setup.sh
```

然后将 `agents.defaults.sandbox.docker.image` 设置为 `openclaw-sandbox-common:bookworm-slim`。沙盒浏览器镜像：

```
scripts/sandbox-browser-setup.sh
```

默认情况下，沙盒容器在**无网络**的情况下运行。使用 `agents.defaults.sandbox.docker.network` 覆盖。捆绑的沙盒浏览器镜像还为容器化工作负载应用了保守的 Chromium 启动默认值。当前的容器默认值包括：

-   `--remote-debugging-address=127.0.0.1`
-   `--remote-debugging-port=`
-   `--user-data-dir=${HOME}/.chrome`
-   `--no-first-run`
-   `--no-default-browser-check`
-   `--disable-3d-apis`
-   `--disable-gpu`
-   `--disable-dev-shm-usage`
-   `--disable-background-networking`
-   `--disable-extensions`
-   `--disable-features=TranslateUI`
-   `--disable-breakpad`
-   `--disable-crash-reporter`
-   `--disable-software-rasterizer`
-   `--no-zygote`
-   `--metrics-recording-only`
-   `--renderer-process-limit=2`
-   `--no-sandbox` 和 `--disable-setuid-sandbox` 当 `noSandbox` 启用时。
-   三个图形硬化标志 (`--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu`) 是可选的，在容器缺乏 GPU 支持时很有用。如果您的工作负载需要 WebGL 或其他 3D/浏览器功能，请设置 `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0`。
-   `--disable-extensions` 默认启用，对于依赖扩展的流程，可以通过 `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` 禁用。
-   `--renderer-process-limit=2` 由 `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` 控制，其中 `0` 保持 Chromium 的默认值。

如果您需要不同的运行时配置文件，请使用自定义浏览器镜像并提供您自己的入口点。对于本地（非容器）Chromium 配置文件，使用 `browser.extraArgs` 来附加额外的启动标志。安全默认值：

-   `network: "host"` 被阻止。
-   `network: "container:"` 默认被阻止（命名空间加入绕过风险）。
-   紧急覆盖：`agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`。

Docker 安装和容器化网关位于此处：[Docker](../install/docker.md) 对于 Docker 网关部署，`docker-setup.sh` 可以引导沙盒配置。设置 `OPENCLAW_SANDBOX=1`（或 `true`/`yes`/`on`）以启用该路径。您可以使用 `OPENCLAW_DOCKER_SOCKET` 覆盖套接字位置。完整设置和环境参考：[Docker](../install/docker.md#enable-agent-sandbox-for-docker-gateway-opt-in)。

## setupCommand（一次性容器设置）

`setupCommand` 在沙盒容器创建后**运行一次**（不是每次运行）。它通过 `sh -lc` 在容器内部执行。路径：

-   全局：`agents.defaults.sandbox.docker.setupCommand`
-   每个代理：`agents.list[].sandbox.docker.setupCommand`

常见问题：

-   默认 `docker.network` 是 `"none"`（无出口），因此软件包安装会失败。
-   `docker.network: "container:"` 需要 `dangerouslyAllowContainerNamespaceJoin: true` 并且仅用于紧急情况。
-   `readOnlyRoot: true` 阻止写入；设置 `readOnlyRoot: false` 或构建自定义镜像。
-   软件包安装的 `user` 必须是 root（省略 `user` 或设置 `user: "0:0"`）。
-   沙盒 exec **不**继承主机的 `process.env`。对于技能 API 密钥，请使用 `agents.defaults.sandbox.docker.env`（或自定义镜像）。

## 工具策略 + 逃生通道

工具的允许/拒绝策略在沙盒规则之前仍然适用。如果一个工具被全局或每个代理拒绝，沙盒化不会恢复它。`tools.elevated` 是一个明确的逃生通道，它在主机上运行 `exec`。`/exec` 指令仅适用于授权的发送者，并且每个会话持久存在；要硬性禁用 `exec`，请使用工具策略拒绝（参见[沙盒 vs 工具策略 vs 提升](./sandbox-vs-tool-policy-vs-elevated.md)）。调试：

-   使用 `openclaw sandbox explain` 来检查有效的沙盒模式、工具策略和修复配置键。
-   有关“为什么这个被阻止？”的思维模型，请参见[沙盒 vs 工具策略 vs 提升](./sandbox-vs-tool-policy-vs-elevated.md)。保持锁定状态。

## 多代理覆盖

每个代理都可以覆盖沙盒和工具：`agents.list[].sandbox` 和 `agents.list[].tools`（以及 `agents.list[].tools.sandbox.tools` 用于沙盒工具策略）。有关优先级，请参见[多代理沙盒与工具](../tools/multi-agent-sandbox-tools.md)。

## 最小启用示例

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## 相关文档

-   [沙盒配置](./configuration.md#agentsdefaults-sandbox)
-   [多代理沙盒与工具](../tools/multi-agent-sandbox-tools.md)
-   [安全](./security.md)

[安全](./security.md)[沙盒 vs 工具策略 vs 提升](./sandbox-vs-tool-policy-vs-elevated.md)