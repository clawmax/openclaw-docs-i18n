

  Referencia técnica

  
# Superficie de Credenciales SecretRef

Esta página define la superficie canónica de credenciales SecretRef. Intención del alcance:

-   En alcance: estrictamente credenciales proporcionadas por el usuario que OpenClaw no genera ni rota.
-   Fuera de alcance: credenciales generadas en tiempo de ejecución o rotativas, material de actualización OAuth y artefactos similares a sesiones.

## Credenciales admitidas

### Destinos en openclaw.json (secrets configure + secrets apply + secrets audit)

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
-   `channels.googlechat.serviceAccount` a través del campo hermano `serviceAccountRef` (excepción de compatibilidad)
-   `channels.googlechat.accounts.*.serviceAccount` a través del campo hermano `serviceAccountRef` (excepción de compatibilidad)

### Destinos en auth-profiles.json (secrets configure + secrets apply + secrets audit)

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

Notas:

-   Los destinos de plan de perfil de autenticación requieren `agentId`.
-   Las entradas del plan apuntan a `profiles.*.key` / `profiles.*.token` y escriben referencias hermanas (`keyRef` / `tokenRef`).
-   Las referencias de perfil de autenticación están incluidas en la resolución en tiempo de ejecución y la cobertura de auditoría.
-   Para proveedores de modelos gestionados por SecretRef, las entradas generadas en `agents/*/agent/models.json` persisten marcadores no secretos (no valores secretos resueltos) para las superficies `apiKey`/header.
-   Para búsqueda web:
    -   En modo de proveedor explícito (`tools.web.search.provider` establecido), solo la clave del proveedor seleccionado está activa.
    -   En modo automático (`tools.web.search.provider` sin establecer), `tools.web.search.apiKey` y las claves específicas del proveedor están activas.

## Credenciales no admitidas

Las credenciales fuera de alcance incluyen:

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

Razón:

-   Estas credenciales son de clases generadas, rotativas, que portan sesión o durables OAuth que no se ajustan a la resolución de SecretRef externa de solo lectura.

[Uso y Costos de Tokens](./token-use.md)[Almacenamiento en Caché de Prompts](./prompt-caching.md)