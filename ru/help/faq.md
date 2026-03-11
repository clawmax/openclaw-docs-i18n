

  Помощь

  
# ЧаВо

Быстрые ответы плюс глубокое устранение неполадок для реальных сценариев (локальная разработка, VPS, мульти-агент, OAuth/API-ключи, отказоустойчивость моделей). Для диагностики во время выполнения см. [Устранение неполадок](../gateway/troubleshooting.md). Для полного справочника по конфигурации см. [Конфигурация](../gateway/configuration.md).

## Содержание

-   \[Быстрый старт и первоначальная настройка\]
    -   [Я застрял, какой самый быстрый способ выбраться?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [Какой рекомендуемый способ установить и настроить OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [Как открыть панель управления после онбординга?](#how-do-i-open-the-dashboard-after-onboarding)
    -   [Как аутентифицировать панель управления (токен) на localhost vs удалённо?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [Какая среда выполнения мне нужна?](#what-runtime-do-i-need)
    -   [Запускается ли на Raspberry Pi?](#does-it-run-on-raspberry-pi)
    -   [Есть ли советы по установке на Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
    -   [Застрял на “wake up my friend” / онбординг не “вылупляется”. Что делать?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [Могу ли я перенести мою настройку на новую машину (Mac mini) без повторного онбординга?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [Где посмотреть, что нового в последней версии?](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [Не могу получить доступ к docs.openclaw.ai (ошибка SSL). Что делать?](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [В чём разница между стабильной и бета-версией?](#whats-the-difference-between-stable-and-beta)
    -   [Как установить бета-версию и в чём разница между beta и dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [Как попробовать самые свежие сборки?](#how-do-i-try-the-latest-bits)
    -   [Сколько обычно занимает установка и онбординг?](#how-long-does-install-and-onboarding-usually-take)
    -   [Установщик завис? Как получить больше информации?](#installer-stuck-how-do-i-get-more-feedback)
    -   [Установка на Windows говорит, что git не найден или openclaw не распознаётся](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [Вывод exec в Windows показывает искажённый китайский текст, что делать](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [В документации не нашлось ответа на мой вопрос - как получить лучший ответ?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [Как установить OpenClaw на Linux?](#how-do-i-install-openclaw-on-linux)
    -   [Как установить OpenClaw на VPS?](#how-do-i-install-openclaw-on-a-vps)
    -   [Где находятся руководства по установке в облако/VPS?](#where-are-the-cloudvps-install-guides)
    -   [Могу ли я попросить OpenClaw обновиться самостоятельно?](#can-i-ask-openclaw-to-update-itself)
    -   [Что на самом деле делает мастер онбординга?](#what-does-the-onboarding-wizard-actually-do)
    -   [Нужна ли подписка на Claude или OpenAI для запуска?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [Могу ли я использовать подписку Claude Max без API-ключа](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [Как работает аутентификация Anthropic “setup-token”?](#how-does-anthropic-setuptoken-auth-work)
    -   [Где найти Anthropic setup-token?](#where-do-i-find-an-anthropic-setuptoken)
    -   [Поддерживается ли аутентификация по подписке Claude (Claude Pro или Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [Почему я вижу `HTTP 429: rate_limit_error` от Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [Поддерживается ли AWS Bedrock?](#is-aws-bedrock-supported)
    -   [Как работает аутентификация Codex?](#how-does-codex-auth-work)
    -   [Поддерживается ли аутентификация по подписке OpenAI (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [Как настроить Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
    -   [Подходит ли локальная модель для обычных чатов?](#is-a-local-model-ok-for-casual-chats)
    -   [Как удерживать трафик хостинговых моделей в определённом регионе?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [Нужно ли покупать Mac Mini для установки?](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [Нужен ли Mac mini для поддержки iMessage?](#do-i-need-a-mac-mini-for-imessage-support)
    -   [Если я куплю Mac mini для запуска OpenClaw, могу ли я подключить его к моему MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [Могу ли я использовать Bun?](#can-i-use-bun)
    -   [Telegram: что указывать в `allowFrom`?](#telegram-what-goes-in-allowfrom)
    -   [Могут ли несколько человек использовать один номер WhatsApp с разными экземплярами OpenClaw?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [Могу ли я запустить агента для “быстрого чата” и агента “Opus для кодинга”?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [Работает ли Homebrew на Linux?](#does-homebrew-work-on-linux)
    -   [В чём разница между хакабельной (git) установкой и npm install?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [Могу ли я позже переключиться между npm и git установками?](#can-i-switch-between-npm-and-git-installs-later)
    -   [Запускать ли Gateway на ноутбуке или на VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [Насколько важно запускать OpenClaw на выделенной машине?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [Каковы минимальные требования к VPS и рекомендуемая ОС?](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [Могу ли я запустить OpenClaw в виртуальной машине и каковы требования](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [Что такое OpenClaw?](#what-is-openclaw)
    -   [Что такое OpenClaw, в одном абзаце?](#what-is-openclaw-in-one-paragraph)
    -   [В чём ценностное предложение?](#whats-the-value-proposition)
    -   [Я только что настроил, что делать в первую очередь](#i-just-set-it-up-what-should-i-do-first)
    -   [Каковы пять основных повседневных сценариев использования OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [Может ли OpenClaw помочь с лидогенерацией, аутричем, рекламой и блогами для SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [Каковы преимущества по сравнению с Claude Code для веб-разработки?](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [Навыки и автоматизация](#skills-and-automation)
    -   [Как кастомизировать навыки, не загрязняя репозиторий?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [Могу ли я загружать навыки из пользовательской папки?](#can-i-load-skills-from-a-custom-folder)
    -   [Как использовать разные модели для разных задач?](#how-can-i-use-different-models-for-different-tasks)
    -   [Бот зависает при выполнении тяжёлой работы. Как разгрузить это?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Cron или напоминания не срабатывают. Что проверить?](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [Как установить навыки на Linux?](#how-do-i-install-skills-on-linux)
    -   [Может ли OpenClaw запускать задачи по расписанию или непрерывно в фоне?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [Могу ли я запускать навыки, специфичные для Apple macOS, из Linux?](#can-i-run-apple-macos-only-skills-from-linux)
    -   [Есть ли интеграция с Notion или HeyGen?](#do-you-have-a-notion-or-heygen-integration)
    -   [Как установить расширение Chrome для захвата браузера?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [Песочница и память](#sandboxing-and-memory)
    -   [Есть ли отдельная документация по песочнице?](#is-there-a-dedicated-sandboxing-doc)
    -   [Как примонтировать папку хоста в песочницу?](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [Как работает память?](#how-does-memory-work)
    -   [Память постоянно забывает вещи. Как сделать, чтобы запоминалось?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [Память сохраняется навсегда? Каковы ограничения?](#does-memory-persist-forever-what-are-the-limits)
    -   [Требуется ли для семантического поиска по памяти ключ API OpenAI?](#does-semantic-memory-search-require-an-openai-api-key)
-   [Где что лежит на диске](#where-things-live-on-disk)
    -   [Все ли данные, используемые с OpenClaw, сохраняются локально?](#is-all-data-used-with-openclaw-saved-locally)
    -   [Где OpenClaw хранит свои данные?](#where-does-openclaw-store-its-data)
    -   [Где должны находиться AGENTS.md / SOUL.md / USER.md / MEMORY.md?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [Какова рекомендуемая стратегия резервного копирования?](#whats-the-recommended-backup-strategy)
    -   [Как полностью удалить OpenClaw?](#how-do-i-completely-uninstall-openclaw)
    -   [Могут ли агенты работать вне рабочего пространства?](#can-agents-work-outside-the-workspace)
    -   [Я в удалённом режиме - где хранилище сессий?](#im-in-remote-mode-where-is-the-session-store)
-   [Основы конфигурации](#config-basics)
    -   [Какой формат у конфига? Где он находится?](#what-format-is-the-config-where-is-it)
    -   [Я установил `gateway.bind: "lan"` (или `"tailnet"`) и теперь ничего не слушает / UI говорит “неавторизован”](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [Почему теперь нужен токен на localhost?](#why-do-i-need-a-token-on-localhost-now)
    -   [Нужно ли перезапускать после изменения конфига?](#do-i-have-to-restart-after-changing-config)
    -   [Как отключить забавные слоганы в CLI?](#how-do-i-disable-funny-cli-taglines)
    -   [Как включить веб-поиск (и веб-запрос)?](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.apply стёр мой конфиг. Как восстановить и избежать этого?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [Как запустить центральный Gateway со специализированными воркерами на разных устройствах?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [Может ли браузер OpenClaw работать в headless-режиме?](#can-the-openclaw-browser-run-headless)
    -   [Как использовать Brave для управления браузером?](#how-do-i-use-brave-for-browser-control)
-   [Удалённые шлюзы и узлы](#remote-gateways-and-nodes)
    -   [Как команды распространяются между Telegram, шлюзом и узлами?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [Как мой агент может получить доступ к моему компьютеру, если Gateway размещён удалённо?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
    -   [Tailscale подключён, но ответов нет. Что делать?](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [Могут ли два экземпляра OpenClaw общаться друг с другом (локальный + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [Нужны ли отдельные VPS для нескольких агентов](#do-i-need-separate-vpses-for-multiple-agents)
    -   [Есть ли преимущество в использовании узла на моём личном ноутбуке вместо SSH с VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [Запускают ли узлы службу шлюза?](#do-nodes-run-a-gateway-service)
    -   [Есть ли API / RPC способ применить конфиг?](#is-there-an-api-rpc-way-to-apply-config)
    -   [Какова минимальная “разумная” конфигурация для первой установки?](#whats-a-minimal-sane-config-for-a-first-install)
    -   [Как настроить Tailscale на VPS и подключиться с Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [Как подключить узел Mac к удалённому Gateway (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [Установить на второй ноутбук или просто добавить узел?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [Переменные окружения и загрузка .env](#env-vars-and-env-loading)
    -   [Как OpenClaw загружает переменные окружения?](#how-does-openclaw-load-environment-variables)
    -   [“Я запустил Gateway через службу, и мои env vars пропали.” Что делать?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [Я установил `COPILOT_GITHUB_TOKEN`, но статус моделей показывает “Shell env: off.” Почему?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [Сессии и множественные чаты](#sessions-and-multiple-chats)
    -   [Как начать новый разговор?](#how-do-i-start-a-fresh-conversation)
    -   [Сбрасываются ли сессии автоматически, если я никогда не отправляю `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [Есть ли способ создать команду из экземпляров OpenClaw: один CEO и много агентов](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [Почему контекст был обрезан в середине задачи? Как предотвратить?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [Как полностью сбросить OpenClaw, но оставить его установленным?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [Получаю ошибки “context too large” - как сбросить или уплотнить?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [Почему я вижу “LLM request rejected: messages.content.tool\_use.input field required”?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [Почему я получаю heartbeat-сообщения каждые 30 минут?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [Нужно ли добавлять “бот-аккаунт” в группу WhatsApp?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [Как получить JID группы WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [Почему OpenClaw не отвечает в группе?](#why-doesnt-openclaw-reply-in-a-group)
    -   [Делятся ли группы/треды контекстом с личными сообщениями?](#do-groupsthreads-share-context-with-dms)
    -   [Сколько рабочих пространств и агентов я могу создать?](#how-many-workspaces-and-agents-can-i-create)
    -   [Могу ли я запускать несколько ботов или чатов одновременно (Slack), и как это настроить?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [Модели: значения по умолчанию, выбор, алиасы, переключение](#models-defaults-selection-aliases-switching)
    -   [Что такое “модель по умолчанию”?](#what-is-the-default-model)
    -   [Какую модель вы рекомендуете?](#what-model-do-you-recommend)
    -   [Как переключить модели, не стирая конфиг?](#how-do-i-switch-models-without-wiping-my-config)
    -   [Могу ли я использовать самохостовые модели (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [Какие модели используют OpenClaw, Flawd и Krill?](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [Как переключать модели на лету (без перезапуска)?](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [Могу ли я использовать GPT 5.2 для повседневных задач и Codex 5.3 для кодинга](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [Почему я вижу “Model … is not allowed” и затем нет ответа?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [Почему я вижу “Unknown model: minimax/MiniMax-M2.5”?](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [Могу ли я использовать MiniMax по умолчанию, а OpenAI для сложных задач?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [Являются ли opus / sonnet / gpt встроенными сокращениями?](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [Как определить/переопределить сокращения моделей (алиасы)?](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [Как добавить модели от других провайдеров, таких как OpenRouter или Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [Отказоустойчивость моделей и “All models failed”](#model-failover-and-all-models-failed)
    -   [Как работает отказоустойчивость?](#how-does-failover-work)
    -   [Что означает эта ошибка?](#what-does-this-error-mean)
    -   [Чек-лист исправления для `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [Почему он также попробовал Google Gemini и провалился?](#why-did-it-also-try-google-gemini-and-fail)
-   [Профили аутентификации: что это и как управлять](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [Что такое профиль аутентификации?](#what-is-an-auth-profile)
    -   [Каковы типичные ID профилей?](#what-are-typical-profile-ids)
    -   [Могу ли я контролировать, какой профиль аутентификации пробуется первым?](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuth vs API-ключ: в чём разница?](#oauth-vs-api-key-whats-the-difference)
-   [Gateway: порты, “already running”, и удалённый режим](#gateway-ports-already-running-and-remote-mode)
    -   [Какой порт использует Gateway?](#what-port-does-the-gateway-use)
    -   [Почему `openclaw gateway status` говорит `Runtime: running`, но `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [Почему `openclaw gateway status` показывает разные `Config (cli)` и `Config (service)`?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [Что означает “another gateway instance is already listening”?](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [Как запустить OpenClaw в удалённом режиме (клиент подключается к Gateway в другом месте)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [Панель управления говорит “unauthorized” (или постоянно переподключается). Что делать?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [Я установил `gateway.bind: "tailnet"`, но он не может привязаться / ничего не слушает](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [Могу ли я запустить несколько Gateway на одном хосте?](#can-i-run-multiple-gateways-on-the-same-host)
    -   [Что означает “invalid handshake” / код 1008?](#what-does-invalid-handshake-code-1008-mean)
-   [Логирование и отладка](#logging-and-debugging)
    -   [Где находятся логи?](#where-are-logs)
    -   [Как запустить/остановить/перезапустить службу Gateway?](#how-do-i-startstoprestart-the-gateway-service)
    -   [Я закрыл терминал на Windows - как перезапустить OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [Gateway запущен, но ответы никогда не приходят. Что проверить?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   [“Disconnected from gateway: no reason” - что делать?](#disconnected-from-gateway-no-reason-what-now)
    -   [Telegram setMyCommands падает с сетевыми ошибками. Что проверить?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [TUI не показывает вывод. Что проверить?](#tui-shows-no-output-what-should-i-check)
    -   [Как полностью остановить, а затем запустить Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
    -   [Объясни как для пятилетнего: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [Какой самый быстрый способ получить больше деталей, когда что-то падает?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [Медиа и вложения](#media-and-attachments)
    -   [Мой навык сгенерировал изображение/PDF, но ничего не было отправлено](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [Безопасность и контроль доступа](#security-and-access-control)
    -   [Безопасно ли открывать OpenClaw для входящих личных сообщений?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [Инъекция промптов - проблема только для публичных ботов?](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [Должен ли мой бот иметь свой собственный email, аккаунт GitHub или номер телефона](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [Могу ли я дать ему автономию над моими текстовыми сообщениями и безопасно ли это](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [Могу ли я использовать более дешёвые модели для задач личного помощника?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [Я выполнил `/start` в Telegram, но не получил код сопряжения](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp: будет ли он писать моим контактам? Как работает сопряжение?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [Команды чата, прерывание задач и “он не останавливается”](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [Как остановить показ внутренних системных сообщений в чате](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [Как остановить/отменить выполняющуюся задачу?](#how-do-i-stopcancel-a-running-task)
    -   [Как отправить сообщение Discord из Telegram? (“Cross-context messaging denied”)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [Почему кажется, что бот “игнорирует” быстрые сообщения?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## Первые 60 секунд, если что-то сломалось

1.  **Быстрый статус (первая проверка)**
    
    Копировать
    
    ```bash
    openclaw status
    ```
    
    Быстрая локальная сводка: ОС + обновление, доступность шлюза/службы, агенты/сессии, конфигурация провайдера + проблемы среды выполнения (когда шлюз доступен).
2.  **Отчёт для вставки (безопасно делиться)**
    
    Копировать
    
    ```bash
    openclaw status --all
    ```
    
    Диагностика только для чтения с хвостом логов (токены скрыты).
3.  **Состояние демона + порта**
    
    Копировать
    
    ```bash
    openclaw gateway status
    ```
    
    Показывает среду выполнения супервизора vs доступность RPC, целевой URL пробы и какой конфиг, вероятно, использовала служба.
4.  **Глубокие пробы**
    
    Копировать
    
    ```bash
    openclaw status --deep
    ```
    
    Запускает проверки здоровья шлюза + пробы провайдеров (требуется доступный шлюз). См. [Здоровье](../gateway/health.md).
5.  **Хвост последнего лога**
    
    Копировать
    
    ```bash
    openclaw logs --follow
    ```
    
    Если RPC не работает, откатиться на:
    
    Копировать
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    Файловые логи отделены от логов службы; см. [Логирование](../logging.md) и [Устранение неполадок](../gateway/troubleshooting.md).
6.  **Запустить доктора (исправления)**
    
    Копировать
    
    ```bash
    openclaw doctor
    ```
    
    Исправляет/мигрирует конфиг/состояние + запускает проверки здоровья. См. [Доктор](../gateway/doctor.md).
7.  **Снимок состояния Gateway**
    
    Копировать
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # показывает целевой URL + путь к конфигу при ошибках
    ```
    
    Запрашивает у запущенного шлюза полный снимок состояния (только WS). См. [Здоровье](../gateway/health.md).

## Быстрый старт и первоначальная настройка

### Я застрял, какой самый быстрый способ выбраться

Используйте локального ИИ-агента, который **может видеть вашу машину**. Это гораздо эффективнее, чем спрашивать в Discord, потому что большинство случаев “я застрял” - это **локальные проблемы конфигурации или окружения**, которые удалённые помощники не могут проверить.

-   **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

Эти инструменты могут читать репозиторий, выполнять команды, проверять логи и помогать исправлять настройки на уровне машины (PATH, службы, разрешения, файлы аутентификации). Дайте им **полную локальную копию исходников** через хакабельную (git) установку:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Это устанавливает OpenClaw **из git-репозитория**, так что агент может читать код + документацию и рассуждать о точной версии, которую вы запускаете. Вы всегда можете вернуться к стабильной версии позже, повторно запустив установщик без `--install-method git`. Совет: попросите агента **спланировать и проконтролировать** исправление (шаг за шагом), затем выполните только необходимые команды. Это сохраняет изменения небольшими и более лёгкими для проверки. Если вы обнаружили настоящий баг или исправление, пожалуйста, создайте issue на GitHub или отправьте PR: [https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls) Начните с этих команд (делитесь выводом, когда просите помощи):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Что они делают:

-   `openclaw status`: быстрый снимок состояния шлюза/агента + базовая конфигурация.
-   `openclaw models status`: проверяет аутентификацию провайдера + доступность моделей.
-   `openclaw doctor`: проверяет и исправляет распространённые проблемы конфигурации/состояния.