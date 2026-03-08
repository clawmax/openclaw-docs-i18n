

  CLI 命令

  
# acp

运行与 OpenClaw 网关通信的[代理客户端协议 (ACP)](https://agentclientprotocol.com/) 桥接器。此命令通过 stdio 与 IDE 进行 ACP 通信，并通过 WebSocket 将提示转发到网关。它保持 ACP 会话与网关会话密钥的映射关系。

## 使用方法

```bash
openclaw acp

# 远程网关
openclaw acp --url wss://gateway-host:18789 --token <token>

# 远程网关（从文件读取令牌）
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 附加到现有的会话密钥
openclaw acp --session agent:main:main

# 通过标签附加（必须已存在）
openclaw acp --session-label "support inbox"

# 在第一个提示之前重置会话密钥
openclaw acp --session agent:main:main --reset-session
```

## ACP 客户端（调试）

使用内置的 ACP 客户端来检查桥接器是否正常工作，而无需 IDE。它会启动 ACP 桥接器并允许您交互式地输入提示。

```bash
openclaw acp client

# 将启动的桥接器指向远程网关
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 覆盖服务器命令（默认：openclaw）
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

权限模型（客户端调试模式）：

-   自动批准基于允许列表，仅适用于受信任的核心工具 ID。
-   `read` 自动批准的范围限定在当前工作目录（设置 `--cwd` 时）。
-   未知/非核心工具名称、超出范围的读取以及危险工具始终需要明确的提示批准。
-   服务器提供的 `toolCall.kind` 被视为不受信任的元数据（不是授权来源）。

## 如何使用

当 IDE（或其他客户端）使用代理客户端协议，并且您希望它驱动 OpenClaw 网关会话时，请使用 ACP。

1.  确保网关正在运行（本地或远程）。
2.  配置网关目标（通过配置或标志）。
3.  将您的 IDE 配置为通过 stdio 运行 `openclaw acp`。

配置示例（持久化）：

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

直接运行示例（不写入配置）：

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# 为了本地进程安全，推荐使用
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## 选择代理

ACP 不直接选择代理。它通过网关会话密钥进行路由。使用代理作用域的会话密钥来定位特定代理：

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

每个 ACP 会话映射到一个网关会话密钥。一个代理可以有许多会话；除非您覆盖密钥或标签，否则 ACP 默认为一个隔离的 `acp:` 会话。

## Zed 编辑器设置

在 `~/.config/zed/settings.json` 中添加自定义 ACP 代理（或使用 Zed 的设置 UI）：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

要定位特定的网关或代理：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

在 Zed 中，打开 Agent 面板并选择 “OpenClaw ACP” 以启动一个线程。

## 会话映射

默认情况下，ACP 会话会获得一个带有 `acp:` 前缀的隔离网关会话密钥。要重用已知会话，请传递会话密钥或标签：

-   `--session `: 使用特定的网关会话密钥。
-   `--session-label `: 通过标签解析现有会话。
-   `--reset-session`: 为该密钥创建一个新的会话 ID（相同密钥，新记录）。

如果您的 ACP 客户端支持元数据，您可以按会话覆盖：

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

在 [/concepts/session](../concepts/session.md) 了解更多关于会话密钥的信息。

## 选项

-   `--url `: 网关 WebSocket URL（配置后默认为 gateway.remote.url）。
-   `--token `: 网关认证令牌。
-   `--token-file `: 从文件读取网关认证令牌。
-   `--password `: 网关认证密码。
-   `--password-file `: 从文件读取网关认证密码。
-   `--session `: 默认会话密钥。
-   `--session-label `: 要解析的默认会话标签。
-   `--require-existing`: 如果会话密钥/标签不存在则失败。
-   `--reset-session`: 在首次使用前重置会话密钥。
-   `--no-prefix-cwd`: 不在提示前添加工作目录前缀。
-   `--verbose, -v`: 向 stderr 输出详细日志。

安全说明：

-   `--token` 和 `--password` 在某些系统的本地进程列表中可能可见。
-   推荐使用 `--token-file`/`--password-file` 或环境变量（`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`）。
-   ACP 运行时后端子进程会收到 `OPENCLAW_SHELL=acp`，可用于特定上下文的 shell/profile 规则。
-   `openclaw acp client` 在启动的桥接器进程上设置 `OPENCLAW_SHELL=acp-client`。

### acp client 选项

-   `--cwd `: ACP 会话的工作目录。
-   `--server `: ACP 服务器命令（默认：`openclaw`）。
-   `--server-args <args...>`: 传递给 ACP 服务器的额外参数。
-   `--server-verbose`: 在 ACP 服务器上启用详细日志。
-   `--verbose, -v`: 详细的客户端日志。

[CLI 参考](../cli.md)[agent](./agent.md)