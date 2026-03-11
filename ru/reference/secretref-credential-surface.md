

  Технический справочник

  
# Поверхность учетных данных SecretRef

На этой странице определена каноническая поверхность учетных данных SecretRef. Цель области действия:

-   В области действия: строго учетные данные, предоставленные пользователем, которые OpenClaw не создает и не обновляет.
-   Вне области действия: создаваемые во время выполнения или обновляемые учетные данные, материалы обновления OAuth и артефакты, подобные сессиям.

## Поддерживаемые учетные данные

### Цели в openclaw.json (secrets configure + secrets apply + secrets audit)

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
-   `channels.googlechat.serviceAccount` через sibling `serviceAccountRef` (исключение для совместимости)
-   `channels.googlechat.accounts.*.serviceAccount` через sibling `serviceAccountRef` (исключение для совместимости)

### Цели в auth-profiles.json (secrets configure + secrets apply + secrets audit)

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

Примечания:

-   Цели плана профилей аутентификации требуют `agentId`.
-   Записи плана нацелены на `profiles.*.key` / `profiles.*.token` и записывают sibling ссылки (`keyRef` / `tokenRef`).
-   Ссылки профилей аутентификации включены в разрешение во время выполнения и покрытие аудита.
-   Для провайдеров моделей, управляемых через SecretRef, сгенерированные записи `agents/*/agent/models.json` сохраняют несекретные маркеры (не разрешенные значения секретов) для поверхностей `apiKey`/заголовков.
-   Для веб-поиска:
    -   В режиме явного провайдера (установлен `tools.web.search.provider`) активен только ключ выбранного провайдера.
    -   В автоматическом режиме (не установлен `tools.web.search.provider`) активны `tools.web.search.apiKey` и ключи, специфичные для провайдера.

## Неподдерживаемые учетные данные

Учетные данные вне области действия включают:

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

Обоснование:

-   Эти учетные данные относятся к классам, которые создаются, обновляются, являются сессионными или относятся к долговременным классам OAuth и не подходят для разрешения внешних SecretRef только для чтения.

[Использование токенов и стоимость](./token-use.md)[Кэширование промптов](./prompt-caching.md)