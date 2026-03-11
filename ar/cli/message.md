

  أوامر CLI

  
# message

أمر إرسال واحد لإرسال الرسائل وإجراءات القنوات (Discord/Google Chat/Slack/Mattermost (إضافة)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## الاستخدام

```bash
openclaw message <subcommand> [flags]
```

اختيار القناة:

-   `--channel` مطلوب إذا تم تكوين أكثر من قناة واحدة.
-   إذا تم تكوين قناة واحدة بالضبط، تصبح هي الافتراضية.
-   القيم: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost يتطلب إضافة)

تنسيقات الهدف (`--target`):

-   WhatsApp: E.164 أو معرف المجموعة JID
-   Telegram: معرف الدردشة أو `@username`
-   Discord: `channel:` أو `user:` (أو `<@id>` للإشارة؛ يتم التعامل مع المعرفات الرقمية الخام كقنوات)
-   Google Chat: `spaces/` أو `users/`
-   Slack: `channel:` أو `user:` (يتم قبول معرف القناة الخام)
-   Mattermost (إضافة): `channel:`، `user:`، أو `@username` (يتم التعامل مع المعرفات المجردة كقنوات)
-   Signal: `+E.164`، `group:`، `signal:+E.164`، `signal:group:`، أو `username:`/`u:`
-   iMessage: المعرف، `chat_id:`، `chat_guid:`، أو `chat_identifier:`
-   MS Teams: معرف المحادثة (`19:...@thread.tacv2`) أو `conversation:` أو `user:<aad-object-id>`

البحث بالاسم:

-   بالنسبة لمزودي الخدمة المدعومين (Discord/Slack/إلخ)، يتم حل أسماء القنوات مثل `Help` أو `#help` عبر ذاكرة التخزين المؤقت للدليل.
-   في حالة عدم وجودها في ذاكرة التخزين المؤقت، سيحاول OpenClaw إجراء بحث مباشر في الدليل عندما يدعم المزود ذلك.

## الأعلام الشائعة

-   `--channel `
-   `--account `
-   `--target ` (القناة أو المستخدم الهدف للإرسال/الاستطلاع/القراءة/إلخ)
-   `--targets ` (كرر؛ للبث فقط)
-   `--json`
-   `--dry-run`
-   `--verbose`

## الإجراءات

### الأساسية

-   `send`
    -   القنوات: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (إضافة)/Signal/iMessage/MS Teams
    -   المطلوب: `--target`، بالإضافة إلى `--message` أو `--media`
    -   اختياري: `--media`، `--reply-to`، `--thread-id`، `--gif-playback`
    -   Telegram فقط: `--buttons` (يتطلب `channels.telegram.capabilities.inlineButtons` للسماح به)
    -   Telegram فقط: `--thread-id` (معرف موضوع المنتدى)
    -   Slack فقط: `--thread-id` (طابع زمني للمحادثة؛ `--reply-to` يستخدم نفس الحقل)
    -   WhatsApp فقط: `--gif-playback`
-   `poll`
    -   القنوات: WhatsApp/Telegram/Discord/Matrix/MS Teams
    -   المطلوب: `--target`، `--poll-question`، `--poll-option` (كرر)
    -   اختياري: `--poll-multi`
    -   Discord فقط: `--poll-duration-hours`، `--silent`، `--message`
    -   Telegram فقط: `--poll-duration-seconds` (5-600)، `--silent`، `--poll-anonymous` / `--poll-public`، `--thread-id`
-   `react`
    -   القنوات: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
    -   المطلوب: `--message-id`، `--target`
    -   اختياري: `--emoji`، `--remove`، `--participant`، `--from-me`، `--target-author`، `--target-author-uuid`
    -   ملاحظة: `--remove` يتطلب `--emoji` (احذف `--emoji` لمسح ردود فعلك الخاصة حيثما يدعم؛ انظر /tools/reactions)
    -   WhatsApp فقط: `--participant`، `--from-me`
    -   ردود فعل مجموعة Signal: `--target-author` أو `--target-author-uuid` مطلوب
-   `reactions`
    -   القنوات: Discord/Google Chat/Slack
    -   المطلوب: `--message-id`، `--target`
    -   اختياري: `--limit`
