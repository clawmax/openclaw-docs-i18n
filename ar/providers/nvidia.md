title: "دليل إعداد وتكوين موفر NVIDIA في OpenClaw"
description: "تعلم كيفية إعداد وتكوين موفر NVIDIA في OpenClaw. قم بالمصادقة باستخدام مفتاح API، وتشغيل أوامر CLI، واستخدام نماذج مثل Nemotron وNeMo."
keywords: ["nvidia", "openclaw", "nemotron", "nemo", "مفتاح api", "llama 3.1", "متوافق مع openai", "إعداد cli"]
---

  المزودون

  
# NVIDIA

يوفر NVIDIA واجهة برمجة تطبيقات متوافقة مع OpenAI على الرابط `https://integrate.api.nvidia.com/v1` لنماذج Nemotron وNeMo. قم بالمصادقة باستخدام مفتاح API من [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## إعداد CLI

قم بتصدير المفتاح مرة واحدة، ثم قم بتشغيل عملية الإعداد وتعيين نموذج NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

إذا كنت لا تزال تمرر `--token`، تذكر أنه سيتم تخزينه في سجل shell وفي ناتج أمر `ps`؛ يفضل استخدام متغير البيئة عندما يكون ذلك ممكنًا.

## مقتطف التكوين

```json
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## معرفات النماذج

-   `nvidia/llama-3.1-nemotron-70b-instruct` (الافتراضي)
-   `meta/llama-3.3-70b-instruct`
-   `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## ملاحظات

-   نقطة النهاية `/v1` المتوافقة مع OpenAI؛ استخدم مفتاح API من NVIDIA NGC.
-   يتم تمكين المزود تلقائيًا عند تعيين `NVIDIA_API_KEY`؛ يستخدم إعدادات افتراضية ثابتة (نافذة سياق 131,072 رمزًا، 4,096 رمزًا كحد أقصى).

[Mistral](./mistral.md)[Ollama](./ollama.md)

---