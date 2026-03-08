title: "دليل المساعدة والأسئلة الشائعة لتثبيت وإعداد واستكشاف أخطاء OpenClaw"
description: "ابحث عن إجابات سريعة واستكشاف أخطاء متعمق لإعداد OpenClaw، التثبيت، VPS، متعدد الوكلاء، OAuth، مفاتيح API، تبديل النماذج الاحتياطي، و Raspberry Pi."
keywords: ["openclaw الأسئلة الشائعة", "تثبيت openclaw", "استكشاف أخطاء openclaw", "إعداد openclaw", "openclaw vps", "openclaw raspberry pi", "تكوين openclaw", "مصادقة openclaw"]
---

  المساعدة

  
# الأسئلة الشائعة

إجابات سريعة بالإضافة إلى استكشاف أخطاء أعمق للإعدادات الواقعية (التطوير المحلي، VPS، متعدد الوكلاء، OAuth/مفاتيح API، تبديل النماذج الاحتياطي). للتشخيص أثناء التشغيل، راجع [استكشاف الأخطاء وإصلاحها](../gateway/troubleshooting.md). للرجوع الكامل للتكوين، راجع [التكوين](../gateway/configuration.md).

## جدول المحتويات

-   \[البدء السريع وإعداد التشغيل الأول\]
    -   [أنا عالق، ما أسرع طريقة للخروج من هذا المأزق؟](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [ما الطريقة الموصى بها لتثبيت وإعداد OpenClaw؟](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [كيف يمكنني فتح لوحة التحكم بعد الإعداد الأولي؟](#how-do-i-open-the-dashboard-after-onboarding)
    -   [كيف أقوم بمصادقة لوحة التحكم (الرمز المميز) على localhost مقابل البعيد؟](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [ما بيئة التشغيل التي أحتاجها؟](#what-runtime-do-i-need)
    -   [هل يعمل على Raspberry Pi؟](#does-it-run-on-raspberry-pi)
    -   [أي نصائح لتثبيت Raspberry Pi؟](#any-tips-for-raspberry-pi-installs)
    -   [إنه عالق على "استيقظ يا صديقي" / الإعداد الأولي لن يفقس. ماذا الآن؟](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [هل يمكنني نقل إعدادي إلى جهاز جديد (Mac mini) دون إعادة الإعداد الأولي؟](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [أين يمكنني رؤية ما هو جديد في أحدث إصدار؟](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [لا يمكنني الوصول إلى docs.openclaw.ai (خطأ SSL). ماذا الآن؟](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [ما الفرق بين المستقر وبيتا؟](#whats-the-difference-between-stable-and-beta)
    -   [كيف أقوم بتثبيت إصدار بيتا، وما الفرق بين بيتا و dev؟](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [كيف أجرب أحدث البتات؟](#how-do-i-try-the-latest-bits)
    -   [كم من الوقت يستغرق التثبيت والإعداد الأولي عادة؟](#how-long-does-install-and-onboarding-usually-take)
    -   [المثبت عالق؟ كيف أحصل على مزيد من التعليقات؟](#installer-stuck-how-do-i-get-more-feedback)
    -   [تثبيت Windows يقول git غير موجود أو openclaw غير معروف](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [إخراج exec في Windows يظهر نصًا صينيًا مشوهاً، ماذا يجب أن أفعل](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [المستندات لم تجب على سؤالي - كيف أحصل على إجابة أفضل؟](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [كيف أقوم بتثبيت OpenClaw على Linux؟](#how-do-i-install-openclaw-on-linux)
    -   [كيف أقوم بتثبيت OpenClaw على VPS؟](#how-do-i-install-openclaw-on-a-vps)
    -   [أين توجد أدلة التثبيت السحابية/VPS؟](#where-are-the-cloudvps-install-guides)
    -   [هل يمكنني أن أطلب من OpenClaw تحديث نفسه؟](#can-i-ask-openclaw-to-update-itself)
    -   [ماذا يفعل معالج الإعداد الأولي فعليًا؟](#what-does-the-onboarding-wizard-actually-do)
    -   [هل أحتاج إلى اشتراك في Claude أو OpenAI لتشغيل هذا؟](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [هل يمكنني استخدام اشتراك Claude Max بدون مفتاح API](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [كيف تعمل مصادقة Anthropic "setup-token"؟](#how-does-anthropic-setuptoken-auth-work)
    -   [أين أجد رمز setup-token الخاص بـ Anthropic؟](#where-do-i-find-an-anthropic-setuptoken)
    -   [هل تدعم مصادقة اشتراك Claude (Claude Pro أو Max)؟](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [لماذا أرى `HTTP 429: rate_limit_error` من Anthropic؟](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [هل AWS Bedrock مدعوم؟](#is-aws-bedrock-supported)
    -   [كيف تعمل مصادقة Codex؟](#how-does-codex-auth-work)
    -   [هل تدعم مصادقة اشتراك OpenAI (Codex OAuth)؟](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [كيف أقوم بإعداد Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
    -   [هل النموذج المحلي مناسب للمحادثات العابرة؟](#is-a-local-model-ok-for-casual-chats)
    -   [كيف أحافظ على حركة مرور النموذج المستضاف في منطقة محددة؟](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [هل يجب علي شراء Mac Mini لتثبيت هذا؟](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [هل أحتاج إلى Mac mini لدعم iMessage؟](#do-i-need-a-mac-mini-for-imessage-support)
    -   [إذا اشتريت Mac mini لتشغيل OpenClaw، فهل يمكنني توصيله بـ MacBook Pro الخاص بي؟](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [هل يمكنني استخدام Bun؟](#can-i-use-bun)
    -   [Telegram: ماذا يوضع في `allowFrom`؟](#telegram-what-goes-in-allowfrom)
    -   [هل يمكن لعدة أشخاص استخدام رقم WhatsApp واحد مع حالات OpenClaw مختلفة؟](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [هل يمكنني تشغيل وكيل "محادثة سريعة" ووكيل "Opus للبرمجة"؟](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [هل يعمل Homebrew على Linux؟](#does-homebrew-work-on-linux)
    -   [ما الفرق بين التثبيت القابل للاختراق (git) وتثبيت npm؟](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [هل يمكنني التبديل بين تثبيتات npm و git لاحقًا؟](#can-i-switch-between-npm-and-git-installs-later)
    -   [هل يجب أن أقوم بتشغيل Gateway على جهاز الكمبيوتر المحمول الخاص بي أو على VPS؟](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [ما مدى أهمية تشغيل OpenClaw على جهاز مخصص؟](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [ما هي الحدود الدنيا لمتطلبات VPS ونظام التشغيل الموصى به؟](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [هل يمكنني تشغيل OpenClaw في VM وما هي المتطلبات](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [ما هو OpenClaw؟](#what-is-openclaw)
    -   [ما هو OpenClaw، في فقرة واحدة؟](#what-is-openclaw-in-one-paragraph)
    -   [ما هي القيمة المقترحة؟](#whats-the-value-proposition)
    -   [لقد قمت بإعداده للتو، ماذا يجب أن أفعل أولاً](#i-just-set-it-up-what-should-i-do-first)
    -   [ما أهم خمس حالات استخدام يومية لـ OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [هل يمكن أن يساعد OpenClaw في استقطاب العملاء المحتملين والإعلانات والمدونات لـ SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [ما هي المزايا مقابل Claude Code لتطوير الويب؟](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [المهارات والأتمتة](#skills-and-automation)
    -   [كيف يمكنني تخصيص المهارات دون ترك المستودع متسخًا؟](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [هل يمكنني تحميل المهارات من مجلد مخصص؟](#can-i-load-skills-from-a-custom-folder)
    -   [كيف يمكنني استخدام نماذج مختلفة لمهام مختلفة؟](#how-can-i-use-different-models-for-different-tasks)
    -   [يتجمد البوت أثناء القيام بعمل شاق. كيف أفرغ ذلك؟](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Cron أو التذكيرات لا تعمل. ماذا يجب أن أتحقق؟](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [كيف أقوم بتثبيت المهارات على Linux؟](#how-do-i-install-skills-on-linux)
    -   [هل يمكن لـ OpenClaw تشغيل المهام على جدول زمني أو بشكل مستمر في الخلفية؟](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [هل يمكنني تشغيل مهارات Apple macOS فقط من Linux؟](#can-i-run-apple-macos-only-skills-from-linux)
    -   [هل لديك تكامل مع Notion أو HeyGen؟](#do-you-have-a-notion-or-heygen-integration)
    -   [كيف أقوم بتثبيت امتداد Chrome للسيطرة على المتصفح؟](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [العزل والتخزين المؤقت](#sandboxing-and-memory)
    -   [هل هناك مستند مخصص للعزل؟](#is-there-a-dedicated-sandboxing-doc)
    -   [كيف أقوم بربط مجلد مضيف في منطقة العزل؟](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [كيف يعمل التخزين المؤقت؟](#how-does-memory-work)
    -   [يستمر التخزين المؤقت في نسيان الأشياء. كيف أجعلها تثبت؟](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [هل يستمر التخزين المؤقت للأبد؟ ما هي الحدود؟](#does-memory-persist-forever-what-are-the-limits)
    -   [هل يتطلب بحث الذاكرة الدلالية مفتاح OpenAI API؟](#does-semantic-memory-search-require-an-openai-api-key)
-   [أين توجد الأشياء على القرص](#where-things-live-on-disk)
    -   [هل يتم حفظ جميع البيانات المستخدمة مع OpenClaw محليًا؟](#is-all-data-used-with-openclaw-saved-locally)
    -   [أين يخزن OpenClaw بياناته؟](#where-does-openclaw-store-its-data)
    -   [أين يجب أن تعيش ملفات AGENTS.md / SOUL.md / USER.md / MEMORY.md؟](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [ما هي استراتيجية النسخ الاحتياطي الموصى بها؟](#whats-the-recommended-backup-strategy)
    -   [كيف أقوم بإلغاء تثبيت OpenClaw بالكامل؟](#how-do-i-completely-uninstall-openclaw)
    -   [هل يمكن للوكلاء العمل خارج مساحة العمل؟](#can-agents-work-outside-the-workspace)
    -   [أنا في الوضع البعيد - أين يوجد مخزن الجلسة؟](#im-in-remote-mode-where-is-the-session-store)
-   [أساسيات التكوين](#config-basics)
    -   [ما هو تنسيق التكوين؟ أين هو؟](#what-format-is-the-config-where-is-it)
    -   [قمت بتعيين `gateway.bind: "lan"` (أو `"tailnet"`) والآن لا شيء يستمع / تقول واجهة المستخدم غير مصرح به](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [لماذا أحتاج إلى رمز مميز على localhost الآن؟](#why-do-i-need-a-token-on-localhost-now)
    -   [هل يجب علي إعادة التشغيل بعد تغيير التكوين؟](#do-i-have-to-restart-after-changing-config)
    -   [كيف أقوم بتعطيل العبارات الطريفة في CLI؟](#how-do-i-disable-funny-cli-taglines)
    -   [كيف أقوم بتمكين البحث على الويب (وجلب الويب)؟](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.apply مسح تكويني. كيف أتعافى وأتجنب هذا؟](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [كيف أقوم بتشغيل Gateway مركزي مع عمال متخصصين عبر الأجهزة؟](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [هل يمكن لمتصفح OpenClaw التشغيل بدون واجهة مرئية؟](#can-the-openclaw-browser-run-headless)
    -   [كيف يمكنني استخدام Brave للتحكم في المتصفح؟](#how-do-i-use-brave-for-browser-control)
-   [البوابات البعيدة والعقد](#remote-gateways-and-nodes)
    -   [كيف تنتشر الأوامر بين Telegram والبوابة والعقد؟](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [كيف يمكن لوكيلي الوصول إلى جهاز الكمبيوتر الخاص بي إذا كانت البوابة مستضافة عن بُعد؟](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
    -   [Tailscale متصل لكنني لا أحصل على ردود. ماذا الآن؟](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [هل يمكن لحالتي OpenClaw التحدث مع بعضهما البعض (محلي + VPS)؟](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [هل أحتاج إلى VPSs منفصلة لوكلاء متعددين](#do-i-need-separate-vpses-for-multiple-agents)
    -   [هل هناك فائدة لاستخدام عقدة على جهاز الكمبيوتر المحمول الشخصي الخاص بي بدلاً من SSH من VPS؟](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [هل تقوم العقد بتشغيل خدمة بوابة؟](#do-nodes-run-a-gateway-service)
    -   [هل هناك طريقة API / RPC لتطبيق التكوين؟](#is-there-an-api-rpc-way-to-apply-config)
    -   [ما هو التكوين "السليم" الأدنى للتثبيت الأول؟](#whats-a-minimal-sane-config-for-a-first-install)
    -   [كيف أقوم بإعداد Tailscale على VPS والتوصيل من جهاز Mac الخاص بي؟](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [كيف أقوم بتوصيل عقدة Mac ببوابة بعيدة (Tailscale Serve)؟](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [هل يجب أن أقوم بالتثبيت على كمبيوتر محمول ثانٍ أو مجرد إضافة عقدة؟](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [متغيرات البيئة وتحميل .env](#env-vars-and-env-loading)
    -   [كيف يقوم OpenClaw بتحميل متغيرات البيئة؟](#how-does-openclaw-load-environment-variables)
    -   ["لقد بدأت البوابة عبر الخدمة واختفى متغيرات البيئة الخاصة بي." ماذا الآن؟](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [قمت بتعيين `COPILOT_GITHUB_TOKEN`، لكن حالة النماذج تظهر "Shell env: off." لماذا؟](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [الجلسات والمحادثات المتعددة](#sessions-and-multiple-chats)
    -   [كيف أبدأ محادثة جديدة؟](#how-do-i-start-a-fresh-conversation)
    -   [هل يتم إعادة تعيين الجلسات تلقائيًا إذا لم أرسل `/new` أبدًا؟](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [هل هناك طريقة لجعل فريق من حالات OpenClaw بمثابة CEO واحد ووكلاء كثيرين](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [لماذا تم اقتطاع السياق في منتصف المهمة؟ كيف أمنع ذلك؟](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [كيف أقوم بإعادة تعيين OpenClaw بالكامل ولكن أبقي مثبتًا؟](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [أواجه أخطاء "السياق كبير جدًا" - كيف أقوم بإعادة التعيين أو الضغط؟](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [لماذا أرى "LLM request rejected: messages.content.tool\_use.input field required"؟](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [لماذا أتلقى رسائل نبضات قلب كل 30 دقيقة؟](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [هل أحتاج إلى إضافة "حساب بوت" إلى مجموعة WhatsApp؟](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [كيف أحصل على JID لمجموعة WhatsApp؟](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [لماذا لا يرد OpenClaw في مجموعة؟](#why-doesnt-openclaw-reply-in-a-group)
    -   [هل تشارك المجموعات/المواضيع السياق مع الرسائل المباشرة؟](#do-groupsthreads-share-context-with-dms)
    -   [كم عدد مساحات العمل والوكلاء التي يمكنني إنشاؤها؟](#how-many-workspaces-and-agents-can-i-create)
    -   [هل يمكنني تشغيل عدة بوتات أو محادثات في نفس الوقت (Slack)، وكيف يجب أن أقوم بإعداد ذلك؟](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [النماذج: الافتراضيات، الاختيار، الأسماء المستعارة، التبديل](#models-defaults-selection-aliases-switching)
    -   [ما هو "النموذج الافتراضي"؟](#what-is-the-default-model)
    -   [ما النموذج الذي توصي به؟](#what-model-do-you-recommend)
    -   [كيف يمكنني تبديل النماذج دون مسح تكويني؟](#how-do-i-switch-models-without-wiping-my-config)
    -   [هل يمكنني استخدام النماذج المستضافة ذاتيًا (llama.cpp، vLLM، Ollama)؟](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [ماذا تستخدم OpenClaw و Flawd و Krill للنماذج؟](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [كيف يمكنني تبديل النماذج على الطاير (دون إعادة التشغيل)؟](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [هل يمكنني استخدام GPT 5.2 للمهام اليومية و Codex 5.3 للبرمجة](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [لماذا أرى "Model … is not allowed" ثم لا رد؟](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [لماذا أرى "Unknown model: minimax/MiniMax-M2.5"؟](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [هل يمكنني استخدام MiniMax كافتراضي و OpenAI للمهام المعقدة؟](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [هل opus / sonnet / gpt اختصارات مدمجة؟](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [كيف أحدد/أتجاوز اختصارات النماذج (الأسماء المستعارة)؟](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [كيف أضيف نماذج من مزودين آخرين مثل OpenRouter أو Z.AI؟](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [التبديل الاحتياطي للنموذج و "فشلت جميع النماذج"](#model-failover-and-all-models-failed)
    -   [كيف يعمل التبديل الاحتياطي؟](#how-does-failover-work)
    -   [ماذا يعني هذا الخطأ؟](#what-does-this-error-mean)
    -   [قائمة التحقق للإصلاح لـ `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [لماذا حاول أيضًا تجربة Google Gemini وفشل؟](#why-did-it-also-try-google-gemini-and-fail)
-   [ملفات تعريف المصادقة: ما هي وكيفية إدارتها](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [ما هو ملف تعريف المصادقة؟](#what-is-an-auth-profile)
    -   [ما هي معرفات ملفات التعريف النموذجية؟](#what-are-typical-profile-ids)
    -   [هل يمكنني التحكم في ملف تعريف المصادقة الذي يتم تجربته أولاً؟](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuth مقابل مفتاح API: ما الفرق؟](#oauth-vs-api-key-whats-the-difference)
-   [البوابة: المنافذ، "يعمل بالفعل"، والوضع البعيد](#gateway-ports-already-running-and-remote-mode)
    -   [ما المنفذ الذي تستخدمه البوابة؟](#what-port-does-the-gateway-use)
    -   [لماذا يقول `openclaw gateway status` `Runtime: running` لكن `RPC probe: failed`؟](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [لماذا يظهر `openclaw gateway status` `Config (cli)` و `Config (service)` مختلفين؟](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [ماذا يعني "another gateway instance is already listening"؟](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [كيف أقوم بتشغيل OpenClaw في الوضع البعيد (يتصل العميل ببوابة في مكان آخر)؟](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [تقول واجهة التحكم "غير مصرح به" (أو تستمر في إعادة الاتصال). ماذا الآن؟](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [قمت بتعيين `gateway.bind: "tailnet"` لكنه لا يمكنه الربط / لا شيء يستمع](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [هل يمكنني تشغيل بوابات متعددة على نفس المضيف؟](#can-i-run-multiple-gateways-on-the-same-host)
    -   [ماذا يعني "invalid handshake" / code 1008؟](#what-does-invalid-handshake-code-1008-mean)
-   [التسجيل والتشخيص](#logging-and-debugging)
    -   [أين توجد السجلات؟](#where-are-logs)
    -   [كيف أبدأ/أوقف/أعيد تشغيل خدمة البوابة؟](#how-do-i-startstoprestart-the-gateway-service)
    -   [أغلقت طرفيتي على Windows - كيف أعيد تشغيل OpenClaw؟](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [البوابة قيد التشغيل لكن الردود لا تصل أبدًا. ماذا يجب أن أتحقق؟](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   ["مفصول من البوابة: لا سبب" - ماذا الآن؟](#disconnected-from-gateway-no-reason-what-now)
    -   [فشل Telegram setMyCommands بأخطاء شبكة. ماذا يجب أن أتحقق؟](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [TUI لا يظهر أي إخراج. ماذا يجب أن أتحقق؟](#tui-shows-no-output-what-should-i-check)
    -   [كيف أوقف ثم أبدأ البوابة بالكامل؟](#how-do-i-completely-stop-then-start-the-gateway)
    -   [ELI5: `openclaw gateway restart` مقابل `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [ما أسرع طريقة للحصول على مزيد من التفاصيل عندما يفشل شيء ما؟](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [الوسائط والمرفقات](#media-and-attachments)
    -   [أنشأت مهارتي صورة/PDF، لكن لم يتم إرسال شيء](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [الأمان والتحكم في الوصول](#security-and-access-control)
    -   [هل من الآمن تعريض OpenClaw للرسائل المباشرة الواردة؟](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [هل حقن الأوامر يهم فقط البوتات العامة؟](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [هل يجب أن يكون للبوت الخاص بي حساب بريد إلكتروني GitHub خاص أو رقم هاتف](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [هل يمكنني منحه الاستقلالية على رسائلي النصية وهل هذا آمن](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [هل يمكنني استخدام نماذج أرخص لمهام المساعد الشخصي؟](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [نفذت `/start` في Telegram لكنني لم أحصل على رمز الاقتران](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp: هل سيرسل رسائل إلى جهات اتصالي؟ كيف يعمل الاقتران؟](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [أوامر الدردشة، إلغاء المهام، و "لن يتوقف"](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [كيف أوقف ظهور رسائل النظام الداخلية في الدردشة](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [كيف أوقف/ألغي مهمة قيد التشغيل؟](#how-do-i-stopcancel-a-running-task)
    -   [كيف أرسل رسالة Discord من Telegram؟ ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [لماذا يبدو أن البوت "يتجاهل" الرسائل السريعة المتتالية؟](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## أول 60 ثانية إذا كان شيء ما معطلاً

1.  **الحالة السريعة (التحقق الأول)**
    
    نسخ
    
    ```bash
    openclaw status
    ```
    
    ملخص محلي سريع: نظام التشغيل + التحديث، إمكانية الوصول إلى البوابة/الخدمة، الوكلاء/الجلسات، تكوين المزود + مشاكل وقت التشغيل (عندما تكون البوابة قابلة للوصول).
2.  **تقرير قابل للنسخ (آمن للمشاركة)**
    
    نسخ
    
    ```bash
    openclaw status --all
    ```
    
    تشخيص للقراءة فقط مع ذيل السجل (الرموز المميزة محجوبة).
3.  **حالة البرنامج الخفي + المنفذ**
    
    نسخ
    
    ```bash
    openclaw gateway status
    ```
    
    يظهر وقت تشغيل المشرف مقابل إمكانية الوصول إلى RPC، عنوان URL المستهدف للتحقق، وأي تكوين استخدمته الخدمة على الأرجح.
4.  **التحقيقات العميقة**
    
    نسخ
    
    ```bash
    openclaw status --deep
    ```
    
    يقوم بإجراء فحوصات صحة البوابة + تحقيقات المزود (يتطلب بوابة قابلة للوصول). راجع [الصحة](../gateway/health.md).
5.  **تتبع أحدث سجل**
    
    نسخ
    
    ```bash
    openclaw logs --follow
    ```
    
    إذا كان RPC معطلاً، استخدم:
    
    نسخ
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    سجلات الملفات منفصلة عن سجلات الخدمة؛ راجع [التسجيل](../logging.md) و [استكشاف الأخطاء وإصلاحها](../gateway/troubleshooting.md).
6.  **تشغيل الطبيب (الإصلاحات)**
    
    نسخ
    
    ```bash
    openclaw doctor
    ```
    
    يصلح/ينقل التكوين/الحالة + يقوم بإجراء فحوصات الصحة. راجع [الطبيب](../gateway/doctor.md).
7.  **لقطة البوابة**
    
    نسخ
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # يظهر عنوان URL المستهدف + مسار التكوين عند الأخطاء
    ```
    
    يطلب من البوابة قيد التشغيل لقطة كاملة (WS فقط). راجع [الصحة](../gateway/health.md).

## البدء السريع وإعداد التشغيل الأول

### أنا عالق، ما أسرع طريقة للخروج من هذا المأزق

استخدم وكيل ذكاء اصطناعي محلي يمكنه **رؤية جهازك**. هذا أكثر فعالية بكثير من السؤال في Discord، لأن معظم حالات "أنا عالق" هي **مشاكل تكوين أو بيئة محلية** لا يمكن للمساعدين البعيدين فحصها.

-   **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

يمكن لهذه الأدوات قراءة المستودع، وتشغيل الأوامر، وفحص السجلات، والمساعدة في إصلاح إعدادك على مستوى الجهاز (PATH، الخدمات، الأذونات، ملفات المصادقة). امنحهم **نسخة المصدر الكاملة** عبر التثبيت القابل للاختراق (git):

```bash
curl -fsSL https://openclaw.ai/install.sh