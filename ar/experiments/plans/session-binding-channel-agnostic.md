

  التجارب

  
# خطة ربط الجلسة المستقلة عن القناة

## نظرة عامة

يحدد هذا المستند نموذج ربط الجلسة طويل المدى المستقل عن القناة والنطاق الملموس لتكرار التنفيذ التالي. الهدف:

-   جعل توجيه جلسة الوكيل الفرعي المقيدة قدرة أساسية
-   الحفاظ على السلوك المحدد للقناة في المحولات
-   تجنب التراجعات في سلوك Discord العادي

## سبب وجود هذا

السلوك الحالي يخلط بين:

-   سياسة محتوى الإكمال
-   سياسة توجيه الوجهة
-   التفاصيل المحددة لـ Discord

تسبب هذا في حالات حدية مثل:

-   تسليم مزدوج للقناة الرئيسية والفرعية تحت التشغيل المتزامن
-   استخدام رمز قديم عند إعادة استخدام مديري الربط
-   فقدان محاسبة النشاط لإرسالات webhook

## نطاق التكرار الأول

هذا التكرار محدود عمداً.

### 1\. إضافة واجهات أساسية مستقلة عن القناة

إضافة أنواع أساسية وواجهات خدمة للربط والتوجيه. الأنواع الأساسية المقترحة:

```bash
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

عقد خدمة أساسية:

```bash
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2\. إضافة موجه تسليم أساسي واحد لإكمالات الوكلاء الفرعية

إضافة مسار قرار وجهة واحد لأحداث الإكمال. عقد الموجه:

```bash
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

لهذا التكرار:

-   فقط `task_completion` يتم توجيهه عبر هذا المسار الجديد
-   المسارات الحالية لأنواع الأحداث الأخرى تبقى كما هي

### 3\. الحفاظ على Discord كمحول

يبقى Discord أول تنفيذ للمحول. مسؤوليات المحول:

-   إنشاء/إعادة استخدام محادثات الخيط
-   إرسال الرسائل المقيدة عبر webhook أو إرسال القناة
-   التحقق من حالة الخيط (مؤرشف/محذوف)
-   تعيين بيانات وصفية للمحول (هوية webhook، معرفات الخيط)

### 4\. إصلاح مشكلات الصحة المعروفة حالياً

مطلوب في هذا التكرار:

-   تحديث استخدام الرمز عند إعادة استخدام مدير ربط الخيط الحالي
-   تسجيل نشاط الصادر لإرسالات Discord القائمة على webhook
-   إيقاف الرجوع الضمني للقناة الرئيسية عند اختيار وجهة خيط مقيدة لإكمال وضع الجلسة

### 5\. الحفاظ على إعدادات الأمان الافتراضية الحالية أثناء التشغيل

لا تغيير في السلوك للمستخدمين الذين لديهم تفعيل ربط الخيط معطل. تبقى الإعدادات الافتراضية:

-   `channels.discord.threadBindings.spawnSubagentSessions = false`

النتيجة:

-   مستخدمو Discord العاديون يبقون على السلوك الحالي
-   المسار الأساسي الجديد يؤثر فقط على توجيه إكمال الجلسة المقيدة حيث يكون مفعلاً

## غير مدرج في التكرار الأول

مؤجل بشكل صريح:

-   أهداف ربط ACP (`targetKind: "acp"`)
-   محولات قنوات جديدة بخلاف Discord
-   الاستبدال العالمي لجميع مسارات التسليم (`spawn_ack`, `subagent_message` المستقبلي)
-   تغييرات على مستوى البروتوكول
-   إعادة تصميم ترحيل/إصدار المخزن لجميع استمرارية الربط

ملاحظات حول ACP:

-   تصميم الواجهة يحتفظ بمجال لـ ACP
-   تنفيذ ACP لم يبدأ في هذا التكرار

## ثوابت التوجيه

هذه الثوابت إلزامية للتكرار الأول.

-   اختيار الوجهة وتوليد المحتوى خطوات منفصلة
-   إذا حل إكمال وضع الجلسة إلى وجهة مقيدة نشطة، يجب أن يستهدف التسليم تلك الوجهة
-   لا إعادة توجيه خفية من الوجهة المقيدة إلى القناة الرئيسية
-   سلوك الرجوع يجب أن يكون صريحاً وقابلاً للملاحظة

## التوافق والنشر

هدف التوافق:

-   لا تراجع للمستخدمين الذين لديهم تفعيل ربط الخيط معطل
-   لا تغيير للقنوات غير Discord في هذا التكرار

النشر:

1.  نشر الواجهات والموجه خلف بوابات الميزات الحالية.
2.  توجيه عمليات التسليم المقيدة لوضع إكمال Discord عبر الموجه.
3.  الاحتفاظ بالمسار القديم للتدفقات غير المقيدة.
4.  التحقق عبر اختبارات مستهدفة وسجلات وقت التشغيل التجريبية.

## الاختبارات المطلوبة في التكرار الأول

تغطية الوحدة والتكامل المطلوبة:

-   تدوير رمز المدير يستخدم أحدث رمز بعد إعادة استخدام المدير
-   إرسالات webhook تقوم بتحديث الطوابع الزمنية لنشاط القناة
-   جلستان مقيدتان نشطتان في نفس قناة الطالب لا تكرران إلى القناة الرئيسية
-   إكمال تشغيل وضع الجلسة المقيدة يحل فقط إلى وجهة الخيط
-   العلم المعطل لتفعيل الربط يحافظ على السلوك القديم دون تغيير

## ملفات التنفيذ المقترحة

الأساسية:

-   `src/infra/outbound/session-binding-service.ts` (جديد)
-   `src/infra/outbound/bound-delivery-router.ts` (جديد)
-   `src/agents/subagent-announce.ts` (تكامل قرار وجهة الإكمال)

محول Discord ووقت التشغيل:

-   `src/discord/monitor/thread-bindings.manager.ts`
-   `src/discord/monitor/reply-delivery.ts`
-   `src/discord/send.outbound.ts`

الاختبارات:

-   `src/discord/monitor/provider*.test.ts`
-   `src/discord/monitor/reply-delivery.test.ts`
-   `src/agents/subagent-announce.format.test.ts`

## معايير الإنجاز للتكرار الأول

-   وجود الواجهات الأساسية وربطها لتوجيه الإكمال
-   دمج إصلاحات الصحة المذكورة أعلاه مع الاختبارات
-   عدم وجود تسليم إكمال مزدوج للقناة الرئيسية والفرعية في عمليات وضع الجلسة المقيدة
-   عدم تغيير السلوك للنشرات ذات تفعيل الربط المعطل
-   بقاء ACP مؤجلاً بشكل صريح

[خطة الإشراف على PTY والعمليات](./pty-process-supervision.md)[بحث ذاكرة مساحة العمل](../research/memory.md)