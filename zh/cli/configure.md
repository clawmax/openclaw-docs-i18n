

  CLI 命令

  
# configure

用于设置凭证、设备和代理默认值的交互式提示。注意：**模型**部分现在包含一个用于 `agents.defaults.models` 允许列表（显示在 `/model` 和模型选择器中的内容）的多选功能。提示：不带子命令的 `openclaw config` 会打开相同的向导。使用 `openclaw config get|set|unset` 进行非交互式编辑。相关：

-   网关配置参考：[配置](../gateway/configuration.md)
-   配置 CLI：[配置](./config.md)

注意事项：

-   选择网关运行位置始终会更新 `gateway.mode`。如果这是您唯一需要的设置，您可以选择“继续”而不配置其他部分。
-   面向频道的服务（Slack/Discord/Matrix/Microsoft Teams）在设置过程中会提示输入频道/房间允许列表。您可以输入名称或 ID；向导会在可能时将名称解析为 ID。
-   如果您运行守护程序安装步骤，令牌认证需要一个令牌，并且 `gateway.auth.token` 由 SecretRef 管理，configure 会验证 SecretRef，但不会将解析出的明文令牌值持久化到监督服务环境元数据中。
-   如果令牌认证需要一个令牌，但配置的令牌 SecretRef 未解析，configure 将阻止守护程序安装，并提供可操作的修复指导。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，configure 将阻止守护程序安装，直到明确设置模式。

## 示例

```bash
openclaw configure
openclaw configure --section model --section channels
```

[config](./config.md)[cron](./cron.md)

---