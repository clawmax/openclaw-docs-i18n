

  الإضافات

  
# إضافة المكالمات الصوتية

مكالمات صوتية لـ OpenClaw عبر إضافة. تدعم الإشعارات الصادرة والمحادثات متعددة الأدوار مع سياسات واردة. مزودون مدعومون حاليًا:

-   `twilio` (برنامج الصوت + تدفقات الوسائط)
-   `telnyx` (التحكم في المكالمات الإصدار 2)
-   `plivo` (واجهة برمجة تطبيقات الصوت + نقل XML + GetInput للكلام)
-   `mock` (للتنمية/بدون شبكة)

النموذج الذهني السريع:

-   قم بتثبيت الإضافة
-   أعد تشغيل البوابة (Gateway)
-   قم بالتكوين تحت `plugins.entries.voice-call.config`
-   استخدم `openclaw voicecall ...` أو أداة `voice_call`

## أين تعمل (محلي مقابل بعيد)

تعمل إضافة المكالمات الصوتية **داخل عملية البوابة (Gateway)**. إذا كنت تستخدم بوابة بعيدة، قم بتثبيت/تكوين الإضافة على **الجهاز الذي يشغل البوابة**، ثم أعد تشغيل البوابة لتحميلها.

## التثبيت

### الخيار أ: التثبيت من npm (موصى به)

```bash
openclaw plugins install @openclaw/voice-call
```

أعد تشغيل البوابة بعد ذلك.

### الخيار ب: التثبيت من مجلد محلي (للتنمية، بدون نسخ)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

أعد تشغيل البوابة بعد ذلك.

## التكوين

قم بتعيين التكوين تحت `plugins.entries.voice-call.config`:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // أو "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // مفتاح ويبهوك Telnyx العام من لوحة تحكم Telnyx Mission Control
            // (سلسلة Base64؛ يمكن أيضًا تعيينها عبر TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // خادم ويبهوك
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // أمان ويبهوك (موصى به للأنفاق والوكلاء)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // التعرض العام (اختر واحدًا)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

ملاحظات:

-   يتطلب Twilio/Telnyx عنوان URL للويبهوك **يمكن الوصول إليه بشكل عام**.
-   يتطلب Plivo عنوان URL للويبهوك **يمكن الوصول إليه بشكل عام**.
-   `mock` هو مزود للتنمية المحلية (بدون مكالمات شبكية).
-   يتطلب Telnyx `telnyx.publicKey` (أو `TELNYX_PUBLIC_KEY`) ما لم يكن `skipSignatureVerification` صحيحًا.
-   `skipSignatureVerification` مخصص للاختبار المحلي فقط.
-   إذا كنت تستخدم الطبقة المجانية من ngrok، عيّن `publicUrl` إلى عنوان ngrok الدقيق؛ يتم دائمًا فرض التحقق من التوقيع.
-   يسمح `tunnel.allowNgrokFreeTierLoopbackBypass: true` لويبهوكات Twilio ذات التوقيعات غير الصالحة **فقط** عندما يكون `tunnel.provider="ngrok"` و `serve.bind` هو loopback (وكيل ngrok المحلي). استخدم للتنمية المحلية فقط.
-   يمكن لعناوين URL الطبقة المجانية من ngrok أن تتغير أو تضيف سلوكًا وسيطًا؛ إذا انحرف `publicUrl`، ستفشل توقيعات Twilio. للإنتاج، يُفضل اسم نطاق ثابت أو قمع Tailscale.
-   إعدادات أمان البث المباشر الافتراضية:
    -   `streaming.preStartTimeoutMs` يغلق المقابس التي لا ترسل أبدًا إطار `start` صالحًا.
    -   `streaming.maxPendingConnections` يحدد الحد الأقصى للمقابس غير المصادق عليها قبل البدء.
    -   `streaming.maxPendingConnectionsPerIp` يحدد الحد الأقصى للمقابس غير المصادق عليها قبل البدء لكل عنوان IP مصدر.
    -   `streaming.maxConnections` يحدد الحد الأقصى للمقابس المفتوحة لتدفق الوسائط (الانتظار + النشطة).

## منظف المكالمات المتعثرة

استخدم `staleCallReaperSeconds` لإنهاء المكالمات التي لا تتلقى ويبهوك نهائيًا (على سبيل المثال، مكالمات وضع الإشعار التي لا تكتمل أبدًا). القيمة الافتراضية هي `0` (معطلة). النطاقات الموصى بها:

