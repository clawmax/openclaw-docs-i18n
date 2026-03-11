

  التجارب

  
# وكلاء ACP المرتبطين بالخيوط

## نظرة عامة

تحدد هذه الخطة كيفية دعم OpenClaw لوكلاء ترميز ACP في القنوات القادرة على التعامل مع الخيوط (Discord أولاً) بدورة حياة واستعادة على مستوى الإنتاج. المستند ذو الصلة:

-   [خطة إعادة هيكلة البث الموحد لوقت التشغيل](./acp-unified-streaming-refactor.md)

تجربة المستخدم المستهدفة:

-   يقوم المستخدم بإنشاء أو تركيز جلسة ACP في خيط
-   يتم توجيه رسائل المستخدم في ذلك الخيط إلى جلسة ACP المرتبطة
-   ينتج الوكيل مخرجات تتدفق مرة أخرى إلى شخصية نفس الخيط
-   يمكن أن تكون الجلسة مستمرة أو لمرة واحدة مع ضوابط تنظيف صريحة

## ملخص القرار

التوصية طويلة المدى هي هندسة معمارية هجينة:

-   نواة OpenClaw تملك مخاوط مستوى التحكم في ACP
    -   هوية الجلسة وبيانات التعريف
    -   ربط الخيوط وقرارات التوجيه
    -   ضمانات التسليم وقمع التكرارات
    -   دلالات تنظيف دورة الحياة والاستعادة
-   خلفية وقت تشغيل ACP قابلة للإضافة
    -   الخلفية الأولى هي خدمة إضافة مدعومة بـ acpx
    -   وقت التشغيل يتعامل مع نقل ACP، والطابور، والإلغاء، وإعادة الاتصال

يجب ألا يعيد OpenClaw تنفيذ آليات النقل الداخلية لـ ACP في النواة. يجب ألا يعتمد OpenClaw على مسار اعتراض يعتمد فقط على الإضافات للتوجيه.

## الهندسة المعمارية المثالية (الهدف النهائي)

معاملة ACP كمستوى تحكم من الدرجة الأولى في OpenClaw، مع محولات وقت تشغيل قابلة للإضافة. الضمانات غير القابلة للتفاوض:

