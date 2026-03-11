

  مزودو الخدمة

  
# وكيل API لـ Claude Max

**claude-max-api-proxy** هي أداة مجتمعية تعرض اشتراكك في Claude Max/Pro كنقطة نهاية API متوافقة مع OpenAI. هذا يسمح لك باستخدام اشتراكك مع أي أداة تدعم تنسيق OpenAI API.

> **⚠️** هذا المسار هو للتكامل التقني فقط. قامت Anthropic بحظر بعض استخدامات الاشتراك خارج Claude Code في الماضي. يجب أن تقرر بنفسك ما إذا كنت ستستخدمه وتتحقق من شروط Anthropic الحالية قبل الاعتماد عليه.

## لماذا تستخدم هذا؟

| النهج | التكلفة | الأفضل لـ |
| --- | --- | --- |
| Anthropic API | الدفع حسب الرمز (~15/مليون إدخال، 75/مليون إخراج لـ Opus) | التطبيقات الإنتاجية، الحجم الكبير |
| اشتراك Claude Max | 200 دولار/شهر ثابت | الاستخدام الشخصي، التطوير، استخدام غير محدود |

إذا كان لديك اشتراك في Claude Max وتريد استخدامه مع أدوات متوافقة مع OpenAI، فقد يقلل هذا الوكيل التكلفة لبعض سير العمل. تظل مفاتيح API المسار الأوضح من حيث السياسة للاستخدام الإنتاجي.

## كيف يعمل

```
تطبيقك → claude-max-api-proxy → Claude Code CLI → Anthropic (عبر الاشتراك)
     (تنسيق OpenAI)              (يحول التنسيق)      (يستخدم تسجيل دخولك)
```

يقوم الوكيل بـ:

1.  قبول طلبات بتنسيق OpenAI على `http://localhost:3456/v1/chat/completions`
2.  تحويلها إلى أوامر Claude Code CLI
3.  إرجاع الردود بتنسيق OpenAI (يدعم البث المباشر)

## التثبيت

```bash
# يتطلب Node.js 20+ و Claude Code CLI
npm install -g claude-max-api-proxy

# التحقق من مصادقة Claude CLI
claude --version
```

## الاستخدام

### بدء تشغيل الخادم

```
claude-max-api
# يعمل الخادم على http://localhost:3456
```

### اختبره

```bash
# فحص الحالة
curl http://localhost:3456/health

# عرض النماذج
curl http://localhost:3456/v1/models

# إكمال محادثة
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### مع OpenClaw

يمكنك توجيه OpenClaw إلى الوكيل كنقطة نهاية مخصصة متوافقة مع OpenAI:

```json
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## النماذج المتاحة

| معرف النموذج | يُعيين إلى |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## التشغيل التلقائي على macOS

أنشئ LaunchAgent لتشغيل الوكيل تلقائيًا:

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## روابط

-   **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **Issues:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## ملاحظات

-   هذه **أداة مجتمعية**، وليست مدعومة رسميًا من Anthropic أو OpenClaw
-   تتطلب اشتراكًا نشطًا في Claude Max/Pro مع مصادقة Claude Code CLI
-   يعمل الوكيل محليًا ولا يرسل البيانات إلى أي خوادم تابعة لأطراف ثالثة
-   الردود المباشرة مدعومة بالكامل

## انظر أيضًا

-   [مزود Anthropic](./anthropic.md) - تكامل OpenClaw الأصلي مع إعداد رمز Claude أو مفاتيح API
-   [مزود OpenAI](./openai.md) - لاشتراكات OpenAI/Codex

[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)

---