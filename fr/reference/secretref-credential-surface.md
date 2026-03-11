

  Référence technique

  
# Surface d'identification SecretRef

Cette page définit la surface d'identification canonique pour SecretRef. Intention du périmètre :

-   Dans le périmètre : strictement les identifiants fournis par l'utilisateur qu'OpenClaw ne génère ni ne fait tourner.
-   Hors périmètre : les identifiants générés à l'exécution ou en rotation, le matériel de rafraîchissement OAuth, et les artefacts de type session.

## Identifiants pris en charge

### Cibles openclaw.json (secrets configure + secrets apply + secrets audit)

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
-   `channels.googlechat.serviceAccount` via sibling `serviceAccountRef` (exception de compatibilité)
-   `channels.googlechat.accounts.*.serviceAccount` via sibling `serviceAccountRef` (exception de compatibilité)

### Cibles auth-profiles.json (secrets configure + secrets apply + secrets audit)

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

Notes :

-   Les cibles de plan de profil d'authentification nécessitent `agentId`.
-   Les entrées de plan ciblent `profiles.*.key` / `profiles.*.token` et écrivent les références sœurs (`keyRef` / `tokenRef`).
-   Les références de profil d'authentification sont incluses dans la résolution à l'exécution et la couverture d'audit.
-   Pour les fournisseurs de modèles gérés par SecretRef, les entrées générées `agents/*/agent/models.json` conservent des marqueurs non secrets (pas les valeurs secrètes résolues) pour les surfaces `apiKey`/header.
-   Pour la recherche web :
    -   En mode fournisseur explicite (`tools.web.search.provider` défini), seule la clé du fournisseur sélectionné est active.
    -   En mode automatique (`tools.web.search.provider` non défini), `tools.web.search.apiKey` et les clés spécifiques aux fournisseurs sont actives.

## Identifiants non pris en charge

Les identifiants hors périmètre incluent :

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

Justification :

-   Ces identifiants sont de classes générées, en rotation, portant une session, ou durables OAuth qui ne correspondent pas à la résolution externe en lecture seule de SecretRef.

[Utilisation et coûts des jetons](./token-use.md)[Mise en cache des invites](./prompt-caching.md)

---