-   كل ربط خيط ACP يشير إلى سجل جلسة ACP صالح
-   كل جلسة ACP لها حالة دورة حياة صريحة (`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)
-   كل تشغيل ACP له حالة تشغيل صريحة (`queued`, `running`, `completed`, `failed`, `cancelled`)
-   الإنشاء، والربط، والوضع الأولي في الطابور هي عمليات ذرية
-   إعادة محاولة الأوامر لا تنتج تأثيرات جانبية (لا توجد تشغيلات مكررة أو مخرجات Discord مكررة)
-   مخرجات قناة الخيط المرتبط هي إسقاط لأحداث تشغيل ACP، وليست تأثيرات جانبية عشوائية

نموذج الملكية طويل المدى:

-   `AcpSessionManager` هو الكاتب الوحيد والمنسق لـ ACP
-   يعيش المدير في عملية البوابة أولاً؛ يمكن نقله إلى عملية مساعدة مخصصة لاحقًا خلف نفس الواجهة
-   لكل مفتاح جلسة ACP، يمتلك المدير ممثلًا واحدًا في الذاكرة (تنفيذ أوامر متسلسل)
-   المحولات (`acpx`، خلفيات مستقبلية) هي تنفيذات للنقل/وقت التشغيل فقط

نموذج الاستمرارية طويل المدى:

-   نقل حالة مستوى التحكم في ACP إلى مخزن SQLite مخصص (وضع WAL) تحت دليل حالة OpenClaw
-   الاحتفاظ بـ `SessionEntry.acp` كإسقاط توافق أثناء الهجرة، وليس كمصدر للحقيقة
-   تخزين أحداث ACP بطريقة الإلحاق فقط لدعم إعادة التشغيل، واستعادة بعد التعطل، والتسليم الحتمي

### استراتيجية التسليم (جسر نحو الهدف النهائي)

-   جسر قصير المدى
    -   الاحتفاظ بميكانيكيات ربط الخيوط الحالية وسطح تكوين ACP الحالي
    -   إصلاح أخطاء فجوة بيانات التعريف وتوجيه أدوار ACP عبر فرع ACP أساسي واحد
    -   إضافة مفاتيح عدم إنتاج التأثيرات الجانبية وفحوصات التوجيه الفاشلة مغلقة على الفور
-   التحويل طويل المدى
    -   نقل مصدر الحقيقة لـ ACP إلى قاعدة بيانات مستوى التحكم + ممثلين
    -   جعل تسليم الخيط المرتبط يعتمد فقط على الإسقاط القائم على الأحداث
    -   إزالة سلوك الرجوع القديم الذي يعتمد على بيانات تعريف إدخال الجلسة الانتهازية

## لماذا ليس فقط إضافات بحتة

خطافات الإضافة الحالية ليست كافية للتوجيه الشامل لجلسات ACP بدون تغييرات في النواة.

-   التوجيه الوارد من ربط الخيط يحل إلى مفتاح جلسة في إرسال النواة أولاً
-   خطافات الرسائل هي من نوع أطلق وانسى ولا يمكنها إيقاف مسار الرد الرئيسي
    -   أوامر الإضافة جيدة لعمليات التحكم ولكن ليس لاستبدال تدفق الإرسال لكل دور في النواة

النتيجة:

-   وقت تشغيل ACP يمكن أن يكون قابلاً للإضافة
-   فرع توجيه ACP يجب أن يوجد في النواة

## الأساس الحالي لإعادة الاستخدام

تم تنفيذه بالفعل ويجب أن يظل معياريًا:

-   هدف ربط الخيط يدعم `subagent` و `acp`
-   تجاوز توجيه الخيط الوارد يحل عن طريق الربط قبل الإرسال العادي
-   هوية الخيط الصادرة عبر webhook في تسليم الرد
-   تدفق `/focus` و `/unfocus` مع توافق هدف ACP
-   مخزن ربط مستمر مع الاستعادة عند بدء التشغيل
-   دورة حياة فك الربط عند الأرشفة، والحذف، وفك التركيز، وإعادة التعيين، والحذف

توسع هذه الخطة هذا الأساس بدلاً من استبداله.

## الهندسة المعمارية

### نموذج الحدود

النواة (يجب أن تكون في نواة OpenClaw):

-   فرع إرسال وضع جلسة ACP في خط أنابيب الرد
-   تحكيم التسليم لتجنب التكرار بين القناة الأصلية والخيط
-   استمرارية مستوى التحكم في ACP (مع إسقاط توافق `SessionEntry.acp` أثناء الهجرة)
-   دلالات فك الربط لدورة الحياة وفصل وقت التشغيل المرتبطة بإعادة تعيين/حذف الجلسة

خلفية الإضافة (تنفيذ acpx):

-   إشراف عامل وقت تشغيل ACP
-   استدعاء عملية acpx وتحليل الأحداث
-   معالجات أوامر ACP (`/acp ...`) وتجربة مستخدم المشغل
-   الإعدادات الافتراضية والتشخيصات الخاصة بالخلفية

### نموذج ملكية وقت التشغيل

-   عملية بوابة واحدة تملك حالة تنسيق ACP
-   يتم تشغيل تنفيذ ACP في عمليات فرعية تحت الإشراف عبر خلفية acpx
-   استراتيجية العملية هي طويلة الأمد لكل مفتاح جلسة ACP نشط، وليس لكل رسالة

هذا يتجنب تكلفة بدء التشغيل في كل مطالبة ويبقي دلالات الإلغاء وإعادة الاتصال موثوقة.

### عقد وقت التشغيل الأساسي

إضافة عقد وقت تشغيل ACP أساسي حتى لا يعتمد كود التوجيه على تفاصيل CLI ويمكنه تبديل الخلفيات دون تغيير منطق الإرسال:

```bash
export type AcpRuntimePromptMode = "prompt" | "steer";

export type AcpRuntimeHandle = {
  sessionKey: string;
  backend: string;
  runtimeSessionName: string;
};

export type AcpRuntimeEvent =
  | { type: "text_delta"; stream: "output" | "thought"; text: string }
  | { type: "tool_call"; name: string; argumentsText: string }
  | { type: "done"; usage?: Record<string, number> }
  | { type: "error"; code: string; message: string; retryable?: boolean };

export interface AcpRuntime {
  ensureSession(input: {
    sessionKey: string;
    agent: string;
    mode: "persistent" | "oneshot";
    cwd?: string;
    env?: Record<string, string>;
    idempotencyKey: string;
  }): Promise<AcpRuntimeHandle>;

  submit(input: {
    handle: AcpRuntimeHandle;
    text: string;
    mode: AcpRuntimePromptMode;
    idempotencyKey: string;
  }): Promise<{ runtimeRunId: string }>;

  stream(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId: string;
    onEvent: (event: AcpRuntimeEvent) => Promise<void> | void;
    signal?: AbortSignal;
  }): Promise<void>;

  cancel(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId?: string;
    reason?: string;
    idempotencyKey: string;
  }): Promise<void>;

  close(input: { handle: AcpRuntimeHandle; reason: string; idempotencyKey: string }): Promise<void>;

  health?(): Promise<{ ok: boolean; details?: string }>;
}
```

تفاصيل التنفيذ:

-   الخلفية الأولى: `AcpxRuntime` يتم شحنها كخدمة إضافة
-   تحل النواة وقت التشغيل عبر السجل وتفشل مع خطأ مشغل صريح عندما لا يكون هناك خلفية وقت تشغيل ACP متاحة

### نموذج بيانات مستوى التحكم والاستمرارية

مصدر الحقيقة طويل المدى هو قاعدة بيانات ACP SQLite مخصصة (وضع WAL)، للتحديثات المعاملية والاستعادة الآمنة من التعطل:

-   `acp_sessions`
    -   `session_key` (pk), `backend`, `agent`, `mode`, `cwd`, `state`, `created_at`, `updated_at`, `last_error`
-   `acp_runs`
    -   `run_id` (pk), `session_key` (fk), `state`, `requester_message_id`, `idempotency_key`, `started_at`, `ended_at`, `error_code`, `error_message`
-   `acp_bindings`
    -   `binding_key` (pk), `thread_id`, `channel_id`, `account_id`, `session_key` (fk), `expires_at`, `bound_at`
-   `acp_events`
    -   `event_id` (pk), `run_id` (fk), `seq`, `kind`, `payload_json`, `created_at`
-   `acp_delivery_checkpoint`
    -   `run_id` (pk/fk), `last_event_seq`, `last_discord_message_id`, `updated_at`
-   `acp_idempotency`
    -   `scope`, `idempotency_key`, `result_json`, `created_at`, unique `(scope, idempotency_key)`

```bash
export type AcpSessionMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

