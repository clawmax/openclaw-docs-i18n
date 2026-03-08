

  CLI 命令

  
# agents

管理隔离代理（工作空间 + 认证 + 路由）。相关概念：

-   多代理路由：[多代理路由](../concepts/multi-agent.md)
-   代理工作空间：[代理工作空间](../concepts/agent-workspace.md)

## 示例

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## 路由绑定

使用路由绑定将入站频道流量固定到特定代理。列出绑定：

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

添加绑定：

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

如果省略 `accountId` (`--bind `)，OpenClaw 会在可用时从频道默认值和插件设置钩子中解析它。

### 绑定范围行为

-   没有 `accountId` 的绑定仅匹配频道的默认账户。
-   `accountId: "*"` 是频道范围的回退（所有账户），其优先级低于显式的账户绑定。
-   如果同一代理已存在一个没有 `accountId` 的匹配频道绑定，而你之后又用显式或解析出的 `accountId` 进行绑定，OpenClaw 会就地升级该现有绑定，而不是添加重复项。

示例：

```bash
# 初始的仅频道绑定
openclaw agents bind --agent work --bind telegram

# 稍后升级为账户范围的绑定
openclaw agents bind --agent work --bind telegram:ops
```

升级后，该绑定的路由范围限定为 `telegram:ops`。如果你还需要默认账户的路由，请显式添加它（例如 `--bind telegram:default`）。移除绑定：

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## 身份文件

每个代理工作空间可以在其根目录包含一个 `IDENTITY.md` 文件：

-   示例路径：`~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` 从工作空间根目录（或显式的 `--identity-file`）读取

头像路径相对于工作空间根目录解析。

## 设置身份

`set-identity` 将字段写入 `agents.list[].identity`：

-   `name`
-   `theme`
-   `emoji`
-   `avatar`（相对于工作空间的路径、http(s) URL 或 data URI）

从 `IDENTITY.md` 加载：

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

显式覆盖字段：

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

配置示例：

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)

---