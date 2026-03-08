

  提供商

  
# Qwen

Qwen 为 Qwen Coder 和 Qwen Vision 模型提供免费层级的 OAuth 流程（每日 2,000 次请求，受 Qwen 速率限制约束）。

## 启用插件

```bash
openclaw plugins enable qwen-portal-auth
```

启用后重启 Gateway。

## 认证

```bash
openclaw models auth login --provider qwen-portal --set-default
```

此命令将运行 Qwen 设备码 OAuth 流程，并将一个提供商条目写入您的 `models.json` 文件（同时创建一个 `qwen` 别名以便快速切换）。

## 模型 ID

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

使用以下命令切换模型：

```bash
openclaw models set qwen-portal/coder-model
```

## 复用 Qwen Code CLI 登录

如果您已使用 Qwen Code CLI 登录，OpenClaw 在加载认证存储时会从 `~/.qwen/oauth_creds.json` 同步凭据。您仍然需要一个 `models.providers.qwen-portal` 条目（使用上面的登录命令来创建一个）。

## 注意事项

-   令牌会自动刷新；如果刷新失败或访问权限被撤销，请重新运行登录命令。
-   默认基础 URL：`https://portal.qwen.ai/v1`（如果 Qwen 提供了不同的端点，可使用 `models.providers.qwen-portal.baseUrl` 覆盖）。
-   有关提供商范围的规则，请参阅[模型提供商](../concepts/model-providers.md)。

[千帆](./qianfan.md)[合成数据](./synthetic.md)

---