قواعد التخزين:

-   الاحتفاظ بـ `SessionEntry.acp` كإسقاط توافق أثناء الهجرة
-   تبقى معرفات العمليات والمقابس في الذاكرة فقط
-   دورة الحياة المستمرة وحالة التشغيل تعيش في قاعدة بيانات ACP، وليس في JSON الجلسة العام
-   إذا مات مالك وقت التشغيل، تعيد البوالة التمييه من قاعدة بيانات ACP وتستأنف من نقاط التفتيش

### التوجيه والتسليم

الوارد:

-   الاحتفاظ بالبحث الحالي لربط الخيوط كخطوة توجيه أولى
-   إذا كان الهدف المرتبط هو جلسة ACP، قم بتوجيهها إلى فرع وقت تشغيل ACP بدلاً من `getReplyFromConfig`
-   الأمر الصريح `/acp steer` يستخدم `mode: "steer"`

الصادر:

-   يتم تسوية تدفق أحداث ACP إلى أجزاء رد OpenClaw
-   يتم حل هدف التسليم عبر مسار الوجهة المرتبط الحالي
-   عندما يكون خيط مرتبط نشطًا لدور تلك الجلسة، يتم قمع اكتمال القناة الأصلية

سياسة البث:

-   بث المخرجات الجزئية مع نافذة دمج
-   فاصل زمني أدنى قابل للتكوين وأقصى بايتات للقطعة للبقاء ضمن حدود معدل Discord
-   يتم دائمًا إصدار الرسالة النهائية عند الاكتمال أو الفشل

