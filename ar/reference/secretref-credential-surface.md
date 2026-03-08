

  مرجع تقني

  
# سطح بيانات الاعتماد SecretRef

تحدد هذه الصفحة سطح بيانات الاعتماد الأساسي لـ SecretRef. نطاق القصد:

-   ضمن النطاق: بيانات الاعتماد التي يزودها المستخدم فقط ولا يقوم OpenClaw بإنشائها أو تدويرها.
-   خارج النطاق: بيانات الاعتماد التي يتم إنشاؤها أثناء التشغيل أو التي يتم تدويرها، ومواد تحديث OAuth، والقطع الأثرية الشبيهة بالجلسات.

## بيانات الاعتماد المدعومة

### أهداف openclaw.json (تكوين الأسرار + تطبيق الأسرار + تدقيق الأسرار)

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
-   `channels.googlechat.serviceAccount` عبر `serviceAccountRef` الشقيق (استثناء التوافق)
-   `channels.googlechat.accounts.*.serviceAccount` عبر `serviceAccountRef` الشقيق (استثناء التوافق)

### أهداف auth-profiles.json (تكوين الأسرار + تطبيق الأسرار + تدقيق الأسرار)

-   `profiles.*.keyRef` (`type: "api_key"`)
-   `profiles.*.tokenRef` (`type: "token"`)

ملاحظات:

-   أهداف خطة ملف تعريف المصادقة تتطلب `agentId`.
-   تستهدف إدخالات الخطة `profiles.*.key` / `profiles.*.token` وتكتب المراجع الشقيقة (`keyRef` / `tokenRef`).
-   يتم تضمين مراجع ملف تعريف المصادقة في دقة وقت التشغيل وتغطية التدقيق.
-   بالنسبة لموفري النماذج المدارين بواسطة SecretRef، تحافظ إدخالات `agents/*/agent/models.json` المُنشأة على علامات غير سرية (وليس قيم الأسرار المحلولة) لأسطح `apiKey`/الرؤوس.
-   بالنسبة للبحث على الويب:
    -   في وضع المزود الصريح (تم تعيين `tools.web.search.provider`)، يكون مفتاح المزود المحدد فقط نشطًا.
    -   في الوضع التلقائي (لم يتم تعيين `tools.web.search.provider`)، يكون `tools.web.search.apiKey` والمفاتيح الخاصة بالمزود نشطة.

## بيانات الاعتماد غير المدعومة

تشمل بيانات الاعتماد خارج النطاق:

-   `commands.ownerDisplaySecret`
-   `channels.matrix.accessToken`
-   `channels.matrix.accounts.*.accessToken`
-   `hooks.token`
-   `hooks.gmail.pushToken`
-   `hooks.mappings[].sessionKey`
-   `auth-profiles.oauth.*`
-   `discord.threadBindings.*.webhookToken`
-   `whatsapp.creds.json`

المبرر:

-   هذه بيانات اعتماد يتم إنشاؤها أو تدويرها أو تحمل جلسات أو فئات دائمة OAuth لا تناسب دقة SecretRef الخارجية للقراءة فقط.

[استخدام الرمز المميز والتكاليف](./token-use.md)[التخزين المؤقت للمطالبة](./prompt-caching.md)