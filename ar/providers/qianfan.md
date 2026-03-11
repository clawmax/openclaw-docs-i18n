

  الموفرون

  
# Qianfan

Qianfan هي منصة MaaS من Baidu، توفر **واجهة برمجة تطبيقات موحدة** توجه الطلبات إلى العديد من النماذج خلف نقطة نهاية واحدة ومفتاح API واحد. وهي متوافقة مع OpenAI، لذا تعمل معظم أدوات تطوير البرمجيات الخاصة بـ OpenAI عن طريق تبديل عنوان URL الأساسي.

## المتطلبات الأساسية

1.  حساب Baidu Cloud مع وصول إلى واجهة برمجة تطبيقات Qianfan
2.  مفتاح API من وحدة تحكم Qianfan
3.  OpenClaw مثبت على نظامك

## الحصول على مفتاح API الخاص بك

1.  قم بزيارة [وحدة تحكم Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  أنشئ تطبيقًا جديدًا أو حدد تطبيقًا موجودًا
3.  قم بإنشاء مفتاح API (التنسيق: `bce-v3/ALTAK-...`)
4.  انسخ مفتاح API لاستخدامه مع OpenClaw

## إعداد واجهة سطر الأوامر

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## الوثائق ذات الصلة

-   [تكوين OpenClaw](../gateway/configuration.md)
-   [موفرو النماذج](../concepts/model-providers.md)
-   [إعداد الوكيل](../concepts/agent.md)
-   [وثائق واجهة برمجة تطبيقات Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---