### آلات الحالة وحدود المعاملات

آلة حالة الجلسة:

-   `creating -> idle -> running -> idle`
-   `running -> cancelling -> idle | error`
-   `idle -> closed`
-   `error -> idle | closed`

آلة حالة التشغيل:

-   `queued -> running -> completed`
-   `running -> failed | cancelled`
-   `queued -> cancelled`

حدود المعاملات المطلوبة:

-   معاملة الإنشاء
    -   إنشاء صف جلسة ACP
    -   إنشاء/تحديث صف ربط خيط ACP
    -   وضع صف تشغيل أولي في الطابور
-   معاملة الإغلاق
    -   وضع علامة على الجلسة كمغلقة
    -   حذف/انتهاء صلاحية صفوف الربط
    -   كتابة حدث إغلاق نهائي
-   معاملة الإلغاء
    -   وضع علامة على التشغيل المستهدف بالإلغاء/ملغي مع مفتاح عدم إنتاج التأثيرات الجانبية

لا يُسمح بنجاح جزئي عبر هذه الحدود.

### نموذج الممثل لكل جلسة

`AcpSessionManager` يشغل ممثلًا واحدًا لكل مفتاح جلسة ACP:

-   صندوق بريد الممثل يسلسل التأثيرات الجانبية `submit`, `cancel`, `close`, و `stream`
-   يمتلك الممثل تمييه المقبض وقت التشغيل ودورة حياة عملية محول وقت التشغيل لتلك الجلسة
-   يكتب الممثل أحداث التشغيل بالترتيب (`seq`) قبل أي تسليم إلى Discord
-   يقوم الممثل بتحديث نقاط تفتيش التسليم بعد الإرسال الصادر الناجح

هذا يزيل السباقات عبر الأدوار ويمنع المخرجات المكررة أو غير المرتبة للخيوط.

### عدم إنتاج التأثيرات الجانبية وإسقاط التسليم

يجب أن تحمل جميع إجراءات ACP الخارجية مفاتيح عدم إنتاج التأثيرات الجانبية:

-   مفتاح عدم إنتاج التأثيرات الجانبية للإنشاء
-   مفتاح عدم إنتاج التأثيرات الجانبية للمطالبة/التوجيه
-   مفتاح عدم إنتاج التأثيرات الجانبية للإلغاء
-   مفتاح عدم إنتاج التأثيرات الجانبية للإغلاق

قواعد التسليم:

-   يتم اشتقاق رسائل Discord من `acp_events` بالإضافة إلى `acp_delivery_checkpoint`
-   تستأنف إعادة المحاولة من نقطة التفتيش دون إعادة إرسال القطع التي تم تسليمها بالفعل
-   انبعاث الرد النهائي هو مرة واحدة بالضبط لكل تشغيل من منطق الإسقاط

### الاستعادة والشفاء الذاتي

عند بدء تشغيل البوابة:

-   تحميل جلسات ACP غير النهائية (`creating`, `idle`, `running`, `cancelling`, `error`)
-   إعادة إنشاء الممثلين بكسل عند أول حدث وارد أو بجدوى ضمن الحد الأقصى المُكون
-   تسوية أي تشغيلات `running` تفتقد نبضات الحياة ووضع علامة `failed` أو الاستعادة عبر المحول

عند رسالة خيط Discord واردة:

