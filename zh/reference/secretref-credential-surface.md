

  技术参考

  
# SecretRef 凭证表面

本页面定义了规范的 SecretRef 凭证表面。范围界定如下：

-   **范围内**：严格指用户提供的、OpenClaw 不生成或轮换的凭证。
-   **范围外**：运行时生成或轮换的凭证、OAuth 刷新材料以及会话类工件。

## 支持的凭证

### openclaw.json 目标（secrets configure + secrets apply + secrets audit）

-   `models.providers.*.apiKey`
-   `models.providers.*.headers.*`
-   `skills.entries.*.apiKey`
-   `agents.defaults.memorySearch.remote.apiKey`
-   `agents.list[].memorySearch.remote.apiKey`
-   `talk.apiKey`
-   `talk.providers.*.apiKey`
-   `messages.tts.elevenlabs.apiKey`
-   `messages.tts.openai.apiKey`
-   `tools.web.search.apiKey`
-   `tools.web.search.gemini.apiKey`
-   `tools.web.search.grok.apiKey`
-   `tools.web.search.kimi.apiKey`
-   `tools.web.search.perplexity.apiKey`
-   `gateway.auth.password`
-   `gateway.auth.token`
-   `gateway.remote.token`
-   `gateway.remote.password`
-   `cron.webhookToken`
-   `channels.telegram.botToken`
-   `channels.telegram.webhookSecret`
-   `channels.telegram.accounts.*.botToken`
-   `channels.telegram.accounts.*.webhookSecret`
-   `channels.slack.botToken`
-   `channels.slack.appToken`
-   `channels.slack.userToken`
-   `channels.slack.signingSecret`
-   `channels.slack.accounts.*.botToken`
-   `channels.slack.accounts.*.appToken`
-   `channels.slack.accounts.*.userToken`
-   `channels.slack.accounts.*.signingSecret`
-   `channels.discord.token`
-   `channels.discord.pluralkit.token`
-   `channels.discord.voice.tts.elevenlabs.apiKey`
-   `channels.discord.voice.tts.openai.apiKey`
-   `channels.discord.accounts.*.token`
-   `channels.discord.accounts.*.pluralkit.token`
-   `channels.discord.accounts.*.voice.tts.elevenlabs.apiKey`
-   `channels.discord.accounts.*.voice.tts.openai.apiKey`
-   `channels.irc.password`
-   `channels.irc.nickserv.password`
-   `channels.irc.accounts.*.password`
-   `channels.irc.accounts.*.nickserv.password`
-   `channels.bluebubbles.password`
-   `channels.bluebubbles.accounts.*.password`
-   `channels.feishu.appSecret`
-   `channels.feishu.verificationToken`
-   `channels.feishu.accounts.*.appSecret`
-   `channels.feishu.accounts.*.verificationToken`
-   `channels.msteams.appPassword`
-   `channels.mattermost.botToken`
-   `channels.mattermost.accounts.*.botToken`
-   `channels.matrix.password`
-   `channels.matrix.accounts.*.password`
-   `channels.nextcloud-talk.botSecret`
-   `channels.nextcloud-talk.apiPassword`
-   `channels.nextcloud-talk.accounts.*.botSecret`
-   `channels.nextcloud-talk.accounts.*.apiPassword`
-   `channels.zalo.botToken`
-   `channels.zalo.webhookSecret`
-   `channels.zalo.accounts.*.botToken`
-   `channels.zalo.accounts.*.webhookSecret`
-   `channels.googlechat.serviceAccount` 通过同级的 `serviceAccountRef`（兼容性例外）
-   `channels.googlechat.accounts.*.serviceAccount` 通过同级的 `serviceAccountRef`（兼容性例外）

### auth-profiles.json 目标（secrets configure + secrets apply + secrets audit）

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

注意：

-   身份验证配置文件计划目标需要 `agentId`。
-   计划条目目标为 `profiles.*.key` / `profiles.*.token`，并写入同级的引用（`keyRef` / `tokenRef`）。
-   身份验证配置文件引用包含在运行时解析和审计覆盖范围内。
-   对于由 SecretRef 管理的模型提供商，生成的 `agents/*/agent/models.json` 条目会为 `apiKey`/header 表面保留非密钥标记（而非解析后的密钥值）。
-   对于网络搜索：
    -   在显式提供商模式（设置了 `tools.web.search.provider`）下，仅选定的提供商密钥处于活动状态。
    -   在自动模式（未设置 `tools.web.search.provider`）下，`tools.web.search.apiKey` 和特定于提供商的密钥均处于活动状态。

## 不支持的凭证

范围外的凭证包括：

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

理由：

-   这些凭证属于生成型、轮换型、会话持有型或 OAuth 持久型类别，不适合只读的外部 SecretRef 解析。

[令牌使用与成本](./token-use.md)[提示词缓存](./prompt-caching.md)