-   `read`
    -   القنوات: Discord/Slack
    -   المطلوب: `--target`
    -   اختياري: `--limit`، `--before`، `--after`
    -   Discord فقط: `--around`
-   `edit`
    -   القنوات: Discord/Slack
    -   المطلوب: `--message-id`، `--message`، `--target`
-   `delete`
    -   القنوات: Discord/Slack/Telegram
    -   المطلوب: `--message-id`، `--target`
-   `pin` / `unpin`
    -   القنوات: Discord/Slack
    -   المطلوب: `--message-id`، `--target`
-   `pins` (قائمة)
    -   القنوات: Discord/Slack
    -   المطلوب: `--target`
-   `permissions`
    -   القنوات: Discord
    -   المطلوب: `--target`
-   `search`
    -   القنوات: Discord
    -   المطلوب: `--guild-id`، `--query`
    -   اختياري: `--channel-id`، `--channel-ids` (كرر)، `--author-id`، `--author-ids` (كرر)، `--limit`

### المحادثات (Threads)

-   `thread create`
    -   القنوات: Discord
    -   المطلوب: `--thread-name`، `--target` (معرف القناة)
    -   اختياري: `--message-id`، `--message`، `--auto-archive-min`
-   `thread list`
    -   القنوات: Discord
    -   المطلوب: `--guild-id`
    -   اختياري: `--channel-id`، `--include-archived`، `--before`، `--limit`
-   `thread reply`
    -   القنوات: Discord
    -   المطلوب: `--target` (معرف المحادثة)، `--message`
    -   اختياري: `--media`، `--reply-to`

### الرموز التعبيرية (Emojis)

-   `emoji list`
    -   Discord: `--guild-id`
    -   Slack: لا توجد أعلام إضافية
-   `emoji upload`
    -   القنوات: Discord
    -   المطلوب: `--guild-id`، `--emoji-name`، `--media`
    -   اختياري: `--role-ids` (كرر)

### الملصقات (Stickers)

-   `sticker send`
    -   القنوات: Discord
    -   المطلوب: `--target`، `--sticker-id` (كرر)
    -   اختياري: `--message`
-   `sticker upload`
    -   القنوات: Discord
    -   المطلوب: `--guild-id`، `--sticker-name`، `--sticker-desc`، `--sticker-tags`، `--media`

### الأدوار / القنوات / الأعضاء / الصوت

-   `role info` (Discord): `--guild-id`
-   `role add` / `role remove` (Discord): `--guild-id`، `--user-id`، `--role-id`
-   `channel info` (Discord): `--target`
-   `channel list` (Discord): `--guild-id`
-   `member info` (Discord/Slack): `--user-id` (+ `--guild-id` لـ Discord)
-   `voice status` (Discord): `--guild-id`، `--user-id`

### الأحداث

-   `event list` (Discord): `--guild-id`
-   `event create` (Discord): `--guild-id`، `--event-name`، `--start-time`
    -   اختياري: `--end-time`، `--desc`، `--channel-id`، `--location`، `--event-type`

### الإشراف (Discord)

-   `timeout`: `--guild-id`، `--user-id` (اختياري `--duration-min` أو `--until`؛ احذف كليهما لمسح المهلة)
-   `kick`: `--guild-id`، `--user-id` (+ `--reason`)
-   `ban`: `--guild-id`، `--user-id` (+ `--delete-days`، `--reason`)
    -   `timeout` يدعم أيضًا `--reason`

### البث

-   `broadcast`
    -   القنوات: أي قناة مكونة؛ استخدم `--channel all` لاستهداف جميع المزودين
    -   المطلوب: `--targets` (كرر)
    -   اختياري: `--message`، `--media`، `--dry-run`

## أمثلة

إرسال رد على Discord:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

إرسال رسالة Discord مع مكونات:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

انظر [مكونات Discord](../channels/discord.md#interactive-components) للحصول على المخطط الكامل. إنشاء استطلاع Discord:

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

إنشاء استطلاع Telegram (يغلق تلقائيًا بعد دقيقتين):

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

إرسال رسالة استباقية على Teams:

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

إنشاء استطلاع على Teams:

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

رد فعل في Slack:

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

رد فعل في مجموعة Signal:

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

إرسال أزرار مضمنة في Telegram:

```bash
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)