-   إذا كان الربط موجودًا ولكن جلسة ACP مفقودة، فشل مغلق مع رسالة ربط قديمة صريحة
-   اختياريًا، فك الربط التلقائي للربط القديم بعد التحقق الآمن للمشغل
-   عدم توجيه روابط ACP القديمة بصمت إلى مسار LLM العادي

### دورة الحياة والسلامة

العمليات المدعومة:

-   إلغاء التشغيل الحالي: `/acp cancel`
-   فك ربط الخيط: `/unfocus`
-   إغلاق جلسة ACP: `/acp close`
-   الإغلاق التلقائي للجلسات الخاملة حسب TTL الفعال

سياسة TTL:

-   TTL الفعال هو الحد الأدنى من
    -   TTL العام/للجلسة
    -   TTL ربط خيط Discord
    -   TTL مالك وقت تشغيل ACP

ضوابط السلامة:

-   قائمة السماح لوكلاء ACP بالاسم
-   تقييد جذور مساحة العمل لجلسات ACP
-   قائمة السماح لبيئة التشغيل للتمرير
-   الحد الأقصى للجلسات المتزامنة لـ ACP لكل حساب وعالميًا
-   تراجع إعادة التشغيل المحدود لتعطلات وقت التشغيل

## سطح التكوين

المفاتيح الأساسية:

-   `acp.enabled`
-   `acp.dispatch.enabled` (مفتاح إيقاف توجيه ACP مستقل)
-   `acp.backend` (الافتراضي `acpx`)
-   `acp.defaultAgent`
-   `acp.allowedAgents[]`
-   `acp.maxConcurrentSessions`
-   `acp.stream.coalesceIdleMs`
-   `acp.stream.maxChunkChars`
-   `acp.runtime.ttlMinutes`
-   `acp.controlPlane.store` (`sqlite` افتراضي)
-   `acp.controlPlane.storePath`
-   `acp.controlPlane.recovery.eagerActors`
-   `acp.controlPlane.recovery.reconcileRunningAfterMs`
-   `acp.controlPlane.checkpoint.flushEveryEvents`
-   `acp.controlPlane.checkpoint.flushEveryMs`
-   `acp.idempotency.ttlHours`
-   `channels.discord.threadBindings.spawnAcpSessions`

مفاتيح الإضافة/الخلفية (قسم إضافة acpx):

-   تجاوزات أمر/مسار الخلفية
-   قائمة السماح لبيئة التشغيل للخلفية
-   الإعدادات المسبقة لكل وكيل للخلفية
-   مهلات بدء/إيقاف الخلفية
-   الحد الأقصى للتشغيلات قيد التنفيذ لكل جلسة للخلفية

## مواصفات التنفيذ

### وحدات مستوى التحكم (جديدة)

إضافة وحدات مستوى تحكم ACP مخصصة في النواة:

-   `src/acp/control-plane/manager.ts`
    -   يملك ممثلي ACP، انتقالات دورة الحياة، تسلسل الأوامر
-   `src/acp/control-plane/store.ts`
    -   إدارة مخطط SQLite، المعاملات، مساعدات الاستعلام
-   `src/acp/control-plane/events.ts`
    -   تعريفات أحداث ACP مكتوبة والتسلسل
-   `src/acp/control-plane/checkpoint.ts`
    -   نقاط تفتيش تسليم مستمرة ومؤشرات إعادة التشغيل
-   `src/acp/control-plane/idempotency.ts`
    -   حجز مفتاح عدم إنتاج التأثيرات الجانبية وإعادة تشغيل الاستجابة
-   `src/acp/control-plane/recovery.ts`
    -   خطة التسوية عند بدء التشغيل وتمييه الممثل

وحدات الجسر التوافقية:

-   `src/acp/runtime/session-meta.ts`
    -   يبقى مؤقتًا للإسقاط في `SessionEntry.acp`
    -   يجب أن يتوقف عن كونه مصدر الحقيقة بعد التحويل

### الضمانات المطلوبة (يجب فرضها في الكود)

