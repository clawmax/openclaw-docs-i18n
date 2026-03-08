title: "Справочник и техническая документация по мастеру настройки OpenClaw"
description: "Полное техническое руководство по мастеру настройки CLI openclaw onboard. Изучите пошаговый процесс локальной установки, аутентификации, настройки рабочего пространства, шлюза и демона."
keywords: ["openclaw", "мастер настройки", "справочник cli", "руководство по установке", "аутентификация", "конфигурация шлюза", "установка демона", "настройка рабочего пространства"]
---

  Технический справочник

  
# Справочник по мастеру настройки

Это полный справочник по мастеру настройки CLI `openclaw onboard`. Для общего обзора см. [Мастер настройки](../start/wizard.md).

## Детали процесса (локальный режим)

### Шаг 1: Обнаружение существующей конфигурации

-   Если существует `~/.openclaw/openclaw.json`, выбрать **Сохранить / Изменить / Сбросить**.
-   Повторный запуск мастера **не** стирает ничего, если вы явно не выберете **Сбросить** (или не передадите `--reset`).
-   CLI `--reset` по умолчанию сбрасывает `config+creds+sessions`; используйте `--reset-scope full`, чтобы также удалить рабочее пространство.
-   Если конфигурация недействительна или содержит устаревшие ключи, мастер останавливается и просит запустить `openclaw doctor` перед продолжением.
-   Сброс использует `trash` (никогда `rm`) и предлагает области:
    -   Только конфигурация
    -   Конфигурация + учетные данные + сессии
    -   Полный сброс (также удаляет рабочее пространство)

### Шаг 2: Модель/Аутентификация

-   **Ключ API Anthropic**: использует `ANTHROPIC_API_KEY`, если присутствует, или запрашивает ключ, затем сохраняет его для использования демоном.
-   **OAuth Anthropic (Claude Code CLI)**: в macOS мастер проверяет элемент Keychain “Claude Code-credentials” (выберите “Always Allow”, чтобы запуски через launchd не блокировались); в Linux/Windows повторно использует `~/.claude/.credentials.json`, если он присутствует.
-   **Токен Anthropic (вставить setup-token)**: запустите `claude setup-token` на любой машине, затем вставьте токен (можно дать ему имя; пустое = по умолчанию).
-   **Подписка OpenAI Code (Codex) (Codex CLI)**: если существует `~/.codex/auth.json`, мастер может повторно использовать его.
-   **Подписка OpenAI Code (Codex) (OAuth)**: браузерный процесс; вставьте `code#state`.
    -   Устанавливает `agents.defaults.model` в `openai-codex/gpt-5.2`, когда модель не задана или равна `openai/*`.