-   **الإنتاج:** `120`–`300` ثانية لتدفقات نمط الإشعار.
-   حافظ على هذه القيمة **أعلى من `maxDurationSeconds`** حتى تتمكن المكالمات العادية من الانتهاء. نقطة بداية جيدة هي `maxDurationSeconds + 30–60` ثانية.

مثال:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## أمان ويبهوك

عندما يكون هناك وكيل أو نفق أمام البوابة، تعيد الإضافة بناء عنوان URL العام للتحقق من التوقيع. تتحكم هذه الخيارات في رؤوس التوجيه الموثوق بها. `webhookSecurity.allowedHosts` تسمح بقوائم المضيفين المسموح بهم من رؤوس التوجيه. `webhookSecurity.trustForwardingHeaders` تثق برؤوس التوجيه دون قائمة سماح. `webhookSecurity.trustedProxyIPs` تثق برؤوس التوجيه فقط عندما يتطابق عنوان IP البعيد للطلب مع القائمة. تم تمكين حماية إعادة تشغيل ويبهوك لـ Twilio و Plivo. يتم الاعتراف بطلبات ويبهوك الصالحة المعاد تشغيلها ولكن يتم تخطيها للآثار الجانبية. تتضمن أدوار محادثة Twilio رمزًا لكل دور في ردود الاتصال ``، لذا لا يمكن لردود الاتصال الكلامية المتعثرة/المعاد تشغيلها تلبية دور نسخة حديث معلق. مثال بمضيف عام ثابت:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## تحويل النص إلى كلام للمكالمات

تستخدم المكالمات الصوتية تكوين `messages.tts` الأساسي (OpenAI أو ElevenLabs) للكلام المتدفق على المكالمات. يمكنك تجاوزه تحت تكوين الإضافة بنفس **الشكل** — حيث يدمج بعمق مع `messages.tts`.

```json
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

ملاحظات:

-   **يتم تجاهل Edge TTS للمكالمات الصوتية** (يحتاج صوت الهاتف إلى PCM؛ ناتج Edge غير موثوق).
-   يتم استخدام TTS الأساسي عندما يكون بث وسائط Twilio ممكّنًا؛ وإلا تعود المكالمات إلى الأصوات الأصلية للمزود.

### المزيد من الأمثلة

استخدم TTS الأساسي فقط (بدون تجاوز):

```json
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

تجاوز إلى ElevenLabs فقط للمكالمات (احتفظ بالافتراضي الأساسي في أماكن أخرى):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

تجاوز نموذج OpenAI فقط للمكالمات (مثال دمج عميق):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## المكالمات الواردة

السياسة الافتراضية للمكالمات الواردة هي `disabled`. لتمكين المكالمات الواردة، عيّن:

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "مرحبًا! كيف يمكنني المساعدة؟",
}
```

تستخدم الردود التلقائية نظام الوكيل. اضبط باستخدام:

-   `responseModel`
-   `responseSystemPrompt`
-   `responseTimeoutMs`

## واجهة سطر الأوامر

```bash
openclaw voicecall call --to "+15555550123" --message "مرحبًا من OpenClaw"
openclaw voicecall continue --call-id <id> --message "أي أسئلة؟"
openclaw voicecall speak --call-id <id> --message "لحظة واحدة"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## أداة الوكيل

اسم الأداة: `voice_call` الإجراءات:

-   `initiate_call` (رسالة، إلى؟، وضع؟)
-   `continue_call` (معرف المكالمة، رسالة)
-   `speak_to_user` (معرف المكالمة، رسالة)
-   `end_call` (معرف المكالمة)
-   `get_status` (معرف المكالمة)

يحتوي هذا المستودع على وثيقة مهارة مطابقة في `skills/voice-call/SKILL.md`.

## بروتوكول RPC للبوابة

-   `voicecall.initiate` (إلى؟، رسالة، وضع؟)
-   `voicecall.continue` (معرف المكالمة، رسالة)
-   `voicecall.speak` (معرف المكالمة، رسالة)
-   `voicecall.end` (معرف المكالمة)
-   `voicecall.status` (معرف المكالمة)

[إضافات المجتمع](./community.md)[إضافة Zalo الشخصية](./zalouser.md)