-   إنشاء جلسة ACP وربط الخيط هما عمليتان ذريتان (معاملة واحدة)
-   يوجد تشغيل نشط واحد على الأكثر لكل ممثل جلسة ACP في وقت واحد
-   `seq` الحدث يتزايد بدقة لكل تشغيل
-   نقطة تفتيش التسليم لا تتقدم أبدًا بعد آخر حدث تم تأكيده
-   إعادة تشغيل عدم إنتاج التأثيرات الجانبية تُرجع حمولة النجاح السابقة لمفاتيح الأمر المكررة
-   بيانات تعريف ACP القديمة/المفقودة لا يمكنها التوجيه إلى مسار الرد غير ACP العادي

### نقاط اللمس الأساسية

الملفات الأساسية للتغيير:

-   `src/auto-reply/reply/dispatch-from-config.ts`
    -   يستدعي فرع ACP `AcpSessionManager.submit` وتسليم الإسقاط القائم على الأحداث
    -   إزالة الرجوع المباشر لـ ACP الذي يتجاوز ضمانات مستوى التحكم
-   `src/auto-reply/reply/inbound-context.ts` (أو أقرب حدود سياق مُسوّاة)
    -   كشف مفاتيح التوجيه المُسوّاة وبذور عدم إنتاج التأثيرات الجانبية لمستوى تحكم ACP
-   `src/config/sessions/types.ts`
    -   الاحتفاظ بـ `SessionEntry.acp` كحقل توافق للإسقاط فقط
-   `src/gateway/server-methods/sessions.ts`
    -   يجب أن تستدعي إعادة التعيين/الحذف/الأرشفة مسار معاملة الإغلاق/فك الربط لمدير ACP
-   `src/infra/outbound/bound-delivery-router.ts`
    -   فرض سلوك الوجهة الفاشلة مغلقة لأدوار جلسة ACP المرتبطة
-   `src/discord/monitor/thread-bindings.ts`
    -   إضافة مساعدات التحقق من الربط القديم لـ ACP المتصلة بالبحث في مستوى التحكم
-   `src/auto-reply/reply/commands-acp.ts`
    -   توجيه الإنشاء/الإلغاء/الإغلاق/التوجيه عبر واجهات برمجة تطبيقات مدير ACP
-   `src/agents/acp-spawn.ts`
    -   التوقف عن كتابة بيانات التعريف العشوائية؛ استدعاء معاملة إنشاء مدير ACP
-   `src/plugin-sdk/**` وجسر وقت تشغيل الإضافة
    -   كشف تسجيل خلفية ACP ودلالات الصحة بشكل نظيف

الملفات الأساسية التي لم يتم استبدالها صراحةً:

-   `src/discord/monitor/message-handler.preflight.ts`
    -   الاحتفاظ بسلوك تجاوز ربط الخيوط كحل مفتاح الجلسة المعياري

### واجهة برمجة تطبيقات سجل وقت تشغيل ACP

إضافة وحدة سجل أساسية:

-   `src/acp/runtime/registry.ts`

الواجهة المطلوبة:

```bash
export type AcpRuntimeBackend = {
  id: string;
  runtime: AcpRuntime;
  healthy?: () => boolean;
};

export function registerAcpRuntimeBackend(backend: AcpRuntimeBackend): void;
export function unregisterAcpRuntimeBackend(id: string): void;
export function getAcpRuntimeBackend(id?: string): AcpRuntimeBackend | null;
export function requireAcpRuntimeBackend(id?: string): AcpRuntimeBackend;
```

السلوك:

-   `requireAcpRuntimeBackend` يرمي خطأ خلفية ACP مكتوبًا عند عدم التوفر
-   تقوم خدمة الإضافة بتسجيل الخلفية عند `start` وإلغاء التسجيل عند `stop`
-   عمليات البحث عن وقت التشغيل للقراءة فقط ومحلية للعملية

### عقد إضافة وقت تشغيل acpx (تفاصيل التنفيذ)

