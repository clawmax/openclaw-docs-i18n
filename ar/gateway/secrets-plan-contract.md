title: "عقد خطة تطبيق أسرار OpenClaw وقواعد التحقق"
description: "تعرف على العقد الصارم لخطط تطبيق أسرار openclaw. افهم التحقق من الهدف، المسارات المدعومة، وكيفية إصلاح الأخطاء قبل كتابة التكوين."
keywords: ["تطبيق الأسرار", "عقد الخطة", "إدارة الأسرار", "التحقق من التكوين", "أسرار openclaw", "مسار الهدف", "سطح بيانات الاعتماد", "تجربة جافة"]
---

  التكوين والعمليات

  
# عقد خطة تطبيق الأسرار

تحدد هذه الصفحة العقد الصارم الذي يفرضه أمر `openclaw secrets apply`. إذا لم يتطابق الهدف مع هذه القواعد، يفشل التطبيق قبل تعديل التكوين.

## شكل ملف الخطة

يتوقع أمر `openclaw secrets apply --from <plan.json>` مصفوفة `targets` من الأهداف المخططة:

```json
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
    {
      type: "auth-profiles.api_key.key",
      path: "profiles.openai:default.key",
      pathSegments: ["profiles", "openai:default", "key"],
      agentId: "main",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## نطاق الهدف المدعوم

يتم قبول الأهداف المخططة للمسارات المدعومة لبيانات الاعتماد في:

-   [سطح بيانات الاعتماد SecretRef](../reference/secretref-credential-surface.md)

## سلوك نوع الهدف

القاعدة العامة:

-   يجب أن يكون `target.type` معروفًا ويجب أن يتطابق مع شكل `target.path` الموحد.

تبقى الأسماء المستعارة للتطابق مقبولة للخطط الحالية:

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## قواعد التحقق من المسار

يتم التحقق من كل هدف باستخدام جميع ما يلي:

-   يجب أن يكون `type` نوع هدف معروف.
-   يجب أن يكون `path` مسارًا نقطيًا غير فارغ.
-   يمكن حذف `pathSegments`. إذا تم توفيره، يجب أن يوحد ليعطي نفس المسار تمامًا مثل `path`.
-   يتم رفض المقاطع المحظورة: `__proto__`, `prototype`, `constructor`.
-   يجب أن يتطابق المسار الموحد مع شكل المسار المسجل لنوع الهدف.
-   إذا تم تعيين `providerId` أو `accountId`، فيجب أن يتطابق مع المعرف المشفر في المسار.
-   تتطلب أهداف `auth-profiles.json` وجود `agentId`.
-   عند إنشاء تعيين جديد في `auth-profiles.json`، قم بتضمين `authProfileProvider`.

## سلوك الفشل

إذا فشل التحقق من هدف، يخرج التطبيق بخطأ مثل:

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

لا يتم حفظ أي كتابات لخطة غير صالحة.

## ملاحظات نطاق وقت التشغيل والتدقيق

-   يتم تضمين إدخالات `auth-profiles.json` المعتمدة على المرجع فقط (`keyRef`/`tokenRef`) في دقة وقت التشغيل ونطاق التدقيق.
-   يكتب أمر `secrets apply` أهداف `openclaw.json` المدعومة، وأهداف `auth-profiles.json` المدعومة، وأهداف التنظيف الاختيارية.

## فحوصات المشغل

```bash
# التحقق من صحة الخطة دون كتابة
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# ثم التطبيق الفعلي
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

إذا فشل التطبيق برسالة مسار هدف غير صالح، قم بإعادة إنشاء الخطة باستخدام `openclaw secrets configure` أو أصلح مسار الهدف ليتطابق مع شكل مدعوم من المذكور أعلاه.

## وثائق ذات صلة

-   [إدارة الأسرار](./secrets.md)
-   [أمر `secrets` في CLI](../cli/secrets.md)
-   [سطح بيانات الاعتماد SecretRef](../reference/secretref-credential-surface.md)
-   [مرجع التكوين](./configuration-reference.md)

[إدارة الأسرار](./secrets.md)[مصادقة الوكيل الموثوق](./trusted-proxy-auth.md)