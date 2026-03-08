title: "دليل اختبار OpenClaw: مجموعات اختبار الوحدة، E2E، والاختبارات الحية"
description: "تعلم كيفية اختبار OpenClaw باستخدام مجموعات اختبار الوحدة، التكامل، E2E، والاختبارات الحية. تشغيل أوامر التصحيح، التغطية، والتحقق من مقدمي الخدمة والنماذج الحقيقية."
keywords: ["اختبار openclaw", "مجموعات vitest", "اختبار e2e", "اختبارات حية", "اختبار دخان البوابة", "اختبار عقدة android", "تكامل النموذج", "تغطية الاختبار"]
---

  البيئة والتصحيح

  
# الاختبار

يحتوي OpenClaw على ثلاث مجموعات Vitest (وحدة/تكامل، e2e، حية) ومجموعة صغيرة من عدّادات Docker. هذا المستند هو دليل "كيف نختبر":

-   ما تغطيه كل مجموعة (وما لا تغطيه عمداً)
-   الأوامر التي يجب تشغيلها لسير العمل الشائعة (محلي، ما قبل الدفع، التصحيح)
-   كيف تكتشف الاختبارات الحية بيانات الاعتماد وتختار النماذج/مقدمي الخدمة
-   كيفية إضافة اختبارات رجعية لمشاكل النماذج/مقدمي الخدمة في العالم الحقيقي

## البدء السريع

في معظم الأيام:

-   البوابة الكاملة (متوقعة قبل الدفع): `pnpm build && pnpm check && pnpm test`

عندما تلمس الاختبارات أو تريد ثقة إضافية:

-   بوابة التغطية: `pnpm test:coverage`
-   مجموعة E2E: `pnpm test:e2e`

عند تصحيح مقدمي الخدمة/النماذج الحقيقية (يتطلب بيانات اعتماد حقيقية):

-   المجموعة الحية (النماذج + فحص أدوات/صور البوابة): `pnpm test:live`

نصيحة: عندما تحتاج فقط لحالة فاشلة واحدة، يُفضل تضييق الاختبارات الحية عبر متغيرات البيئة لقائمة السماح الموضحة أدناه.

## مجموعات الاختبار (ما الذي يعمل وأين)

فكر في المجموعات على أنها "زيادة الواقعية" (وزيادة التقلب/التكلفة):

### الوحدة / التكامل (الافتراضي)

