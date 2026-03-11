

  طرق تثبيت أخرى

  
# Docker

Docker هو **اختياري**. استخدمه فقط إذا كنت تريد بوابة معتمدة على الحاويات أو للتحقق من سير عمل Docker.

## هل Docker مناسب لي؟

-   **نعم**: إذا كنت تريد بيئة بوابة معزولة وقابلة للتجاهل أو لتشغيل OpenClaw على مضيف بدون تثبيتات محلية.
-   **لا**: إذا كنت تشغل على جهازك الخاص وتريد فقط أسرع حلقة تطوير. استخدم طريقة التثبيت العادية بدلاً من ذلك.
-   **ملاحظة حول التجزئة (Sandboxing)**: تجزئة الوكيل تستخدم Docker أيضًا، لكنها **لا** تتطلب تشغيل البوابة بالكامل داخل Docker. راجع [التجزئة (Sandboxing)](../gateway/sandboxing.md).

يغطي هذا الدليل:

-   البوابة المعتمدة على الحاويات (OpenClaw كامل داخل Docker)
-   تجزئة الوكيل لكل جلسة (بوابة المضيف + أدوات الوكيل المعزولة بـ Docker)

تفاصيل التجزئة: [التجزئة (Sandboxing)](../gateway/sandboxing.md)

## المتطلبات