للخلفية الأولى للإنتاج (`extensions/acpx`)، يتم توصيل OpenClaw و acpx بعقد أمر صارم:

-   معرف الخلفية: `acpx`
-   معرف خدمة الإضافة: `acpx-runtime`
-   ترميز مقبض وقت التشغيل: `runtimeSessionName = acpx:v1:<base64url(json)>`
-   حقول الحمولة المشفرة:
    -   `name` (جلسة acpx مسماة؛ تستخدم `sessionKey` لـ OpenClaw)
    -   `agent` (أمر وكيل acpx)
    -   `cwd` (جذر مساحة عمل الجلسة)
    -   `mode` (`persistent | oneshot`)

تعيين الأمر:

-   التأكد من الجلسة:
    -   `acpx --format json --json-strict --cwd   sessions ensure --name `
-   دور المطالبة:
    -   `acpx --format json --json-strict --cwd   prompt --session  --file -`
-   الإلغاء:
    -   `acpx --format json --json-strict --cwd   cancel --session `
-   الإغلاق:
    -   `acpx --format json --json-strict --cwd   sessions close `

البث:

-   يستهلك OpenClaw أحداث ndjson من `acpx --format json --json-strict`
-   `text` => `text_delta/output`
-   `thought` => `text_delta/thought`
-   `tool_call` => `tool_call`
-   `done` => `done`
-   `error` => `error`

### ترقيع مخطط الجلسة

ترقيع `SessionEntry` في `src/config/sessions/types.ts`:

```typescript
type SessionAcpMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

الحقل المستمر:

-   `SessionEntry.acp?: SessionAcpMeta`

قواعد الهجرة:

-   المرحلة أ: الكتابة المزدوجة (إسقاط `acp` + مصدر الحقيقة SQLite لـ ACP)
-   المرحلة ب: القراءة الأساسية من SQLite لـ ACP، القراءة الاحتياطية من `SessionEntry.acp` القديم
-   المرحلة ج: أمر الهجرة يملأ صفوف ACP المفقودة من الإدخالات القديمة الصالحة
-   المرحلة د: إزالة القراءة الاحتياطية والاحتفاظ بالإسقاط اختياريًا لتجربة المستخدم فقط
-   تبقى الحقول القديمة (`cliSessionIds`, `claudeCliSessionId`) دون مساس

### عقد الخطأ

إضافة رموز خطأ ACP ثابتة ورسائل موجهة للمستخدم:

-   `ACP_BACKEND_MISSING`
    -   الرسالة: `خلفية وقت تشغيل ACP غير مُكونة. قم بتثبيت وتمكين إضافة وقت تشغيل acpx.`
-   `ACP_BACKEND_UNAVAILABLE`
    -   الرسالة: `خلفية وقت تشغيل ACP غير متاحة حاليًا. حاول مرة أخرى بعد قليل.`
-   `ACP_SESSION_INIT_FAILED`
    -   الرسالة: `تعذر تهيئة وقت تشغيل جلسة ACP.`
-   `ACP_TURN_FAILED`
    -   الرسالة: `فشل دور ACP قبل الاكتمال.`

القواعد:

-   إرجاع رسالة آمنة وقابلة للتنفيذ للمستخدم داخل الخيط
-   تسجيل خطأ الخلفية/النظام التفصيلي فقط في سجلات وقت التشغيل
-   عدم الرجوع بصمت إلى مسار LLM العادي عندما تم اختيار توجيه ACP صراحةً

### تحكيم التسليم المكرر

قاعدة توجيه واحدة لأدوار ACP المرتبطة:

-   إذا كان هناك ربط خيط نشط موجود لجلسة ACP المستهدفة وسيقة الطالب، قم بالتسليم فقط إلى ذلك الخيط المرتبط
-   عدم الإرسال أيضًا إلى القناة الأصلية لنفس الدور
-   إذا كان اختيار وجهة الربط غامضًا، فشل مغلق مع خطأ صريح (لا رجوع ضمني إلى الأصل)
-   إذا لم يكن هناك ربط نشط موجود، استخدم سلوك وجهة الجلسة العادي

### إمكانية المراقبة والجاهزية التشغيلية

المقاييس المطلوبة:

-   عدد نجاح/فشل إنشاء ACP حسب الخلفية ورمز الخطأ
-   النسب المئوية لوقت استجابة تشغيل ACP (وقت انتظار الطابور، وقت دور وقت التشغيل، وقت إسقاط التسليم)
-   عدد إعادة تشغيل ممثل ACP وسبب إعادة التشغيل
-   عدد اكتشاف الربط القديم
-   معدل ضرب إعادة تشغيل عدم إنتاج التأثيرات الجانبية
-   عدادات إعادة محاولة تسليم Discord وحدود المعدل

السجلات المطلوبة:

-   سجلات منظمة مفتاحها `sessionKey`, `runId`, `backend`, `threadId`, `idempotencyKey`
-   سجلات انتقال حالة صريحة لآلات حالة الجلسة والتشغيل
-   سجلات أمر المحول مع وسائط آمنة من الحذف وملخص الخروج

التشخيصات المطلوبة:

-   `/acp sessions` يتضمن الحالة، التشغيل النشط، آخر خطأ، وحالة الربط
-   `/acp doctor` (أو ما يعادله) يتحقق من تسجيل الخلفية، صحة المخزن، والروابط القديمة

### أولوية التكوين والقيم الفعالة

أولوية تمكين ACP:

-   تجاوز الحساب: `channels.discord.accounts..threadBindings.spawnAcpSessions`
-   تجاوز القناة: `channels.discord.threadBindings.spawnAcpSessions`
-   بوابة ACP العالمية: `acp.enabled`
-   بوابة الإرسال: `acp.dispatch.enabled`
-   توفر الخلفية: خلفية مسجلة لـ `acp.backend`

سلوك التمكين التلقائي:

-   عندما يتم تكوين ACP (`acp.enabled=true`, `acp.dispatch.enabled=true`, أو `acp.backend=acpx`)، يضع التمكين التلقائي للإضافة `plugins.entries.acpx.enabled=true` ما لم يتم منعه أو تعطيله صراحةً

القيمة الفعالة لـ TTL:

-   `min(session ttl, discord thread binding ttl, acp runtime ttl)`

### خريطة الاختبار

اختبارات الوحدة:

-   `src/acp/runtime/registry.test.ts` (جديد)
-   `src/auto-reply/reply/dispatch-from-config.acp.test.ts` (جديد)
-   `src/infra/outbound/bound-delivery-router.test.ts` (توسيع حالات فشل ACP مغلقة)
-   `src/config/sessions/types.test.ts` أو أقرب اختبارات مخزن الجلسات (استمرارية بيانات تعريف ACP)

اختبارات التكامل:

-   `src/discord/monitor/reply-delivery.test.ts` (سلوك وجهة تسليم ACP المرتبط)
-   `src/discord/monitor/message-handler.preflight*.test.ts` (استمرارية توجيه مفتاح جلسة ACP المرتبط)
-   اختبارات وقت تشغيل إضافة acpx في حزمة الخلفية (تسجيل/بدء/إيقاف الخدمة + تسوية الأحداث)

اختبارات e2e للبوابة:

-   `src/gateway/server.sessions.gateway-server-sessions-a.e2e.test.ts` (توسيع تغطية دورة الحياة إعادة تعيين/حذف ACP)
-   e2e لدورة الخيط لـ ACP للإنشاء، الرسالة، البث، الإلغاء، فك التركيز، استعادة إعادة التشغيل

### حارس النشر

إضافة مفتاح إيقاف إرسال ACP مستقل:

-   `acp.dispatch.enabled` الافتراضي `false` للإصدار الأول
-   عند التعطيل:
    -   قد لا تزال أوامر التحكم في إنشاء/تركيز ACP تربط الجلسات
    -   مسار إرسال ACP لا يتم تفعيله
    -   يتلقى المستخدم