-   الأمر: `pnpm test`
-   التكوين: `scripts/test-parallel.mjs` (يشغل `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
-   الملفات: `src/**/*.test.ts`, `extensions/**/*.test.ts`
-   النطاق:
    -   اختبارات وحدة خالصة
    -   اختبارات تكامل داخلية (مصادقة البوابة، التوجيه، الأدوات، التحليل، التكوين)
    -   اختبارات رجعية حتمية للأخطاء المعروفة
-   التوقعات:
    -   يعمل في CI
    -   لا يتطلب مفاتيح حقيقية
    -   يجب أن يكون سريعاً ومستقراً
-   ملاحظة حول المجمع:
    -   يستخدم OpenClaw `vmForks` من Vitest على Node 22/23 لتقسيمات الوحدة الأسرع.
    -   على Node 24+، يعود OpenClaw تلقائياً إلى `forks` العادية لتجنب أخطاء ربط Node VM (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
    -   تجاوز يدوياً باستخدام `OPENCLAW_TEST_VM_FORKS=0` (فرض `forks`) أو `OPENCLAW_TEST_VM_FORKS=1` (فرض `vmForks`).

### E2E (اختبار دخان البوابة)

-   الأمر: `pnpm test:e2e`
-   التكوين: `vitest.e2e.config.ts`
-   الملفات: `src/**/*.e2e.test.ts`
-   إعدادات وقت التشغيل الافتراضية:
    -   يستخدم `vmForks` من Vitest لبدء أسرع للملفات.
    -   يستخدم عمال تكيفيين (CI: 2-4، محلي: 4-8).
    -   يعمل في الوضع الصامت افتراضياً لتقليل حمل إدخال/إخراج وحدة التحكم.
-   تجاوزات مفيدة:
    -   `OPENCLAW_E2E_WORKERS=` لفرض عدد العمال (بحد أقصى 16).
    -   `OPENCLAW_E2E_VERBOSE=1` لإعادة تمكين الإخراج التفصيلي لوحدة التحكم.
-   النطاق:
    -   سلوك البوابة من طرف إلى طرف متعدد الحالات
    -   أسطح WebSocket/HTTP، إقران العقد، والشبكات الأثقل
-   التوقعات:
    -   يعمل في CI (عند تمكينه في خط الأنابيب)
    -   لا يتطلب مفاتيح حقيقية
    -   أجزاء متحركة أكثر من اختبارات الوحدة (يمكن أن يكون أبطأ)

### الاختبارات الحية (مقدمو خدمة حقيقيون + نماذج حقيقية)

-   الأمر: `pnpm test:live`
-   التكوين: `vitest.live.config.ts`
-   الملفات: `src/**/*.live.test.ts`
-   الافتراضي: **مُمكّن** بواسطة `pnpm test:live` (يضبط `OPENCLAW_LIVE_TEST=1`)
-   النطاق:
    -   "هل يعمل مقدم الخدمة/النموذج هذا *اليوم* فعلاً مع بيانات الاعتماد الحقيقية؟"
    -   اكتشاف تغييرات تنسيق مقدم الخدمة، غرائب استدعاء الأدوات، مشاكل المصادقة، وسلوك حد المعدل
-   التوقعات:
    -   غير مستقر في CI حسب التصميم (شبكات حقيقية، سياسات مقدم خدمة حقيقية، حصص، انقطاعات)
    -   يكلف مالاً / يستخدم حدود المعدل
    -   يُفضل تشغيل مجموعات فرعية ضيقة بدلاً من "كل شيء"
    -   ستعمل التشغيلات الحية على تحميل `~/.profile` لالتقاط مفاتيح API المفقودة
-   تناوب مفتاح API (خاص بمقدم الخدمة): اضبط `*_API_KEYS` بتنسيق فاصلة/فاصلة منقوطة أو `*_API_KEY_1`, `*_API_KEY_2` (على سبيل المثال `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) أو تجاوز لكل اختبار حي عبر `OPENCLAW_LIVE_*_KEY`؛ تحاول الاختبارات مرة أخرى عند استجابات حد المعدل.

## أي مجموعة يجب أن أُشغِّل؟

استخدم جدول القرار هذا:

-   تحرير المنطق/الاختبارات: شغِّل `pnpm test` (و `pnpm test:coverage` إذا قمت بتغيير الكثير)
-   لمس شبكات البوابة / بروتوكول WS / الإقران: أضف `pnpm test:e2e`
-   تصحيح "بوتي معطل" / أعطال خاصة بمقدم الخدمة / استدعاء الأدوات: شغِّل مجموعة ضيقة من `pnpm test:live`

## الاختبارات الحية: فحص قدرات عقدة Android

-   الاختبار: `src/gateway/android-node.capabilities.live.test.ts`
-   السكريبت: `pnpm android:test:integration`
-   الهدف: استدعاء **كل أمر معلن عنه حالياً** بواسطة عقدة Android متصلة والتأكد من سلوك عقد الأمر.
-   النطاق:
    -   إعداد مسبق/يدوي (المجموعة لا تقوم بتثبيت/تشغيل/إقران التطبيق).
    -   تحقق من `node.invoke` للبوابة أمراً بأمر للعقدة Android المحددة.
-   الإعداد المسبق المطلوب:
    -   تطبيق Android متصل بالفعل + مقترن بالبوابة.
    -   إبقاء التطبيق في المقدمة.
    -   منح الأذونات/موافقة الالتقاط للقدرات التي تتوقع نجاحها.
-   تجاوزات الهدف الاختيارية:
    -   `OPENCLAW_ANDROID_NODE_ID` أو `OPENCLAW_ANDROID_NODE_NAME`.
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
-   تفاصيل إعداد Android الكاملة: [تطبيق Android](../platforms/android.md)

## الاختبارات الحية: اختبار دخان النموذج (مفاتيح الملف الشخصي)

تنقسم الاختبارات الحية إلى طبقتين حتى نتمكن من عزل الأعطال:

-   "النموذج المباشر" يخبرنا أن مقدم الخدمة/النموذج يمكنه الإجابة على الإطلاق بالمفتاح المعطى.
-   "اختبار دخان البوابة" يخبرنا أن خط أنابيب البوابة+الوكيل الكامل يعمل لذلك النموذج (الجلسات، السجل، الأدوات، سياسة الحماية الآمنة، إلخ).

