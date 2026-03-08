

  技術リファレンス

  
# SecretRef 認証情報サーフェス

このページでは、正規の SecretRef 認証情報サーフェスを定義します。スコープの意図:

-   スコープ内: OpenClaw が発行またはローテーションしない、厳密にユーザー提供の認証情報。
-   スコープ外: ランタイム発行またはローテーションする認証情報、OAuth リフレッシュマテリアル、セッション様のアーティファクト。

## サポートされる認証情報

### openclaw.json のターゲット (secrets configure + secrets apply + secrets audit)

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
-   `channels.googlechat.serviceAccount` via sibling `serviceAccountRef` (互換性例外)
-   `channels.googlechat.accounts.*.serviceAccount` via sibling `serviceAccountRef` (互換性例外)

### auth-profiles.json のターゲット (secrets configure + secrets apply + secrets audit)

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

注記:

-   認証プロファイルの計画ターゲットには `agentId` が必要です。
-   計画エントリは `profiles.*.key` / `profiles.*.token` をターゲットとし、兄弟参照 (`keyRef` / `tokenRef`) を書き込みます。
-   認証プロファイル参照は、ランタイム解決と監査カバレッジに含まれます。
-   SecretRef 管理のモデルプロバイダーでは、生成された `agents/*/agent/models.json` エントリは、`apiKey`/ヘッダーサーフェスに対して非シークレットマーカー（解決されたシークレット値ではない）を保持します。
-   Web 検索の場合:
    -   明示的プロバイダーモード (`tools.web.search.provider` 設定時) では、選択されたプロバイダーのキーのみがアクティブです。
    -   自動モード (`tools.web.search.provider` 未設定) では、`tools.web.search.apiKey` とプロバイダー固有のキーがアクティブです。

## サポートされない認証情報

スコープ外の認証情報には以下が含まれます:

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

理由:

-   これらの認証情報は、発行、ローテーション、セッション保持、または読み取り専用の外部 SecretRef 解決に適合しない OAuth 耐久クラスです。

[トークン使用とコスト](./token-use.md)[プロンプトキャッシング](./prompt-caching.md)