

  طرق تثبيت أخرى

  
# بودمان

شغل بوابة OpenClaw في حاوية **بدون صلاحيات جذر** لـ Podman. تستخدم نفس الصورة المستخدمة مع Docker (يتم بناؤها من ملف [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) في المستودع).

## المتطلبات

-   Podman (بدون صلاحيات جذر)
-   صلاحيات Sudo للإعداد لمرة واحدة (إنشاء مستخدم، بناء الصورة)

## البدء السريع

**1\. الإعداد لمرة واحدة** (من جذر المستودع؛ ينشئ مستخدمًا، يبني الصورة، يثبت سيناريو التشغيل):

```
./setup-podman.sh
```

هذا أيضًا ينشئ ملفًا أساسيًا `~openclaw/.openclaw/openclaw.json` (يضبط `gateway.mode="local"`) حتى تتمكن البوابة من البدء دون تشغيل معالج الإعداد. بشكل افتراضي **لا** يتم تثبيت الحاوية كخدمة systemd، يمكنك بدؤها يدويًا (انظر أدناه). لإعداد شبيه بالإنتاج مع بدء تلقائي وإعادة تشغيل، قم بتثبيتها كخدمة مستخدم systemd Quadlet بدلاً من ذلك:

```bash
./setup-podman.sh --quadlet
```

(أو اضبط `OPENCLAW_PODMAN_QUADLET=1`؛ استخدم `--container` لتثبيت الحاوية وبرنامج التشغيل فقط). متغيرات بيئة اختيارية وقت البناء (اضبطها قبل تشغيل `setup-podman.sh`):

-   `OPENCLAW_DOCKER_APT_PACKAGES` — تثبيت حزم apt إضافية أثناء بناء الصورة
-   `OPENCLAW_EXTENSIONS` — تثبيت تبعيات الإضافات مسبقًا (أسماء إضافات مفصولة بمسافات، مثال: `diagnostics-otel matrix`)

**2\. بدء البوابة** (يدويًا، للاختبار السريع):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3\. معالج الإعداد** (مثال: لإضافة قنوات أو موفّرين):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

ثم افتح `http://127.0.0.1:18789/` واستخدم الرمز المميز من `~openclaw/.openclaw/.env` (أو القيمة المطبوعة بواسطة الإعداد).

## Systemd (Quadlet، اختياري)

إذا قمت بتشغيل `./setup-podman.sh --quadlet` (أو `OPENCLAW_PODMAN_QUADLET=1`)، فسيتم تثبيت وحدة [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) بحيث تعمل البوابة كخدمة مستخدم systemd لمستخدم openclaw. يتم تمكين الخدمة وبدئها في نهاية الإعداد.

-   **بدء:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
-   **إيقاف:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
-   **الحالة:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
-   **السجلات:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

يوجد ملف quadlet في `~openclaw/.config/containers/systemd/openclaw.container`. لتغيير المنافذ أو متغيرات البيئة، قم بتحرير هذا الملف (أو ملف `.env` الذي يستورده)، ثم `sudo systemctl --machine openclaw@ --user daemon-reload` وأعد تشغيل الخدمة. عند التمهيد، تبدأ الخدمة تلقائيًا إذا كان التمكين المستمر مفعلًا لـ openclaw (يقوم الإعداد بذلك عندما يكون loginctl متاحًا). لإضافة quadlet **بعد** إعداد أولي لم يستخدمه، أعد التشغيل: `./setup-podman.sh --quadlet`.

## مستخدم openclaw (غير تفاعلي)

ينشئ `setup-podman.sh` مستخدم نظام مخصص `openclaw`:

-   **الطرفية:** `nologin` — لا تسجيل دخول تفاعلي؛ يقلل من السطح المعرض للهجوم.
-   **المجلد الرئيسي:** مثال: `/home/openclaw` — يحتفظ بـ `~/.openclaw` (الإعدادات، مساحة العمل) وبرنامج التشغيل `run-openclaw-podman.sh`.
-   **Podman بدون صلاحيات جذر:** يجب أن يكون للمستخدم نطاق **subuid** و **subgid**. العديد من التوزيعات تقوم بتعيين هذه تلقائيًا عند إنشاء المستخدم. إذا طبع الإعداد تحذيرًا، أضف الأسطر إلى `/etc/subuid` و `/etc/subgid`:
    
    نسخ
    
    ```
    openclaw:100000:65536
    ```
    
    ثم ابدأ البوابة كذاك المستخدم (مثال: من cron أو systemd):
    
    نسخ
    
    ```bash
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
    ```
    
-   **الإعدادات:** فقط `openclaw` و root يمكنهما الوصول إلى `/home/openclaw/.openclaw`. لتحرير الإعدادات: استخدم واجهة التحكم بمجرد تشغيل البوابة، أو `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## البيئة والإعدادات

-   **الرمز المميز:** مخزن في `~openclaw/.openclaw/.env` كـ `OPENCLAW_GATEWAY_TOKEN`. يقوم `setup-podman.sh` و `run-openclaw-podman.sh` بتوليده إذا كان مفقودًا (يستخدم `openssl`، `python3`، أو `od`).
-   **اختياري:** في ملف `.env` ذاك يمكنك تعيين مفاتيح المزودين (مثال: `GROQ_API_KEY`، `OLLAMA_API_KEY`) ومتغيرات بيئة OpenClaw الأخرى.
-   **منافذ المضيف:** بشكل افتراضي، يقوم البرنامج النصي بتعيين `18789` (البوابة) و `18790` (الجسر). غيّر تعيين منفذ **المضيف** باستخدام `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` و `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` عند البدء.
-   **ربط البوابة:** بشكل افتراضي، يبدأ `run-openclaw-podman.sh` البوابة بـ `--bind loopback` للوصول المحلي الآمن. للتعرض على الشبكة المحلية، اضبط `OPENCLAW_GATEWAY_BIND=lan` وضبط `gateway.controlUi.allowedOrigins` (أو فعّل بشكل صريح استرجاع رأس المضيف) في `openclaw.json`.
-   **المسارات:** إعدادات المضيف ومساحة العمل الافتراضية هي `~openclaw/.openclaw` و `~openclaw/.openclaw/workspace`. غيّر مسارات المضيف المستخدمة بواسطة برنامج التشغيل باستخدام `OPENCLAW_CONFIG_DIR` و `OPENCLAW_WORKSPACE_DIR`.

## نموذج التخزين

-   **بيانات المضيف المستمرة:** يتم ربط `OPENCLAW_CONFIG_DIR` و `OPENCLAW_WORKSPACE_DIR` داخل الحاوية وتحتفظ بالحالة على المضيف.
-   **tmpfs للحاوية العازلة المؤقتة:** إذا قمت بتمكين `agents.defaults.sandbox`، فإن حاويات أداة الحاوية العازلة تربط `tmpfs` على `/tmp`، `/var/tmp`، و `/run`. هذه المسارات مدعومة بالذاكرة وتختفي مع حاوية الحاوية العازلة؛ إعداد حاوية Podman الرئيسية لا يضيف نقاط ربط tmpfs خاصة به.
-   **بؤر نمو القرص:** المسارات الرئيسية للمراقبة هي `media/`، `agents//sessions/sessions.json`، ملفات JSONL للنصوص، `cron/runs/*.jsonl`، وسجلات الملفات الدوارة تحت `/tmp/openclaw/` (أو `logging.file` الذي قمت بتكوينه).

