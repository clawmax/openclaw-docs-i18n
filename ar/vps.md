title: "دليل استضافة VPS ونشر OpenClaw"
description: "تعلم كيفية نشر OpenClaw على مزودي VPS مثل Railway وOracle Cloud وFly.io. أدلة الإعداد، نصائح الأمان، وضبط الأداء للوكلاء السحابية."
keywords: ["استضافة vps", "نشر openclaw", "بوابة سحابية", "railway", "oracle cloud", "ضبط systemd", "العقد", "تحسين arm"]
---

  الاستضافة والنشر

  
# استضافة VPS

يربط هذا المركز بأدلة الاستضافة/VPS المدعومة ويشرح كيفية عمل النشرات السحابية على مستوى عالٍ.

## اختر مزودًا

-   **Railway** (نقرة واحدة + إعداد عبر المتصفح): [Railway](./install/railway.md)
-   **Northflank** (نقرة واحدة + إعداد عبر المتصفح): [Northflank](./install/northflank.md)
-   **Oracle Cloud (مجاني دائمًا)**: [Oracle](./platforms/oracle.md) — $0/شهر (مجاني دائمًا، ARM؛ قد تكون السعة/التسجيل متقلبة)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + وكيل HTTPS): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/الطبقة المجانية)**: تعمل بشكل جيد أيضًا. دليل فيديو: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## كيف تعمل إعدادات السحابة

-   تعمل **البوابة على VPS** وتمتلك الحالة + مساحة العمل.
-   تتصل من حاسوبك المحمول/هاتفك عبر **واجهة التحكم** أو **Tailscale/SSH**.
-   عامل VPS كمصدر للحقيقة و**احفظ نسخة احتياطية** من الحالة + مساحة العمل.
-   الأمان الافتراضي: احتفظ بالبوابة على loopback وادخل إليها عبر نفق SSH أو Tailscale Serve. إذا ربطتها بـ `lan`/`tailnet`، اشترط `gateway.auth.token` أو `gateway.auth.password`.

الوصول عن بُعد: [بوابة عن بُعد](./gateway/remote.md)  
مركز المنصات: [المنصات](./platforms.md)

## وكيل شركة مشترك على VPS

هذا إعداد صالح عندما يكون المستخدمون ضمن حدود ثقة واحدة (على سبيل المثال فريق شركة واحد)، ويكون الوكيل مخصصًا للأعمال فقط.

-   احتفظ به على بيئة تشغيل مخصصة (VPS/VM/حاوية + مستخدم/حسابات نظام تشغيل مخصصة).
-   لا تسجل دخول تلك البيئة إلى حسابات Apple/Google الشخصية أو ملفات تعريف المتصفح/مدير كلمات المرور الشخصية.
-   إذا كان المستخدمون في حالة تنافسية مع بعضهم البعض، قم بالفصل حسب البوابة/المضيف/مستخدم نظام التشغيل.

تفاصيل نموذج الأمان: [الأمان](./gateway/security.md)

## استخدام العقد مع VPS

يمكنك الاحتفاظ بالبوابة في السحابة وإقران **عقد** على أجهزتك المحلية (Mac/iOS/Android/بدون واجهة). توفر العقد شاشة محلية/كاميرا/لوحة وامكانيات `system.run` بينما تبقى البوابة في السحابة. المستندات: [العقد](./nodes.md)، [واجهة سطر أوامر العقد](./cli/nodes.md)

## ضبط بدء التشغيل لأجهزة VM الصغيرة ومضيفي ARM

إذا بدت أوامر CLI بطيئة على أجهزة VM منخفضة الطاقة (أو مضيفي ARM)، فعّل ذاكرة التخزين المؤقتة لتجميع وحدات Node:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` يحسن أوقات بدء تشغيل الأوامر المتكررة.
-   `OPENCLAW_NO_RESPAWN=1` يتجنب النفقات الإضافية لبدء التشغيل من مسار إعادة التشغيل الذاتي.
-   تشغيل الأمر الأول يسخن ذاكرة التخزين المؤقتة؛ عمليات التشغيل اللاحقة أسرع.
-   للتفاصيل الخاصة بـ Raspberry Pi، انظر [Raspberry Pi](./platforms/raspberry-pi.md).

### قائمة مراجعة ضبط systemd (اختياري)

لمضيفي VM الذين يستخدمون `systemd`، ضع في الاعتبار:

-   أضف متغيرات بيئة للخدمة لمسار بدء تشغيل مستقر:
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   حافظ على سلوك إعادة التشغيل صريحًا:
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   فضّل الأقراص المدعومة بـ SSD لمسارات الحالة/ذاكرة التخزين المؤقتة لتقليل عقوبات بدء التشغيل البارد لعمليات الإدخال/الإخراج العشوائية.

مثال:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

كيف تساعد سياسات `Restart=` في الاسترداد الآلي: [systemd يمكنه أتمتة استرداد الخدمة](https://www.redhat.com/en/blog/systemd-automate-recovery).

[إلغاء التثبيت](./install/uninstall.md)[Fly.io](./install/fly.md)