### الطبقة 1: إكمال النموذج المباشر (بدون بوابة)

-   الاختبار: `src/agents/models.profiles.live.test.ts`
-   الهدف:
    -   تعداد النماذج المكتشفة
    -   استخدام `getApiKeyForModel` لاختيار النماذج التي لديك بيانات اعتماد لها
    -   تشغيل إكمال صغير لكل نموذج (واختبارات رجعية مستهدفة حيثما لزم الأمر)
-   كيفية التمكين:
    -   `pnpm test:live` (أو `OPENCLAW_LIVE_TEST=1` إذا كنت تستدعي Vitest مباشرة)
-   اضبط `OPENCLAW_LIVE_MODELS=modern` (أو `all`، اسم مستعار لـ modern) لتشغيل هذه المجموعة فعلياً؛ وإلا ستتخطاها للحفاظ على تركيز `pnpm test:live` على اختبار دخان البوابة
-   كيفية اختيار النماذج:
    -   `OPENCLAW_LIVE_MODELS=modern` لتشغيل قائمة السماح الحديثة (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_MODELS=all` هو اسم مستعار لقائمة السماح الحديثة
    -   أو `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (قائمة سماح مفصولة بفواصل)
-   كيفية اختيار مقدمي الخدمة:
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (قائمة سماح مفصولة بفواصل)
-   من أين تأتي المفاتيح:
    -   افتراضياً: مخزن الملف الشخصي وبدائل البيئة
    -   اضبط `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` لفرض **مخزن الملف الشخصي** فقط
-   سبب وجود هذا:
    -   يفصل بين "API مقدم الخدمة معطل / المفتاح غير صالح" و "خط أنابيب وكيل البوابة معطل"
    -   يحتوي على اختبارات رجعية صغيرة ومعزولة (مثال: إعادة تشغيل استدلال OpenAI Responses/Codex Responses + تدفقات استدعاء الأدوات)

### الطبقة 2: دخان البوابة + وكيل التطوير (ما يفعله "@openclaw" فعلاً)

-   الاختبار: `src/gateway/gateway-models.profiles.live.test.ts`
-   الهدف:
    -   تشغيل بوابة داخلية
    -   إنشاء/تعديل جلسة `agent:dev:*` (تجاوز النموذج لكل تشغيل)
    -   تكرار النماذج ذات المفاتيح والتأكد من:
        -   استجابة "ذات معنى" (بدون أدوات)
        -   عمل استدعاء أداة حقيقي (فحص القراءة)
        -   فحوصات أدوات إضافية اختيارية (فحص التنفيذ+القراءة)
        -   استمرار عمل مسارات الانحدار لـ OpenAI (استدعاء-أداة-فقط → متابعة)
-   تفاصيل الفحص (حتى تتمكن من شرح الأعطال بسرعة):
    -   فحص `read`: يكتب الاختبار ملف nonce في مساحة العمل ويطلب من الوكيل `قراءته` وترديد nonce مرة أخرى.
    -   فحص `exec+read`: يطلب الاختبار من الوكيل `تنفيذ` كتابة nonce في ملف مؤقت، ثم `قراءته` مرة أخرى.
    -   فحص الصورة: يرفق الاختبار صورة PNG مُنشأة (قطة + كود عشوائي) ويتوقع من النموذج إرجاع `cat `.
    -   مرجع التنفيذ: `src/gateway/gateway-models.profiles.live.test.ts` و `src/gateway/live-image-probe.ts`.
-   كيفية التمكين:
    -   `pnpm test:live` (أو `OPENCLAW_LIVE_TEST=1` إذا كنت تستدعي Vitest مباشرة)
-   كيفية اختيار النماذج:
    -   الافتراضي: قائمة السماح الحديثة (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` هو اسم مستعار لقائمة السماح الحديثة
    -   أو اضبط `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (أو قائمة مفصولة بفواصل) للتضييق
-   كيفية اختيار مقدمي الخدمة (تجنب "كل شيء من OpenRouter"):
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (قائمة سماح مفصولة بفواصل)
-   فحوصات الأداة والصورة دائمًا مفعلة في هذا الاختبار الحي:
    -   فحص `read` + فحص `exec+read` (إجهاد الأداة)
    -   يعمل فحص الصورة عندما يعلن النموذج عن دعم إدخال الصور
    -   التدفق (عالي المستوى):
        -   يولد الاختبار صورة PNG صغيرة تحتوي على "CAT" + كود عشوائي (`src/gateway/live-image-probe.ts`)
        -   يرسلها عبر `agent` `attachments: [{ mimeType: "image/png", content: "" }]`
        -   تحلل البوابة المرفقات إلى `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
        -   يوجه الوكيل المضمن رسالة مستخدم متعددة الوسائط إلى النموذج
        -   التأكيد: تحتوي الرد على `cat` + الكود (تحمّل OCR: يُسمح بأخطاء طفيفة)

نصيحة: لمعرفة ما يمكنك اختباره على جهازك (ومعرفات `provider/model` الدقيقة)، شغِّل:

```bash
openclaw models list
openclaw models list --json
```

## الاختبارات الحية: دخان رمز إعداد Anthropic

-   الاختبار: `src/agents/anthropic.setup-token.live.test.ts`
-   الهدف: التحقق من أن رمز إعداد Claude Code CLI (أو ملف شخصي لرمز إعداد تم لصقه) يمكنه إكمال مطالبة Anthropic.
-   التمكين:
    -   `pnpm test:live` (أو `OPENCLAW_LIVE_TEST=1` إذا كنت تستدعي Vitest مباشرة)
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   مصادر الرمز (اختر واحداً):
    -   الملف الشخصي: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   الرمز الخام: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   تجاوز النموذج (اختياري):
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

مثال للإعداد:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## الاختبارات الحية: دخان الواجهة الخلفية لـ CLI (Claude Code CLI أو CLIs محلية أخرى)

-   الاختبار: `src/gateway/gateway-cli-backend.live.test.ts`
-   الهدف: التحقق من خط أنابيب البوابة + الوكيل باستخدام واجهة خلفية محلية لـ CLI، دون لمس التكوين الافتراضي الخاص بك.
-   التمكين:
    -   `pnpm test:live` (أو `OPENCLAW_LIVE_TEST=1` إذا كنت تستدعي Vitest مباشرة)
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   الإعدادات الافتراضية:
    -   النموذج: `claude-cli/claude-sonnet-4-6`
    -   الأمر: `claude`
    -   الوسائط: `["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   تجاوزات (اختيارية):
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` لإرسال مرفق صورة حقيقية (يتم حقق المسارات في المطالبة).
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` لتمرير مسارات ملفات الصور كوسائط CLI بدلاً من الحقن في المطالبة.
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (أو `"list"`) للتحكم في كيفية تمرير وسائط الصور عند تعيين `IMAGE_ARG`.
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` لإرسال منعطف ثاني والتحقق من تدفق الاستئناف.
-   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` للحفاظ على تمكين تكوين MCP لـ Claude Code CLI (الإعداد الافتراضي يعطل تكوين MCP بملف فارغ مؤقت).

مثال:

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### وصفات الاختبار الحي الموصى بها

قوائم السماح الضيقة والصريحة هي الأسرع والأقل تقلباً:

-   نموذج واحد، مباشر (بدون بوابة):
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   نموذج واحد، دخان البوابة:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   استدعاء الأدوات عبر عدة مقدمي خدمة:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   التركيز على Google (مفتاح Gemini API + Antigravity):
    -   Gemini (مفتاح API): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

ملاحظات:

-   `google/...` يستخدم Gemini API (مفتاح API).
-   `google-antigravity/...` يستخدم جسر OAuth لـ Antigravity (نقطة نهاية وكيل على طراز Cloud Code Assist).
-   `google-gemini-cli/...` يستخدم Gemini CLI المحلي على جهازك (مصادقة منفصلة + غرائب الأدوات).
-   Gemini API مقابل Gemini CLI:
    -   API: يستدعي OpenClaw Gemini API المستضافة من Google عبر HTTP (مصادقة مفتاح API / الملف الشخصي)؛ هذا ما يعنيه معظم المستخدمين بـ "Gemini".
    -   CLI: يقوم OpenClaw بإنشاء shell للثنائي المحلي `gemini`؛ لديه مصادقته الخاصة ويمكن أن يتصرف بشكل مختلف (دعم البث/الأدوات/انحراف الإصدار).

## الاختبارات الحية: مصفوفة النماذج (ما نغطيه)

لا توجد "قائمة نماذج CI" ثابتة (الاختبارات الحية اختيارية)، ولكن هذه هي النماذج **الموصى بها** للتغطية بانتظام على جهاز مطور بالمفاتيح.

### مجموعة الدخان الحديثة (استدعاء الأدوات + الصور)

هذا هو تشغيل "النماذج الشائعة" الذي نتوقع أن يستمر في العمل:

-   OpenAI (غير Codex): `openai/gpt-5.2` (اختياري: `openai/gpt-5.1`)
-   OpenAI Codex: `openai-codex/gpt-5.4`
-   Anthropic: `anthropic/claude-opus-4-6` (أو `anthropic/claude-sonnet-4-5`)
-   Google (Gemini API): `google/gemini-3-pro-preview` و `google/gemini-3-flash-preview` (تجنب نماذج Gemini 2.x الأقدم)
-   Google (Antigravity): `google-antigravity/claude-opus-4-6-thinking` و `google-antigravity/gemini-3-flash`
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

شغِّل دخان البوابة مع الأدوات + الصور: `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### الأساسي: استدعاء الأدوات (قراءة + تنفيذ اختياري)

اختر واحداً على الأقل من كل عائلة مقدم خدمة:

-   OpenAI: `openai/gpt-5.2` (أو `openai/gpt-5-mini`)
-   Anthropic: `anthropic/claude-opus-4-6` (أو `anthropic/claude-sonnet-4-5`)
-   Google: `google/gemini-3-flash-preview` (أو `google/gemini-3-pro-preview`)
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

تغطية إضافية اختيارية (جيدة أن تكون موجودة):

-   xAI: `xai/grok-4` (أو أحدث نموذج متاح)
-   Mistral: `mistral/`… (اختر نموذجاً واحداً قادراً على "الأدوات" لديك مفعلاً)
-   Cerebras: `cerebras/`… (إذا كان لديك وصول)
-   LM Studio: `lmstudio/`… (محلي؛ يعتمد استدعاء الأدوات على وضع API)

### الرؤية: إرسال الصور (مرفق → رسالة متعددة الوسائط)

أدرج نموذجاً واحداً على الأقل قادراً على معالجة الصور في `OPENCLAW_LIVE_GATEWAY_MODELS` (متغيرات Claude/Gemini/OpenAI القادرة على الرؤية، إلخ.) لممارسة فحص الصور.

### المجمعون / البوابات البديلة

إذا كان لديك مفاتيح مفعلة، فإننا ندعم الاختبار عبر:

-   OpenRouter: `openrouter/...` (مئات النماذج؛ استخدم `openclaw models scan` للعثور على مرشحين قادرين على الأدوات+الصور)
-   OpenCode Zen: `opencode/...` (المصادقة عبر `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

مزيد من مقدمي الخدمة الذين يمكنك تضمينهم في المصفوفة الحية (إذا كان لديك بيانات اعتماد/تكوين):

-   المدمجون: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
-   عبر `models.providers` (نقاط نهاية مخصصة): `minimax` (سحابة/API)، بالإضافة إلى أي وكيل متوافق مع OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, إلخ.)

نصيحة: لا تحاول ترميز "كل النماذج" في المستندات. القائمة الموثوقة هي ما يُرجعه `discoverModels(...)` على جهازك + أي مفاتيح متاحة.

## بيانات الاعتماد (لا تُلتزم أبداً)

تكتشف الاختبارات الحية بيانات الاعتماد بنفس طريقة CLI. الآثار العملية:

-   إذا عمل CLI، يجب أن تجد الاختبارات الحية نفس المفاتيح.
-   إذا قال اختبار حي "لا توجد بيانات اعتماد"، فقم بتصحيحه بنفس الطريقة التي تصحح بها `openclaw models list` / اختيار النموذج.
-   مخزن الملف الشخصي: `~/.openclaw/credentials/` (مفضل؛ ما تعنيه "مفاتيح الملف الشخصي" في الاختبارات)
-   التكوين: `~/.openclaw/openclaw.json` (أو `OPENCLAW_CONFIG_PATH`)

إذا كنت تريد الاعتماد على مفاتيح البيئة (مثل المصدرة في `~/.profile` الخاص بك)، شغِّل الاختبارات المحلية بعد `source ~/.profile`، أو استخدم عدّادات Docker أدناه (يمكنها تحميل `~/.profile` إلى الحاوية).

## Deepgram الحي (النسخ الصوتي)

-   الاختبار: `src/media-understanding/providers/deepgram/audio.live.test.ts`
-   التمكين: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## اختبار خطة الترميز BytePlus الحي

-   الاختبار: `src/agents/byteplus.live.test.ts`
-   التمكين: `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   تجاوز النموذج اختياري: `BYTEPLUS_CODING_MODEL=ark-code-latest`

## عدّادات Docker (فحوصات اختيارية "يعمل في Linux")

تشغِّل هذه `pnpm test:live` داخل صورة Docker للمستودع، مع تحميل دليل التكوين المحلي ومساحة العمل (وتحميل `~/.profile` إذا تم تحميله):

-   النماذج المباشرة: `pnpm test:docker:live-models` (سكريبت: `scripts/test-live-models-docker.sh`)
-   البوابة + وكيل التطوير: `pnpm test:docker:live-gateway` (سكريبت: `scripts/test-live-gateway-models-docker.sh`)
-   معالج الإعداد (TTY، سقالة كاملة): `pnpm test:docker:onboard` (سكريبت: `scripts/e2e/onboard-docker.sh`)
-   شبكات البوابة (حاويتان، مصادقة WS + الصحة): `pnpm test:docker:gateway-network` (سكريبت: `scripts/e2e/gateway-network-docker.sh`)
-   الإضافات (تحميل امتداد مخصص + دخان السجل): `pnpm test:docker:plugins` (سكريبت: `scripts/e2e/plugins-docker.sh`)

دخان سلسلة لغة عادية يدوية لـ ACP (ليس CI):

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   احتفظ بهذا السكريبت لسير عمل الانحدار/التصحيح. قد تكون هناك حاجة إليه مرة أخرى للتحقق من توجيه سلسلة ACP، لذا لا تحذفه.

متغيرات بيئة مفيدة:

-   `OPENCLAW_CONFIG_DIR=...` (الافتراضي: `~/.openclaw`) محمّل إلى `/home/node/.openclaw`
-   `OPENCLAW_WORKSPACE_DIR=...` (الافتراضي: `~/.openclaw/workspace`) محمّل إلى `/home/node/.openclaw/workspace`
-   `OPENCLAW_PROFILE_FILE=...` (الافتراضي: `~/.profile`) محمّل إلى `/home/node/.profile` ويتم تحميله قبل تشغيل الاختبارات
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` لتضييق التشغيل
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` لضمان أن بيانات الاعتماد تأتي من مخزن الملف الشخصي (وليس من البيئة)

## سلامة المستندات

شغِّل فحوصات المستندات بعد تحرير المستندات: `pnpm docs:list`.

## الانحدار دون اتصال (آمن لـ CI)

هذه هي اختبارات الانحدار "لخط الأنابيب الحقيقي" بدون مقدمي خدمة حقيقيين:

-   استدعاء أدوات البوابة (OpenAI وهمي، بوابة حقيقية + حلقة وكيل): `src/gateway/gateway.test.ts` (الحالة: "تشغيل استدعاء أداة OpenAI وهمي من طرف إلى طرف عبر حلقة وكيل البوابة")
-   معالج البوابة (WS `wizard.start`/`wizard.next`، يكتب التكوين + فرض المصادقة): `src/gateway/gateway.test.ts` (الحالة: "تشغيل المعالج عبر ws وكتابة تكوين رمز المصادقة")

## تقييمات موثوقية الوكيل (المهارات)

لدينا بالفعل بعض الاختبارات الآمنة لـ CI التي تتصرف مثل "تقييمات موثوقية الوكيل":

-   استدعاء أدوات وهمي عبر بوابة حقيقية + حلقة وكيل (`src/gateway/gateway.test.ts`).
-   تدفقات معالج من طرف إلى طرف تتحقق من توصيل الجلسة وتأثيرات التكوين (`src/gateway/gateway.test.ts`).

ما لا يزال مفقوداً للمهارات (انظر [المهارات](../tools/skills.md)):

-   **اتخاذ القرار:** عند سرد المهارات في المطالبة، هل يختار الوكيل المهارة الصحيحة (أو يتجنب المهارات غير ذات الصلة)؟
-   **الامتثال:** هل يقرأ الوكيل `SKILL.md` قبل الاستخدام ويتبع الخطوات/الوسائط المطلوبة؟
-   **عقود س