-   Docker Desktop (أو Docker Engine) + Docker Compose الإصدار v2
-   ذاكرة وصول عشوائي (RAM) بسعة 2 جيجابايت على الأقل لبناء الصورة (قد يتم إيقاف `pnpm install` بسبب نفاد الذاكرة على مضيفين بسعة 1 جيجابايت مع خروج برقم 137)
-   مساحة قرص كافية للصور + السجلات
-   إذا كنت تشغل على مضيف VPS/عام، راجع [تعزيز الأمان للتعرض للشبكة](../gateway/security.md#04-network-exposure-bind--port--firewall)، خاصة سياسة جدار الحماية `DOCKER-USER` لـ Docker.

## البوابة المعتمدة على الحاويات (Docker Compose)

### البدء السريع (موصى به)

> **ℹ️** إعدادات Docker الافتراضية هنا تفترض أوضاع الربط (`lan`/`loopback`)، وليس أسماء مستعارة للمضيف (host aliases). استخدم قيم وضع الربط في `gateway.bind` (على سبيل المثال `lan` أو `loopback`)، وليس أسماء مستعارة للمضيف مثل `0.0.0.0` أو `localhost`.

 من جذر المستودع:

```
./docker-setup.sh
```

هذا البرنامج النصي:

-   يبني صورة البوابة محليًا (أو يسحب صورة بعيدة إذا تم تعيين `OPENCLAW_IMAGE`)
-   يشغل معالج الإعداد الأولي (onboarding wizard)
-   يطبع تلميحات إعداد مزود اختيارية
-   يبدأ البوابة عبر Docker Compose
-   ينشئ رمز بوابة (token) ويكتبه في `.env`

متغيرات البيئة الاختيارية:

-   `OPENCLAW_IMAGE` — استخدام صورة بعيدة بدلاً من البناء محليًا (مثل `ghcr.io/openclaw/openclaw:latest`)
-   `OPENCLAW_DOCKER_APT_PACKAGES` — تثبيت حزم apt إضافية أثناء البناء
-   `OPENCLAW_EXTENSIONS` — تثبيت تبعيات الإضافات مسبقًا في وقت البناء (أسماء إضافات مفصولة بمسافات، مثل `diagnostics-otel matrix`)
-   `OPENCLAW_EXTRA_MOUNTS` — إضافة نقاط ربط (bind mounts) إضافية من المضيف
-   `OPENCLAW_HOME_VOLUME` — الإبقاء على `/home/node` في وحدة تخزين مسماة (named volume)
-   `OPENCLAW_SANDBOX` — الاشتراك في تجهيز تجزئة بوابة Docker. فقط القيم الصحيحة الصريحة تمكنها: `1`, `true`, `yes`, `on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — تمرير وسيطة البناء (build arg) لبناء الصور المحلية (`1` يثبت Docker CLI في الصورة). `docker-setup.sh` يضبط هذا تلقائيًا عندما يكون `OPENCLAW_SANDBOX=1` للبناء المحلي.
-   `OPENCLAW_DOCKER_SOCKET` — تجاوز مسار مقبس Docker (الافتراضي: المسار `DOCKER_HOST=unix://...`، وإلا `/var/run/docker.sock`)
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — كسر الحماية: السماح بأهداف `ws://` خاصة وموثوقة على الشبكة الخاصة لمسارات عميل CLI/الإعداد الأولي (الافتراضي هو loopback فقط)
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — تعطيل علامات تعزيز أمان متصفح الحاوية `--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu` عندما تحتاج إلى توافق WebGL/3D.
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — إبقاء الإضافات ممكّنة عندما تتطلب سير عمل المتصفح ذلك (الافتراضي يعطل الإضافات في متصفح التجربة (sandbox)).
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — تعيين حد عمليات عرض Chromium؛ عيّنه على `0` لتخطي العلامة واستخدام السلوك الافتراضي لـ Chromium.

بعد انتهائه:

-   افتح `http://127.0.0.1:18789/` في متصفحك.
-   الصق الرمز (token) في واجهة التحكم (الإعدادات → الرمز).
-   هل تحتاج إلى الرابط مرة أخرى؟ شغل `docker compose run --rm openclaw-cli dashboard --no-open`.

### تمكين تجزئة الوكيل (sandbox) للبوابة المعتمدة على Docker (اختياري)

يمكن لـ `docker-setup.sh` أيضًا تجهيز `agents.defaults.sandbox.*` لنشرات Docker. مكّن باستخدام:

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

مسار مقبس مخصص (على سبيل المثال Docker بدون صلاحيات الجذر (rootless)):

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

ملاحظات:

-   البرنامج النصي يربط `docker.sock` فقط بعد اجتياز متطلبات التجربة (sandbox).
-   إذا تعذر إكمال إعداد التجربة (sandbox)، يعيد البرنامج النصي تعيين `agents.defaults.sandbox.mode` إلى `off` لتجنب تكوين تجربة (sandbox) قديم/معطل عند إعادة التشغيل.
-   إذا كان `Dockerfile.sandbox` مفقودًا، يطبع البرنامج النصي تحذيرًا ويكمل؛ ابنِ `openclaw-sandbox:bookworm-slim` باستخدام `scripts/sandbox-setup.sh` إذا لزم الأمر.
-   لقيم `OPENCLAW_IMAGE` غير المحلية، يجب أن تحتوي الصورة بالفعل على دعم Docker CLI لتنفيذ التجربة (sandbox).

### الأتمتة/التكامل المستمر (غير تفاعلي، بدون ضوضاء TTY)

للبرامج النصية والتكامل المستمر، عطّل تخصيص TTY الزائف لـ Compose باستخدام `-T`:

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

إذا كانت أتمتتك لا تصدر متغيرات جلسة Claude، فإن تركها غير معيّنة الآن يحل إلى قيم فارغة افتراضيًا في `docker-compose.yml` لتجنب تحذيرات متكررة "المتغير غير معيّن".

### ملاحظة أمان الشبكة المشتركة (CLI + البوابة)

يستخدم `openclaw-cli` `network_mode: "service:openclaw-gateway"` حتى تتمكن أوامر CLI من الوصول إلى البوابة بشكل موثوق عبر `127.0.0.1` داخل Docker. عالج هذا كحد ثقة مشترك: ربط loopback ليس عزلًا بين هاتين الحاويتين. إذا كنت تحتاج إلى فصل أقوى، شغل الأوامر من مسار شبكة حاوية/مضيف منفصل بدلاً من خدمة `openclaw-cli` المجمعة. لتقليل التأثير إذا تم اختراق عملية CLI، يلغي تكوين compose `NET_RAW`/`NET_ADMIN` ويمكّن `no-new-privileges` على `openclaw-cli`. يكتب التكوين/مساحة العمل على المضيف:

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

هل تشغل على VPS؟ راجع [Hetzner (Docker VPS)](./hetzner.md).

### استخدام صورة بعيدة (تخطي البناء المحلي)

يتم نشر الصور المبنية مسبقًا الرسمية في:

-   [حزمة سجل حاويات GitHub](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

استخدم اسم الصورة `ghcr.io/openclaw/openclaw` (وليس صور Docker Hub ذات الأسماء المشابهة). العلامات الشائعة:

-   `main` — أحدث بناء من `main`
-   `` — بناء علامات الإصدار (على سبيل المثال `2026.2.26`)
-   `latest` — أحدث علامة إصدار مستقرة

### بيانات وصفية للصورة الأساسية

تستخدم صورة Docker الرئيسية حاليًا:

-   `node:22-bookworm`

تنشر صورة docker الآن شروحات صورة الأساس OCI (sha256 هو مثال):

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=OpenClaw gateway and CLI runtime container image`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

المرجع: [شروحات صورة OCI](https://github.com/opencontainers/image-spec/blob/main/annotations.md) سياق الإصدار: يستخدم تاريخ هذا المستودع المميز بالفعل Bookworm في `v2026.2.22` وعلامات 2026 السابقة (على سبيل المثال `v2026.2.21`, `v2026.2.9`). افتراضيًا، يبني برنامج الإعداد الصورة من المصدر. لسحب صورة مبنية مسبقًا بدلاً من ذلك، عيّن `OPENCLAW_IMAGE` قبل تشغيل البرنامج النصي:

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

يكتشف البرنامج النصي أن `OPENCLAW_IMAGE` ليس الافتراضي `openclaw:local` ويشغل `docker pull` بدلاً من `docker build`. كل شيء آخر (الإعداد الأولي، بدء البوابة، إنشاء الرمز) يعمل بنفس الطريقة. لا يزال `docker-setup.sh` يعمل من جذر المستودع لأنه يستخدم `docker-compose.yml` المحلي والملفات المساعدة. يتخطى `OPENCLAW_IMAGE` وقت بناء الصورة المحلية؛ لا يحل محل سير عمل compose/الإعداد.

### مساعدات Shell (اختياري)

لتسهيل إدارة Docker اليومية، ثبّت `ClawDock`:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**أضف إلى تكوين shell الخاص بك (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

ثم استخدم `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`، إلخ. شغل `clawdock-help` لجميع الأوامر. راجع [دليل `ClawDock` المساعد](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) للحصول على التفاصيل.

### السير اليدوي (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

ملاحظة: شغل `docker compose ...` من جذر المستودع. إذا مكّنت `OPENCLAW_EXTRA_MOUNTS` أو `OPENCLAW_HOME_VOLUME`، يكتب برنامج الإعداد `docker-compose.extra.yml`؛ قم بتضمينه عند تشغيل Compose في مكان آخر:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### رمز واجهة التحكم + الاقتران (Docker)

إذا رأيت "غير مصرح" أو "تم قطع الاتصال (1008): الاقتران مطلوب"، احصل على رابط لوحة تحكم جديد ووافق على جهاز المتصفح:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

مزيد من التفاصيل: [لوحة التحكم](../web/dashboard.md), [الأجهزة](../cli/devices.md).

### نقاط ربط إضافية (اختياري)

إذا كنت تريد ربط أدلة مضيف إضافية داخل الحاويات، عيّن `OPENCLAW_EXTRA_MOUNTS` قبل تشغيل `docker-setup.sh`. يقبل هذا قائمة مفصولة بفواصل من نقاط ربط Docker ويطبقها على كل من `openclaw-gateway` و `openclaw-cli` عن طريق إنشاء `docker-compose.extra.yml`. مثال:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

ملاحظات:

-   يجب مشاركة المسارات مع Docker Desktop على macOS/Windows.
-   يجب أن يكون كل إدخال `source:target[:options]` بدون مسافات أو علامات تبويب أو أسطر جديدة.
-   إذا قمت بتحرير `OPENCLAW_EXTRA_MOUNTS`، أعد تشغيل `docker-setup.sh` لإعادة إنشاء ملف compose الإضافي.
-   يتم إنشاء `docker-compose.extra.yml`. لا تقم بتحريره يدويًا.

### الإبقاء على مجلد home الحاوية بالكامل (اختياري)

إذا كنت تريد بقاء `/home/node` عبر إعادة إنشاء الحاوية، عيّن وحدة تخزين مسماة عبر `OPENCLAW_HOME_VOLUME`. ينشئ هذا وحدة تخزين Docker ويربطها في `/home/node`، مع الحفاظ على نقاط ربط التكوين/مساحة العمل القياسية. استخدم وحدة تخزين مسماة هنا (وليس مسار ربط bind)؛ لنقاط ربط bind، استخدم `OPENCLAW_EXTRA_MOUNTS`. مثال:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

يمكنك الجمع بين هذا ونقاط الربط الإضافية:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

ملاحظات:

-   يجب أن تطابق وحدات التخزين المسماة `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
-   إذا قمت بتغيير `OPENCLAW_HOME_VOLUME`، أعد تشغيل `docker-setup.sh` لإعادة إنشاء ملف compose الإضافي.
-   تبقى وحدة التخزين المسماة حتى تتم إزالتها باستخدام `docker volume rm `.

### تثبيت حزم apt إضافية (اختياري)

إذا كنت تحتاج إلى حزم نظام داخل الصورة (على سبيل المثال، أدوات بناء أو مكتبات وسائط)، عيّن `OPENCLAW_DOCKER_APT_PACKAGES` قبل تشغيل `docker-setup.sh`. يثبت هذا الحزم أثناء بناء الصورة، لذا تبقى حتى إذا تم حذف الحاوية. مثال:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

ملاحظات:

-   يقبل هذا قائمة مفصولة بمسافات من أسماء حزم apt.
-   إذا قمت بتغيير `OPENCLAW_DOCKER_APT_PACKAGES`، أعد تشغيل `docker-setup.sh` لإعادة بناء الصورة.

### تثبيت تبعيات الإضافات مسبقًا (اختياري)

الإضافات التي تحتوي على `package.json` الخاص بها (مثل `diagnostics-otel`, `matrix`, `msteams`) تثبت تبعيات npm الخاصة بها عند التحميل الأول. لدمج تلك التبعيات في الصورة بدلاً من ذلك، عيّن `OPENCLAW_EXTENSIONS` قبل تشغيل `docker-setup.sh`:

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

أو عند البناء مباشرة:

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

ملاحظات:

-   يقبل هذا قائمة مفصولة بمسافات من أسماء أدلة الإضافات (تحت `extensions/`).
-   فقط الإضافات التي تحتوي على `package.json` تتأثر؛ يتم تجاهل الإضافات الخفيفة التي لا تحتوي على واحد.
-   إذا قمت بتغيير `OPENCLAW_EXTENSIONS`، أعد تشغيل `docker-setup.sh` لإعادة بناء الصورة.

### مستخدم متقدم / حاوية كاملة الميزات (اختياري)

صورة Docker الافتراضية هي **أولوية للأمان** وتشغل كـ مستخدم غير الجذر `node`. هذا يحافظ على سطح الهجوم صغيرًا، لكنه يعني:

-   لا تثبيت لحزم النظام في وقت التشغيل
-   لا يوجد Homebrew افتراضيًا
-   لا يوجد متصفحات Chromium/Playwright مجمعة

إذا كنت تريد حاوية أكثر اكتمالاً بالميزات، استخدم هذه المقابض الاختيارية:

1.  **الإبقاء على `/home/node`** حتى تبقى تنزيلات المتصفح وذاكرة التخزين المؤقت للأدوات:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **دمج تبعيات النظام في الصورة** (قابلة للتكرار + دائمة):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **تثبيت متصفحات Playwright بدون `npx`** (يتجنب تعارضات تجاوز npm):

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

إذا كنت تحتاج إلى Playwright لتثبيت تبعيات النظام، أعد بناء الصورة باستخدام `OPENCLAW_DOCKER_APT_PACKAGES` بدلاً من استخدام `--with-deps` في وقت التشغيل.

4.  **الإبقاء على تنزيلات متصفح Playwright**:

-   عيّن `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` في `docker-compose.yml`.
-   تأكد من بقاء `/home/node` عبر `OPENCLAW_HOME_VOLUME`، أو اربط `/home/node/.cache/ms-playwright` عبر `OPENCLAW_EXTRA_MOUNTS`.

### الأذونات + EACCES

تشغل الصورة كـ `node` (uid 1000). إذا رأيت أخطاء أذونات على `/home/node/.openclaw`، تأكد من أن نقاط ربط المضيف الخاصة بك مملوكة لـ uid 1000. مثال (مضيف Linux):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

إذا اخترت التشغيل كـ جذر للراحة، فأنت تقبل مقايضة الأمان.

### إعادة بناء أسرع (موصى به)

لتسريع عمليات إعادة البناء، رتب Dockerfile الخاص بك بحيث يتم تخزين طبقات التبعية مؤقتًا. هذا يتجنب إعادة تشغيل `pnpm install` إلا إذا تغيرت ملفات القفل:

```dockerfile
FROM node:22-bookworm

# Install Bun (required for build scripts)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Cache dependencies unless package metadata changes
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### إعداد القنوات (اختياري)

استخدم حاوية CLI لتكوين القنوات، ثم أعد تشغيل البوابة إذا لزم الأمر. WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (رمز البوت):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (رمز البوت):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

الوثائق: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md)

### مصادقة OpenAI Codex OAuth (Docker بدون واجهة)

إذا اخترت مصادقة OpenAI Codex OAuth في المعالج، فإنه يفتح عنوان URL للمتصفح ويحاول التقاط رد اتصال على `http://127.0.0.1:1455/auth/callback`. في إعدادات Docker أو بدون واجهة، قد يظهر هذا الرد اتصال خطأ في المتصفح. انسخ عنوان URL إعادة التوجيه الكامل الذي تصل إليه والصقه مرة أخرى في المعالج لإكمال المصادقة.

### فحوصات الصحة

نقاط فحص الحاوية (لا تتطلب مصادقة):

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

أسماء مستعارة: `/health` و `/ready`. `/healthz` هو فحص حياة سطحي لـ "عملية البوابة قيد التشغيل". يبقى `/readyz` جاهزًا خلال فترة السماح عند البدء، ثم يصبح `503` فقط إذا كانت القنوات المدارة المطلوبة لا تزال غير متصلة بعد فترة السماح أو تنقطع لاحقًا. تحتوي صورة Docker على `HEALTHCHECK` مدمج يفحص `/healthz` في الخلفية. بعبارات بسيطة: يستمر Docker في التحقق مما إذا كان OpenClaw لا يزال مستجيبًا. إذا استمرت الفحوصات في الفشل، يحدد Docker الحاوية على أنها `unhealthy`، ويمكن لأنظمة التنسيق (سياسة إعادة تشغيل Docker Compose، Swarm، Kubernetes، إلخ.) إعادة تشغيلها أو استبدالها تلقائيًا. لقطة صحية عميقة مع مصادقة (البوابة + القنوات):

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### اختبار دخان شامل (Docker)

```
scripts/e2e/onboard-docker.sh
```

### اختبار دخان استيراد QR (Docker)

```bash
pnpm test:docker:qr
```

### LAN مقابل loopback (Docker Compose)

يضبط `docker-setup.sh` افتراضيًا `OPENCLAW_GATEWAY_BIND=lan` حتى يعمل وصول المضيف إلى `http://127.0.0.1:18789` مع نشر منفذ Docker.

-   `lan` (الافتراضي): يمكن لمتصفح المضيف + CLI المضيف الوصول إلى منفذ البوابة المنشور.
-   `loopback`: فقط العمليات داخل نطاق اسم شبكة الحاوية يمكنها الوصول إلى البوابة مباشرة؛ قد يفشل الوصول إلى المنفذ المنشور على المضيف.

يثبت برنامج الإعداد أيضًا `gateway.mode=local` بعد الإعداد الأولي حتى تستهدف أوامر CLI لـ Docker افتراضيًا loopback المحلي. ملاحظة تكوين قديم: استخدم قيم وضع الربط في `gateway.bind` (`lan` / `loopback` / `custom` / `tailnet` / `auto`)، وليس أسماء مستعارة للمضيف (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`). إذا رأيت `Gateway target: ws://172.x.x.x:18789` أو أخطاء متكررة `pairing required` من أوامر CLI لـ Docker، شغل:

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### ملاحظات

-   ربط البوابة الافتراضي هو `lan` لاستخدام الحاوية (`OPENCLAW_GATEWAY_BIND`).
-   يستخدم Dockerfile CMD `--allow-unconfigured`؛ سيظل التكوين المربوط مع `gateway.mode` ليس `local` يبدأ. تجاوز CMD لفرض الحماية.
-   حاوية البوابة هي مصدر الحقيقة للجلسات (`~/.openclaw/agents//sessions/`).

### نموذج التخزين

-   **بيانات المضيف الدائمة:** تربط Docker Compose `OPENCLAW_CONFIG_DIR` بـ `/home/node/.openclaw` و `OPENCLAW_WORKSPACE_DIR` بـ `/home/node/.openclaw/workspace`، لذا تبقى هذه المسارات عند استبدال الحاوية.
-   **tmpfs مؤقت للتجربة (sandbox):** عند تمكين `agents.defaults.sandbox`، تستخدم حاويات التجربة (sandbox) `tmpfs` لـ `/tmp`, `/var/tmp`, و `/run`. نقاط الربط هذه منفصلة عن كومة Compose الرئيسية وتختفي مع حاوية التجربة (sandbox).
-   **بؤر نمو القرص:** راقب `media/`, `agents//sessions/sessions.json`, ملفات JSONL للنصوص, `cron/runs/*.jsonl`, وسجلات الملفات المتداولة تحت `/tmp/openclaw/` (أو `logging.file` الذي قمت بتكوينه). إذا كنت تشغل أيضًا تطبيق macOS خارج Docker، فإن سجلات خدمته منفصلة مرة أخرى: `~/.openclaw/logs/gateway.log`, `~/.openclaw/logs/gateway.err.log`, و `/tmp/openclaw/openclaw-gateway.log`.

## تجزئة الوكيل (بوابة المضيف + أدوات Docker)

غوص عميق: [التجزئة (Sandboxing)](../gateway/sandboxing.md)

### ما الذي يفعله

عند تمكين `agents.defaults.sandbox`، تشغل **الجلسات غير الرئيسية** الأدوات داخل حاوية Docker. تبقى البوابة على مضيفك، لكن تنفيذ الأداة معزول:

-   النطاق: `"agent"` افتراضيًا (حاوية واحدة + مساحة عمل لكل وكيل)
-   النطاق: `"session"` للعزل لكل جلسة
-   مجلد مساحة عمل لكل نطاق مربوط في `/workspace`
-   وصول اختياري لمساحة عمل الوكيل (`agents.defaults.sandbox.workspaceAccess`)
-   سياسة السماح/رفض الأداة (الرفض يفوز)
-   يتم نسخ الوسائط الواردة في مساحة عمل التجربة (sandbox) النشطة (`media/inbound/*`) حتى تتمكن الأدوات من قراءتها (مع `workspaceAccess: "rw"`، يهبط هذا في مساحة عمل الوكيل)

تحذير: `scope: "shared"` يعطل العزل بين الجلسات. تشارك جميع الجلسات حاوية واحدة ومساحة عمل واحدة.

### ملفات تعريف تجزئة لكل وكيل (متعدد الوكلاء)

إذا كنت تستخدم توجيه متعدد الوكلاء، يمكن لكل وكيل تجاوز إعدادات التجربة (sandbox) + الأدوات: `agents.list[].sandbox` و `agents.list[].tools` (بالإضافة إلى `agents.list[].tools.sandbox.tools`). هذا يتيح لك تشغيل مستويات وصول مختلطة في بوابة واحدة:

-   وصول كامل (وكيل شخصي)
-   أدوات للقراءة فقط + مساحة عمل للقراءة فقط (وكيل عائلي/عمل)
-   لا أدوات نظام ملفات/ shell (وكيل عام)

راجع [تجربة وأدوات متعددة الوكلاء (Multi-Agent Sandbox & Tools)](../tools/multi-agent-sandbox-tools.md) للحصول على أمثلة وأسبقية واستكشاف الأخطاء وإصلاحها.

### السلوك الافتراضي

-   الصورة: `openclaw-sandbox:bookworm-slim`
-   حاوية واحدة لكل وكيل
-   وصول مساحة عمل الوكيل: `workspaceAccess: "none"` (الافتراضي) يستخدم `~/.openclaw/sandboxes`
    -   `"ro"` يحافظ على مساحة عمل التجربة (sandbox) في `/workspace` ويربط مساحة عمل الوكيل للقراءة فقط في `/agent` (يعطل `write`/`edit`/`apply_patch`)
    -   `"rw"` يربط مساحة عمل الوكيل للقراءة/الكتابة في `/workspace`
-   التقليم التلقائي: خامل > 24 ساعة أو عمر > 7 أيام
-   الشبكة: `none` افتراضيًا (اشترك صراحة إذا كنت تحتاج إلى خروج)
    -   `host` محظور.
    -   `container:` محظور افتراضيًا (خطر الانضمام إلى نطاق الأسماء).
-   السماح الافتراضي: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   الرفض الافتراضي: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### تمكين التجزئة (sandboxing)

إذا كنت تخطط لتثبيت حزم في `setupCommand`، لاحظ:

-   `docker.network` الافتراضي هو `"none"` (لا خروج).
-   `docker.network: "host"` محظور.
-   `docker.network: "container:"` محظور افتراضيًا.
-   تجاوز كسر الحماية: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
-   `readOnlyRoot: true` يمنع تثبيت الحزم.
-   يجب أن يكون `user` هو الجذر لـ `apt-get` (احذف `user` أو عيّن `user: "0:0"`). يعيد OpenClaw إنشاء الحاويات تلقائيًا عندما يتغير `setupCommand` (أو تكوين docker) إلا إذا كانت الحاوية **مستخدمة مؤخرًا** (خلال ~5 دقائق). تسجل الحاويات الساخنة تحذيرًا مع أمر `openclaw sandbox recreate ...` الدقيق.

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", //