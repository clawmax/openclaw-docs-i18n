

  技能

  
# 技能配置

所有与技能相关的配置都位于 `~/.openclaw/openclaw.json` 文件的 `skills` 部分下。

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway 运行时仍为 Node；不推荐 bun)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // 或纯文本字符串
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## 字段说明

-   `allowBundled`: 可选，仅针对**捆绑**技能的允许列表。设置后，只有列表中的捆绑技能才可用（不影响托管/工作区技能）。
-   `load.extraDirs`: 要扫描的额外技能目录（优先级最低）。
-   `load.watch`: 监视技能文件夹并刷新技能快照（默认：true）。
-   `load.watchDebounceMs`: 技能监视器事件的防抖时间（毫秒）（默认：250）。
-   `install.preferBrew`: 在可用时优先使用 brew 安装器（默认：true）。
-   `install.nodeManager`: Node 安装器偏好（`npm` | `pnpm` | `yarn` | `bun`，默认：npm）。这仅影响**技能安装**；Gateway 运行时仍应为 Node（不推荐将 Bun 用于 WhatsApp/Telegram）。
-   `entries.`: 每项技能的覆盖设置。

每项技能的字段：

-   `enabled`: 设置为 `false` 以禁用某项技能，即使它是捆绑/已安装的。
-   `env`: 为智能体运行注入的环境变量（仅在尚未设置时生效）。
-   `apiKey`: 可选，为声明了主要环境变量的技能提供便利。支持纯文本字符串或 SecretRef 对象（`{ source, provider, id }`）。

## 注意事项

-   `entries` 下的键默认映射到技能名称。如果技能定义了 `metadata.openclaw.skillKey`，则使用该键。
-   当监视器启用时，对技能的更改会在智能体的下一个回合被拾取。

### 沙盒化技能 + 环境变量

当会话处于**沙盒化**状态时，技能进程在 Docker 内运行。沙盒**不会**继承主机的 `process.env`。请使用以下方法之一：

-   `agents.defaults.sandbox.docker.env`（或每个智能体的 `agents.list[].sandbox.docker.env`）
-   将环境变量烘焙到您的自定义沙盒镜像中

全局的 `env` 和 `skills.entries..env/apiKey` 仅适用于**主机**运行。

[技能](./skills.md)[ClawHub](./clawhub.md)

---