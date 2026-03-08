

  macOS 伴侣应用

  
# 技能

macOS 应用通过网关呈现 OpenClaw 技能；它不在本地解析技能。

## 数据源

-   `skills.status`（网关）返回所有技能及其资格状态和缺失要求（包括捆绑技能的允许列表阻止）。
-   要求源自每个 `SKILL.md` 中的 `metadata.openclaw.requires`。

## 安装操作

-   `metadata.openclaw.install` 定义安装选项（brew/node/go/uv）。
-   应用调用 `skills.install` 在网关主机上运行安装程序。
-   当提供多个安装程序时，网关仅呈现一个首选安装程序（如果可用则使用 brew，否则使用 `skills.install` 中的节点管理器，默认为 npm）。

## 环境变量/API 密钥

-   应用将密钥存储在 `~/.openclaw/openclaw.json` 中的 `skills.entries.` 下。
-   `skills.update` 用于修补 `enabled`、`apiKey` 和 `env`。

## 远程模式

-   安装和配置更新发生在网关主机上（而非本地 Mac）。

[macOS IPC](./xpc.md)[Peekaboo 桥接](./peekaboo.md)

---