يقوم `setup-podman.sh` الآن بتخزين صورة tar مؤقتًا في دليل خاص مؤقت ويطبع مجلد القاعدة المختار أثناء الإعداد. للتشغيلات بدون صلاحيات جذر، يقبل `TMPDIR` فقط عندما تكون تلك القاعدة آمنة للاستخدام؛ وإلا فإنه يتراجع إلى `/var/tmp`، ثم `/tmp`. يبقى ملف tar المحفوظ ملكية خاصة للمالك ويتم تحميله في `podman load` الخاص بالمستخدم المستهدف، لذا فإن المجلدات المؤقتة الخاصة للمنشئ لا تعيق الإعداد.

## أوامر مفيدة

-   **السجلات:** مع quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. مع البرنامج النصي: `sudo -u openclaw podman logs -f openclaw`
-   **إيقاف:** مع quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. مع البرنامج النصي: `sudo -u openclaw podman stop openclaw`
-   **بدء مرة أخرى:** مع quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. مع البرنامج النصي: أعد تشغيل برنامج التشغيل أو `podman start openclaw`
-   **إزالة الحاوية:** `sudo -u openclaw podman rm -f openclaw` — يتم الاحتفاظ بالإعدادات ومساحة العمل على المضيف

## استكشاف الأخطاء وإصلاحها

-   **تم رفض الإذن (EACCES) على الإعدادات أو ملفات تعريف المصادقة:** تعمل الحاوية افتراضيًا بـ `--userns=keep-id` وتعمل بنفس uid/gid كمستخدم المضيف الذي يشغل البرنامج النصي. تأكد من أن `OPENCLAW_CONFIG_DIR` و `OPENCLAW_WORKSPACE_DIR` على المضيف مملوكان لذلك المستخدم.
-   **بدء البوابة محظور (مفقود `gateway.mode=local`):** تأكد من وجود `~openclaw/.openclaw/openclaw.json` وأنه يضبط `gateway.mode="local"`. يقوم `setup-podman.sh` بإنشاء هذا الملف إذا كان مفقودًا.
-   **فشل Podman بدون صلاحيات جذر لمستخدم openclaw:** تحقق من أن `/etc/subuid` و `/etc/subgid` يحتويان على سطر لـ `openclaw` (مثال: `openclaw:100000:65536`). أضفه إذا كان مفقودًا وأعد التشغيل.
-   **اسم الحاوية قيد الاستخدام:** يستخدم برنامج التشغيل `podman run --replace`، لذا يتم استبدال الحاوية الموجودة عند البدء مرة أخرى. للتنظيف يدويًا: `podman rm -f openclaw`.
-   **البرنامج النصي غير موجود عند التشغيل كـ openclaw:** تأكد من تشغيل `setup-podman.sh` بحيث يتم نسخ `run-openclaw-podman.sh` إلى المجلد الرئيسي لـ openclaw (مثال: `/home/openclaw/run-openclaw-podman.sh`).
-   **خدمة Quadlet غير موجودة أو تفشل في البدء:** شغل `sudo systemctl --machine openclaw@ --user daemon-reload` بعد تحرير ملف `.container`. يتطلب Quadlet cgroups v2: يجب أن يظهر `podman info --format '{{.Host.CgroupsVersion}}'` القيمة `2`.

## اختياري: التشغيل كمستخدمك الخاص

لتشغيل البوابة كمستخدمك العادي (بدون مستخدم openclaw مخصص): ابنِ الصورة، وأنشئ `~/.openclaw/.env` بـ `OPENCLAW_GATEWAY_TOKEN`، وشغل الحاوية بـ `--userns=keep-id` ونقاط ربط إلى `~/.openclaw` الخاص بك. تم تصميم برنامج التشغيل لتدفق مستخدم openclaw؛ لإعداد مستخدم واحد، يمكنك بدلاً من ذلك تشغيل أمر `podman run` من البرنامج النصي يدويًا، مشيرًا بالإعدادات ومساحة العمل إلى مجلدك الرئيسي. موصى به لمعظم المستخدمين: استخدم `setup-podman.sh` وشغل كمستخدم openclaw بحيث تكون الإعدادات والعملية معزولتين.

[Docker](./docker.md)[Nix](./nix.md)