

  Мета-информация документации

  
# Центры документации

> **ℹ️** Если вы новичок в OpenClaw, начните с [Начало работы](./getting-started.md).

 Используйте эти центры, чтобы найти каждую страницу, включая углубленные материалы и справочную документацию, которые не отображаются в левой навигации.

## Начните здесь

-   [Главная](../home.md)
-   [Начало работы](./getting-started.md)
-   [Быстрый старт](./quickstart.md)
-   [Введение](./onboarding.md)
-   [Мастер настройки](./wizard.md)
-   [Установка](./setup.md)
-   [Панель управления (локальный шлюз)](http://127.0.0.1:18789/)
-   [Помощь](../help.md)
-   [Указатель документации](./docs-directory.md)
-   [Конфигурация](../gateway/configuration.md)
-   [Примеры конфигурации](../gateway/configuration-examples.md)
-   [Ассистент OpenClaw](./openclaw.md)
-   [Демонстрация](./showcase.md)
-   [История](./lore.md)

## Установка + обновления

-   [Docker](../install/docker.md)
-   [Nix](../install/nix.md)
-   [Обновление / откат](../install/updating.md)
-   [Рабочий процесс Bun (экспериментальный)](../install/bun.md)

## Основные концепции

-   [Архитектура](../concepts/architecture.md)
-   [Возможности](../concepts/features.md)
-   [Сетевой хаб](../network.md)
-   [Среда выполнения агента](../concepts/agent.md)
-   [Рабочее пространство агента](../concepts/agent-workspace.md)
-   [Память](../concepts/memory.md)
-   [Цикл агента](../concepts/agent-loop.md)
-   [Потоковая передача + разбиение на части](../concepts/streaming.md)
-   [Маршрутизация между агентами](../concepts/multi-agent.md)
-   [Компактизация](../concepts/compaction.md)
-   [Сессии](../concepts/session.md)
-   [Очистка сессий](../concepts/session-pruning.md)
-   [Инструменты сессии](../concepts/session-tool.md)
-   [Очередь](../concepts/queue.md)
-   [Слэш-команды](../tools/slash-commands.md)
-   [RPC-адаптеры](../reference/rpc.md)
-   [Схемы TypeBox](../concepts/typebox.md)
-   [Обработка часовых поясов](../concepts/timezone.md)
-   [Присутствие](../concepts/presence.md)
-   [Обнаружение + транспорты](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)
-   [Маршрутизация каналов](../channels/channel-routing.md)
-   [Группы](../channels/groups.md)
-   [Групповые сообщения](../channels/group-messages.md)
-   [Резервирование моделей](../concepts/model-failover.md)
-   [OAuth](../concepts/oauth.md)

## Провайдеры + входящий трафик

-   [Центр чат-каналов](../channels.md)
-   [Центр провайдеров моделей](../providers/models.md)
-   [WhatsApp](../channels/whatsapp.md)
-   [Telegram](../channels/telegram.md)
-   [Slack](../channels/slack.md)
-   [Discord](../channels/discord.md)
-   [Mattermost](../channels/mattermost.md) (плагин)
-   [Signal](../channels/signal.md)
-   [BlueBubbles (iMessage)](../channels/bluebubbles.md)
-   [iMessage (устаревшая версия)](../channels/imessage.md)
-   [Парсинг местоположения](../channels/location.md)
-   [Веб-чат](../web/webchat.md)
-   [Вебхуки](../automation/webhook.md)
-   [Gmail Pub/Sub](../automation/gmail-pubsub.md)

## Шлюз + эксплуатация

-   [Ранбук шлюза](../gateway.md)
-   [Сетевая модель](../gateway/network-model.md)
-   [Сопряжение шлюзов](../gateway/pairing.md)
-   [Блокировка шлюза](../gateway/gateway-lock.md)
-   [Фоновый процесс](../gateway/background-process.md)
-   [Состояние системы](../gateway/health.md)
-   [Пульс](../gateway/heartbeat.md)
-   [Диагностика](../gateway/doctor.md)
-   [Логирование](../gateway/logging.md)
-   [Песочница](../gateway/sandboxing.md)
-   [Панель управления](../web/dashboard.md)
-   [Управляющий интерфейс](../web/control-ui.md)
-   [Удаленный доступ](../gateway/remote.md)
-   [README удаленного шлюза](../gateway/remote-gateway-readme.md)
-   [Tailscale](../gateway/tailscale.md)
-   [Безопасность](../gateway/security.md)
-   [Устранение неполадок](../gateway/troubleshooting.md)

## Инструменты + автоматизация

-   [Поверхность инструментов](../tools.md)
-   [OpenProse](../prose.md)
-   [Справочник CLI](../cli.md)
-   [Инструмент Exec](../tools/exec.md)
-   [Инструмент PDF](../tools/pdf.md)
-   [Привилегированный режим](../tools/elevated.md)
-   [Cron-задачи](../automation/cron-jobs.md)
-   [Cron vs Пульс](../automation/cron-vs-heartbeat.md)
-   [Размышление + подробный вывод](../tools/thinking.md)
-   [Модели](../concepts/models.md)
-   [Подчиненные агенты](../tools/subagents.md)
-   [CLI отправки агенту](../tools/agent-send.md)
-   [Терминальный интерфейс](../web/tui.md)
-   [Управление браузером](../tools/browser.md)
-   [Браузер (устранение неполадок в Linux)](../tools/browser-linux-troubleshooting.md)
-   [Опросы](../automation/poll.md)

## Узлы, медиа, голос

-   [Обзор узлов](../nodes.md)
-   [Камера](../nodes/camera.md)
-   [Изображения](../nodes/images.md)
-   [Аудио](../nodes/audio.md)
-   [Команда местоположения](../nodes/location-command.md)
-   [Голосовое пробуждение](../nodes/voicewake.md)
-   [Режим разговора](../nodes/talk.md)

## Платформы

-   [Обзор платформ](../platforms.md)
-   [macOS](../platforms/macos.md)
-   [iOS](../platforms/ios.md)
-   [Android](../platforms/android.md)
-   [Windows (WSL2)](../platforms/windows.md)
-   [Linux](../platforms/linux.md)
-   [Веб-поверхности](../web.md)

## Компаньон-приложение для macOS (продвинутый уровень)

-   [Настройка разработки для macOS](../platforms/mac/dev-setup.md)
-   [Строка меню macOS](../platforms/mac/menu-bar.md)
-   [Голосовое пробуждение в macOS](../platforms/mac/voicewake.md)
-   [Голосовое наложение в macOS](../platforms/mac/voice-overlay.md)
-   [Веб-чат для macOS](../platforms/mac/webchat.md)
-   [Холст в macOS](../platforms/mac/canvas.md)
-   [Дочерний процесс в macOS](../platforms/mac/child-process.md)
-   [Состояние системы в macOS](../platforms/mac/health.md)
-   [Иконка в macOS](../platforms/mac/icon.md)
-   [Логирование в macOS](../platforms/mac/logging.md)
-   [Разрешения в macOS](../platforms/mac/permissions.md)
-   [Удаленный доступ в macOS](../platforms/mac/remote.md)
-   [Подписание в macOS](../platforms/mac/signing.md)
-   [Релиз для macOS](../platforms/mac/release.md)
-   [Шлюз для macOS (launchd)](../platforms/mac/bundled-gateway.md)
-   [XPC в macOS](../platforms/mac/xpc.md)
-   [Навыки для macOS](../platforms/mac/skills.md)
-   [Peekaboo для macOS](../platforms/mac/peekaboo.md)

## Рабочее пространство + шаблоны

-   [Навыки](../tools/skills.md)
-   [ClawHub](../tools/clawhub.md)
-   [Конфигурация навыков](../tools/skills-config.md)
-   [AGENTS по умолчанию](../reference/AGENTS.default.md)
-   [Шаблоны: AGENTS](../reference/templates/AGENTS.md)
-   [Шаблоны: BOOTSTRAP](../reference/templates/BOOTSTRAP.md)
-   [Шаблоны: HEARTBEAT](../reference/templates/HEARTBEAT.md)
-   [Шаблоны: IDENTITY](../reference/templates/IDENTITY.md)
-   [Шаблоны: SOUL](../reference/templates/SOUL.md)
-   [Шаблоны: TOOLS](../reference/templates/TOOLS.md)
-   [Шаблоны: USER](../reference/templates/USER.md)

## Эксперименты (исследовательские)

-   [Протокол конфигурации введения](../experiments/onboarding-config-protocol.md)
-   [Исследование: память](../experiments/research/memory.md)
-   [Исследование конфигурации моделей](../experiments/proposals/model-config.md)

## Проект

-   [Благодарности](../reference/credits.md)

## Тестирование + релиз

-   [Тестирование](../reference/test.md)
-   [Контрольный список релиза](../reference/RELEASING.md)
-   [Модели устройств](../reference/device-models.md)

[CI Pipeline](../ci.md)[Указатель документации](./docs-directory.md)

---