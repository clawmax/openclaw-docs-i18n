

  المزودون

  
# GitHub Copilot

## ما هو GitHub Copilot؟

GitHub Copilot هو مساعد البرمجة الذكي من GitHub. يوفر وصولاً إلى نماذج Copilot لحساب GitHub وخطتك. يمكن لـ OpenClaw استخدام Copilot كمزود نموذج بطريقتين مختلفتين.

## طريقتان لاستخدام Copilot في OpenClaw

### 1) مزود GitHub Copilot المدمج (github-copilot)

استخدم عملية تسجيل الدخول عبر الجهاز الأصلية للحصول على رمز GitHub، ثم استبدله برموز واجهة برمجة تطبيقات Copilot عند تشغيل OpenClaw. هذه هي الطريقة **الافتراضية** والأبسط لأنها لا تتطلب VS Code.

### 2) ملحق Copilot Proxy (copilot-proxy)

استخدم ملحق **Copilot Proxy** لـ VS Code كجسر محلي. يتحدث OpenClaw مع نقطة النهاية `/v1` للوكيل ويستخدم قائمة النماذج التي تقوم بتكوينها هناك. اختر هذه الطريقة عندما تكون قد قمت بالفعل بتشغيل Copilot Proxy في VS Code أو تحتاج إلى التوجيه من خلاله. يجب عليك تمكين الملحق والحفاظ على تشغيل ملحق VS Code. استخدم GitHub Copilot كمزود نموذج (`github-copilot`). يقوم أمر تسجيل الدخول بتشغيل عملية جهاز GitHub، وحفظ ملف تعريف مصادقة، وتحديث التكوين الخاص بك لاستخدام هذا الملف الشخصي.

## إعداد سطر الأوامر

```bash
openclaw models auth login-github-copilot
```

سيُطلب منك زيارة عنوان URL وإدخال رمز لمرة واحدة. ابق الطرفية مفتوحة حتى تكتمل العملية.

### أعلام اختيارية

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## تعيين نموذج افتراضي

```bash
openclaw models set github-copilot/gpt-4o
```

### مقتطف التكوين

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## ملاحظات

-   يتطلب طرفية تفاعلية (TTY)؛ قم بتشغيله مباشرة في طرفية.
-   يعتمد توفر نموذج Copilot على خطتك؛ إذا تم رفض نموذج ما، جرب معرفًا آخر (على سبيل المثال `github-copilot/gpt-4.1`).
-   يقوم تسجيل الدخول بتخزين رمز GitHub في مخزن ملف تعريف المصادقة واستبداله برمز واجهة برمجة تطبيقات Copilot عند تشغيل OpenClaw.

[Deepgram](./deepgram.md)[Hugging Face (Inference)](./huggingface.md)

---