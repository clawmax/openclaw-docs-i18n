

  إعداد المطور

  
# الإعداد

> **ℹ️** إذا كنت تقوم بالإعداد لأول مرة، ابدأ بـ [الشروع في العمل](./getting-started.md). للتفاصيل حول المعالج، راجع [معالج الإعداد الأولي](./wizard.md).

 آخر تحديث: 2026-01-01

## ملخص سريع

-   **التخصيص خارج المستودع:** `~/.openclaw/workspace` (مساحة العمل) + `~/.openclaw/openclaw.json` (التكوين).
-   **سير العمل المستقر:** قم بتثبيت تطبيق macOS؛ ودعه يشغل Gateway المرفق.
-   **سير العمل المتقدمة:** شغل Gateway بنفسك عبر `pnpm gateway:watch`، ثم دع تطبيق macOS يتصل في وضع المحلي.

## المتطلبات المسبقة (من المصدر)

-   Node `>=22`
-   `pnpm`
-   Docker (اختياري؛ فقط للإعداد المعتمد على الحاويات/اختبارات النهاية إلى النهاية — راجع [Docker](../install/docker.md))

## استراتيجية التخصيص (حتى لا تؤثر التحديثات)

إذا كنت تريد "تخصيص 100٪ لي" *و* تحديثات سهلة، احتفظ بتخصيصاتك في:

-   **التكوين:** `~/.openclaw/openclaw.json` (JSON/JSON5-ish)
-   **مساحة العمل:** `~/.openclaw/workspace` (المهارات، المطالبات، الذكريات؛ اجعلها مستودع git خاص)

قم بالتهيئة مرة واحدة:

```bash
openclaw setup
```

من داخل هذا المستودع، استخدم نقطة دخول CLI المحلية:

```bash
openclaw setup
```

إذا لم يكن لديك تثبيت عام بعد، شغله عبر `pnpm openclaw setup`.

## تشغيل Gateway من هذا المستودع

بعد `pnpm build`، يمكنك تشغيل CLI المعبأ مباشرة:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## سير العمل المستقر (تطبيق macOS أولاً)

1.  قم بتثبيت + تشغيل **OpenClaw.app** (شريط القوائم).
2.  أكمل قائمة التحقق من الإعداد الأولي/الصلاحيات (مطالبات TCC).
3.  تأكد من أن Gateway في وضع **المحلي** ويعمل (التطبيق يديره).
4.  ربط القنوات (مثال: WhatsApp):

```bash
openclaw channels login
```

5.  فحص السلامة:

```bash
openclaw health
```

إذا لم يكن الإعداد الأولي متاحًا في بنيتك:

-   شغل `openclaw setup`، ثم `openclaw channels login`، ثم ابدأ Gateway يدويًا (`openclaw gateway`).

## سير العمل المتقدمة (Gateway في طرفية)

الهدف: العمل على Gateway المكتوب بـ TypeScript، الحصول على إعادة تحميل فورية، والحفاظ على واجهة تطبيق macOS متصلة.

### 0) (اختياري) تشغيل تطبيق macOS من المصدر أيضًا

إذا كنت تريد أيضًا تطبيق macOS في الإصدار المتقدم:

```
./scripts/restart-mac.sh
```

### 1) ابدأ Gateway للتطوير

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` يشغل gateway في وضع المراقبة ويعيد التحميل عند تغييرات TypeScript.

### 2) وجه تطبيق macOS إلى Gateway الذي يعمل لديك

في **OpenClaw.app**:

-   وضع الاتصال: **المحلي** سيتصل التطبيق بـ gateway الذي يعمل على المنفذ المُكوَّن.

### 3) التحقق

-   يجب أن يقرأ حالة Gateway داخل التطبيق **"باستخدام gateway موجود …"**
-   أو عبر CLI:

```bash
openclaw health
```

### الأخطاء الشائعة

-   **المنفذ الخطأ:** Gateway WS يستخدم افتراضيًا `ws://127.0.0.1:18789`؛ حافظ على تطبيق + CLI على نفس المنفذ.
-   **مكان تخزين الحالة:**
    -   بيانات الاعتماد: `~/.openclaw/credentials/`
    -   الجلسات: `~/.openclaw/agents//sessions/`
    -   السجلات: `/tmp/openclaw/`

## خريطة تخزين بيانات الاعتماد

استخدم هذا عند تصحيح أخطاء المصادقة أو عند اتخاذ قرار بشأن ما يجب نسخه احتياطيًا:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **رمز Telegram bot**: في التكوين/env أو `channels.telegram.tokenFile`
-   **رمز Discord bot**: في التكوين/env أو SecretRef (مزودي env/file/exec)
-   **رموز Slack**: في التكوين/env (`channels.slack.*`)
-   **قوائم السماح بالاقتران**:
    -   `~/.openclaw/credentials/-allowFrom.json` (الحساب الافتراضي)
    -   `~/.openclaw/credentials/--allowFrom.json` (الحسابات غير الافتراضية)
-   **ملفات تعريف مصادقة النموذج**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **حمولة الأسرار المدعومة بملف (اختياري)**: `~/.openclaw/secrets.json`
-   **استيراد OAuth القديم**: `~/.openclaw/credentials/oauth.json` المزيد من التفاصيل: [الأمان](../gateway/security.md#credential-storage-map).

## التحديث (دون إفساد إعدادك)

-   احتفظ بـ `~/.openclaw/workspace` و `~/.openclaw/` على أنها "أشياءك الخاصة"؛ لا تضع مطالب/تكوينات شخصية في مستودع `openclaw`.
-   تحديث المصدر: `git pull` + `pnpm install` (عند تغيير ملف القفل) + استمر في استخدام `pnpm gateway:watch`.

## Linux (خدمة مستخدم systemd)

تستخدم تثبيتات Linux خدمة **مستخدم** systemd. افتراضيًا، يوقف systemd خدمات المستخدم عند تسجيل الخروج/عدم النشاط، مما يؤدي إلى إيقاف Gateway. يحاول الإعداد الأولي تمكين التمهل نيابة عنك (قد يطلب sudo). إذا كان لا يزال معطلًا، شغل:

```bash
sudo loginctl enable-linger $USER
```

للخوادم دائمة التشغيل أو متعددة المستخدمين، فكر في استخدام خدمة **نظام** بدلاً من خدمة مستخدم (لا حاجة للتمهل). راجع [دفتر تشغيل Gateway](../gateway.md) للحصول على ملاحظات systemd.

## وثائق ذات صلة

-   [دفتر تشغيل Gateway](../gateway.md) (الأعلام، الإشراف، المنافذ)
-   [تكوين Gateway](../gateway/configuration.md) (مخطط التكوين + أمثلة)
-   [Discord](../channels/discord.md) و [Telegram](../channels/telegram.md) (علامات الرد + إعدادات replyToMode)
-   [إعداد مساعد OpenClaw](./openclaw.md)
-   [تطبيق macOS](../platforms/macos.md) (دورة حياة gateway)

[الغوص العميق في إدارة الجلسات](../reference/session-management-compaction.md)[سير عمل تطوير Pi](../pi-dev.md)

---