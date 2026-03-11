

  نظرة عامة

  
# قنوات الدردشة

يمكن لـ OpenClaw التحدث معك على أي تطبيق دردشة تستخدمه بالفعل. كل قناة تتصل عبر البوابة (Gateway). النصوص مدعومة في كل مكان؛ بينما تختلف دعم الوسائط وردود الفعل حسب القناة.

## القنوات المدعومة

-   [BlueBubbles](./channels/bluebubbles.md) — **موصى به لـ iMessage**؛ يستخدم واجهة برمجة تطبيقات REST لخادم BlueBubbles على macOS مع دعم كامل للميزات (التعديل، الإلغاء، المؤثرات، ردود الفعل، إدارة المجموعات — التعديل معطل حاليًا على macOS 26 Tahoe).
-   [Discord](./channels/discord.md) — واجهة برمجة تطبيقات بوت Discord + البوابة؛ يدعم الخوادم، والقنوات، والرسائل المباشرة.
-   [Feishu](./channels/feishu.md) — بوت Feishu/Lark عبر WebSocket (إضافة، يتم تثبيتها بشكل منفصل).
-   [Google Chat](./channels/googlechat.md) — تطبيق واجهة برمجة تطبيقات Google Chat عبر Webhook HTTP.
-   [iMessage (قديم)](./channels/imessage.md) — تكامل قديم لنظام macOS عبر سطر أوامر imsg (مهمل، استخدم BlueBubbles للإعدادات الجديدة).
-   [IRC](./channels/irc.md) — خوادم IRC الكلاسيكية؛ القنوات + الرسائل المباشرة مع ضوابط الاقتران/القائمة المسموح بها.
-   [LINE](./channels/line.md) — بوت واجهة برمجة تطبيقات LINE للمراسلة (إضافة، يتم تثبيتها بشكل منفصل).
-   [Matrix](./channels/matrix.md) — بروتوكول Matrix (إضافة، يتم تثبيتها بشكل منفصل).
-   [Mattermost](./channels/mattermost.md) — واجهة برمجة تطبيقات البوت + WebSocket؛ القنوات، المجموعات، الرسائل المباشرة (إضافة، يتم تثبيتها بشكل منفصل).
-   [Microsoft Teams](./channels/msteams.md) — إطار عمل البوت؛ دعم المؤسسات (إضافة، يتم تثبيتها بشكل منفصل).
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — دردشة ذاتية الاستضافة عبر Nextcloud Talk (إضافة، يتم تثبيتها بشكل منفصل).
-   [Nostr](./channels/nostr.md) — رسائل مباشرة لامركزية عبر NIP-04 (إضافة، يتم تثبيتها بشكل منفصل).
-   [Signal](./channels/signal.md) — signal-cli؛ يركز على الخصوصية.
-   [Synology Chat](./channels/synology-chat.md) — دردشة Synology NAS عبر webhooks صادرة وواردة (إضافة، يتم تثبيتها بشكل منفصل).
-   [Slack](./channels/slack.md) — Bolt SDK؛ تطبيقات مساحة العمل.
-   [Telegram](./channels/telegram.md) — واجهة برمجة تطبيقات البوت عبر grammY؛ يدعم المجموعات.
-   [Tlon](./channels/tlon.md) — مراسل يعتمد على Urbit (إضافة، يتم تثبيتها بشكل منفصل).
-   [Twitch](./channels/twitch.md) — دردشة Twitch عبر اتصال IRC (إضافة، يتم تثبيتها بشكل منفصل).
-   [WebChat](./web/webchat.md) — واجهة مستخدم WebChat للبوابة عبر WebSocket.
-   [WhatsApp](./channels/whatsapp.md) — الأكثر شعبية؛ يستخدم Baileys ويتطلب اقترانًا عبر QR.
-   [Zalo](./channels/zalo.md) — واجهة برمجة تطبيقات بوت Zalo؛ المراسل الشعبي في فيتنام (إضافة، يتم تثبيتها بشكل منفصل).
-   [Zalo Personal](./channels/zalouser.md) — حساب Zalo الشخصي عبر تسجيل الدخول بـ QR (إضافة، يتم تثبيتها بشكل منفصل).

## ملاحظات

-   يمكن تشغيل القنوات في وقت واحد؛ قم بتكوين عدة قنوات وسيقوم OpenClaw بتوجيه المحادثات وفقًا لذلك.
-   أسرع إعداد عادةً هو **Telegram** (رمز بوت بسيط). يتطلب WhatsApp اقترانًا عبر QR ويخزن حالة أكثر على القرص.
-   يختلف سلوك المجموعات حسب القناة؛ راجع [المجموعات](./channels/groups.md).
-   يتم فرض الاقتران في الرسائل المباشرة والقوائم المسموح بها لأسباب أمنية؛ راجع [الأمان](./gateway/security.md).
-   استكشاف الأخطاء وإصلاحها: [استكشاف أخطاء القناة وإصلاحها](./channels/troubleshooting.md).
-   موفرو النماذج موثقون بشكل منفصل؛ راجع [موفرو النماذج](./providers/models.md).

[BlueBubbles](./channels/bluebubbles.md)

---