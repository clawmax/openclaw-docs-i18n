

  الأتمتة

  
# مراقبة المصادقة

يكشف OpenClaw عن حالة صحة انتهاء صلاحية OAuth عبر `openclaw models status`. استخدم ذلك للأتمتة وإرسال التنبيهات؛ النصوص البرمجية هي إضافات اختيارية لسير عمل الهاتف.

## الطريقة المفضلة: فحص CLI (قابل للنقل)

```bash
openclaw models status --check
```

أكواد الخروج:

-   `0`: OK
-   `1`: بيانات الاعتماد منتهية الصلاحية أو مفقودة
-   `2`: على وشك الانتهاء (خلال 24 ساعة)

يعمل هذا في cron/systemd ولا يتطلب نصوصًا برمجية إضافية.

## نصوص برمجية اختيارية (عمليات التشغيل / سير عمل الهاتف)

توجد هذه النصوص تحت `scripts/` وهي **اختيارية**. تفترض وصول SSH إلى مضيف البوابة ومضبوطة لـ systemd + Termux.

-   `scripts/claude-auth-status.sh` تستخدم الآن `openclaw models status --json` كمصدر للحقيقة (مع التراجع إلى قراءة الملفات مباشرة إذا كان CLI غير متاح)، لذا احتفظ بـ `openclaw` على `PATH` للمؤقتات.
-   `scripts/auth-monitor.sh`: هدف مؤقت cron/systemd؛ يرسل تنبيهات (ntfy أو هاتف).
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}`: مؤقت مستخدم systemd.
-   `scripts/claude-auth-status.sh`: مدقق مصادقة Claude Code + OpenClaw (كامل/json/مبسط).
-   `scripts/mobile-reauth.sh`: تدفق إعادة مصادقة موجه عبر SSH.
-   `scripts/termux-quick-auth.sh`: حالة عنصر واجهة بنقرة واحدة + فتح رابط المصادقة.
-   `scripts/termux-auth-widget.sh`: تدفق عنصر واجهة موجه كامل.
-   `scripts/termux-sync-widget.sh`: مزامنة بيانات اعتماد Claude Code → OpenClaw.

إذا كنت لا تحتاج إلى أتمتة الهاتف أو مؤقتات systemd، فتخطى هذه النصوص البرمجية.

[الاستطلاعات](./poll.md)[العقد](../nodes.md)