-   **Ключ API OpenAI**: использует `OPENAI_API_KEY`, если присутствует, или запрашивает ключ, затем сохраняет его в профилях аутентификации.
-   **Ключ API xAI (Grok)**: запрашивает `XAI_API_KEY` и настраивает xAI как провайдера моделей.
-   **OpenCode Zen (мультимодельный прокси)**: запрашивает `OPENCODE_API_KEY` (или `OPENCODE_ZEN_API_KEY`, получите его на [https://opencode.ai/auth](https://opencode.ai/auth)).
-   **Ключ API**: сохраняет ключ для вас.
-   **Vercel AI Gateway (мультимодельный прокси)**: запрашивает `AI_GATEWAY_API_KEY`.
-   Подробнее: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**: запрашивает Account ID, Gateway ID и `CLOUDFLARE_AI_GATEWAY_API_KEY`.
-   Подробнее: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**: конфигурация записывается автоматически.
-   Подробнее: [MiniMax](../providers/minimax.md)
-   **Synthetic (Anthropic-совместимый)**: запрашивает `SYNTHETIC_API_KEY`.
-   Подробнее: [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**: конфигурация записывается автоматически.
-   **Kimi Coding**: конфигурация записывается автоматически.
-   Подробнее: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **Пропустить**: аутентификация пока не настроена.
-   Выберите модель по умолчанию из обнаруженных вариантов (или введите провайдер/модель вручную). Для наилучшего качества и снижения риска инъекции промптов выбирайте самую мощную модель последнего поколения, доступную в вашем стеке провайдеров.
-   Мастер выполняет проверку модели и предупреждает, если настроенная модель неизвестна или отсутствует аутентификация.
-   Режим хранения ключей API по умолчанию — обычные значения в профиле аутентификации. Используйте `--secret-input-mode ref`, чтобы хранить ссылки на переменные окружения (например, `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
-   Учетные данные OAuth находятся в `~/.openclaw/credentials/oauth.json`; профили аутентификации находятся в `~/.openclaw/agents//agent/auth-profiles.json` (ключи API + OAuth).
-   Подробнее: [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** Совет для headless/серверов: завершите OAuth на машине с браузером, затем скопируйте `~/.openclaw/credentials/oauth.json` (или `$OPENCLAW_STATE_DIR/credentials/oauth.json`) на хост шлюза.

### Шаг 3: Рабочее пространство

-   По умолчанию `~/.openclaw/workspace` (настраивается).
-   Заполняет файлы рабочего пространства, необходимые для ритуала начальной загрузки агента.
-   Полная структура рабочего пространства + руководство по резервному копированию: [Рабочее пространство агента](../concepts/agent-workspace.md)

### Шаг 4: Шлюз

-   Порт, привязка, режим аутентификации, экспозиция через Tailscale.
-   Рекомендация по аутентификации: оставьте **Токен**, даже для loopback, чтобы локальные WS-клиенты должны были проходить аутентификацию.
-   В режиме токена интерактивная настройка предлагает:
    -   **Сгенерировать/сохранить токен в открытом виде** (по умолчанию)
    -   **Использовать SecretRef** (опционально)
    -   Quickstart повторно использует существующие `gateway.auth.token` SecretRef для `env`, `file` и `exec` провайдеров при начальной загрузке пробы/панели управления.
    -   Если этот SecretRef настроен, но не может быть разрешен, настройка завершается на раннем этапе с понятным сообщением об исправлении, вместо тихого ухудшения аутентификации во время выполнения.
-   В режиме пароля интерактивная настройка также поддерживает хранение в открытом виде или через SecretRef.
-   Путь к SecretRef токена для неинтерактивного режима: `--gateway-token-ref-env <ENV_VAR>`.
    -   Требует непустую переменную окружения в среде процесса настройки.
    -   Нельзя комбинировать с `--gateway-token`.
-   Отключайте аутентификацию только если вы полностью доверяете каждому локальному процессу.
-   Привязки не к loopback все равно требуют аутентификации.

### Шаг 5: Каналы

-   [WhatsApp](../channels/whatsapp.md): опциональный вход по QR-коду.
-   [Telegram](../channels/telegram.md): токен бота.
-   [Discord](../channels/discord.md): токен бота.
-   [Google Chat](../channels/googlechat.md): JSON сервисного аккаунта + аудитория вебхука.
-   [Mattermost](../channels/mattermost.md) (плагин): токен бота + базовый URL.
-   [Signal](../channels/signal.md): опциональная установка `signal-cli` + настройка аккаунта.
-   [BlueBubbles](../channels/bluebubbles.md): **рекомендуется для iMessage**; URL сервера + пароль + вебхук.
-   [iMessage](../channels/imessage.md): устаревший путь CLI `imsg` + доступ к БД.
-   Безопасность личных сообщений: по умолчанию используется сопряжение. Первое личное сообщение отправляет код; подтвердите через `openclaw pairing approve  ` или используйте списки разрешений.

### Шаг 6: Веб-поиск

-   Выберите провайдера: Perplexity, Brave, Gemini, Grok или Kimi (или пропустите).
-   Вставьте ваш ключ API (QuickStart автоматически обнаруживает ключи из переменных окружения или существующей конфигурации).
-   Пропустить с помощью `--skip-search`.
-   Настроить позже: `openclaw configure --section web`.

### Шаг 7: Установка демона

-   macOS: LaunchAgent
    -   Требует сеанс вошедшего в систему пользователя; для headless используйте пользовательский LaunchDaemon (не поставляется).
-   Linux (и Windows через WSL2): systemd user unit
    -   Мастер пытается включить lingering через `loginctl enable-linger `, чтобы шлюз оставался активным после выхода из системы.
    -   Может запросить sudo (записывает `/var/lib/systemd/linger`); сначала пытается без sudo.
-   **Выбор среды выполнения:** Node (рекомендуется; требуется для WhatsApp/Telegram). Bun **не рекомендуется**.
-   Если для аутентификации по токену требуется токен и `gateway.auth.token` управляется через SecretRef, установка демона проверяет его, но не сохраняет разрешенные значения токена в открытом виде в метаданные среды службы супервизора.
-   Если для аутентификации по токену требуется токен и настроенный SecretRef токена не разрешен, установка демона блокируется с указанием действий для исправления.
-   Если настроены и `gateway.auth.token`, и `gateway.auth.password`, а `gateway.auth.mode` не задан, установка демона блокируется до явной установки режима.

### Шаг 8: Проверка состояния

-   Запускает шлюз (если нужно) и выполняет `openclaw health`.
-   Совет: `openclaw status --deep` добавляет пробы состояния шлюза в вывод статуса (требует доступный шлюз).

### Шаг 9: Навыки (рекомендуется)

-   Читает доступные навыки и проверяет требования.
-   Позволяет выбрать менеджер пакетов Node: **npm / pnpm** (bun не рекомендуется).
-   Устанавливает опциональные зависимости (некоторые используют Homebrew на macOS).

### Шаг 10: Завершение

-   Итог + следующие шаги, включая приложения iOS/Android/macOS для дополнительных функций.

 

> **ℹ️** Если графический интерфейс не обнаружен, мастер выводит инструкции по SSH port-forward для панели управления вместо открытия браузера. Если ресурсы панели управления отсутствуют, мастер пытается их собрать; запасной вариант — `pnpm ui:build` (автоматически устанавливает зависимости UI).

## Неинтерактивный режим

Используйте `--non-interactive` для автоматизации или скриптования настройки:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Добавьте `--json` для машиночитаемой сводки. SecretRef токена шлюза в неинтерактивном режиме:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` и `--gateway-token-ref-env` взаимоисключающие. 

> **ℹ️** `--json` **не** подразумевает неинтерактивный режим. Используйте `--non-interactive` (и `--workspace`) для скриптов.

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### Добавление агента (неинтерактивно)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## RPC мастера настройки шлюза

Шлюз предоставляет процесс настройки через RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). Клиенты (приложение macOS, панель управления) могут отображать шаги без повторной реализации логики настройки.

## Настройка Signal (signal-cli)

Мастер может установить `signal-cli` из релизов GitHub:

-   Загружает соответствующий релиз.
-   Сохраняет его в `~/.openclaw/tools/signal-cli//`.
-   Записывает `channels.signal.cliPath` в вашу конфигурацию.

Примечания:

-   Сборки JVM требуют **Java 21**.
-   Нативные сборки используются, когда доступны.
-   Windows использует WSL2; установка signal-cli следует процессу Linux внутри WSL.

## Что записывает мастер

Типичные поля в `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (если выбран Minimax)
-   `tools.profile` (локальная настройка по умолчанию использует `"coding"`, когда не задано; существующие явные значения сохраняются)
-   `gateway.*` (режим, привязка, аутентификация, tailscale)
-   `session.dmScope` (детали поведения: [Справочник по настройке CLI](../start/wizard-cli-reference.md#outputs-and-internals))
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Списки разрешений каналов (Slack/Discord/Matrix/Microsoft Teams), когда вы соглашаетесь во время запросов (имена разрешаются в ID, когда возможно).
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` записывает `agents.list[]` и опциональные `bindings`. Учетные данные WhatsApp помещаются в `~/.openclaw/credentials/whatsapp//`. Сессии хранятся в `~/.openclaw/agents//sessions/`. Некоторые каналы поставляются как плагины. Когда вы выбираете один во время настройки, мастер предложит установить его (npm или локальный путь), прежде чем его можно будет настроить.

## Связанная документация

-   Обзор мастера: [Мастер настройки](../start/wizard.md)
-   Настройка в приложении macOS: [Настройка](../start/onboarding.md)
-   Справочник по конфигурации: [Конфигурация шлюза](../gateway/configuration.md)
-   Провайдеры: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md), [Google Chat](../channels/googlechat.md), [Signal](../channels/signal.md), [BlueBubbles](../channels/bluebubbles.md) (iMessage), [iMessage](../channels/imessage.md) (устаревший)
-   Навыки: [Навыки](../tools/skills.md), [Конфигурация навыков](../tools/skills-config.md)

[USER](./templates/USER.md)[Использование токенов и стоимость](./token-use.md)