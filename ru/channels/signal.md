

  Платформы обмена сообщениями

  
# Signal

Статус: внешняя интеграция CLI. Шлюз взаимодействует с `signal-cli` через HTTP JSON-RPC + SSE.

## Предварительные требования

-   OpenClaw установлен на вашем сервере (нижеприведенный процесс для Linux протестирован на Ubuntu 24).
-   `signal-cli` доступен на хосте, где работает шлюз.
-   Номер телефона, способный получить одну SMS для верификации (для пути регистрации через SMS).
-   Доступ к браузеру для капчи Signal (`signalcaptchas.org`) во время регистрации.

## Быстрая настройка (для начинающих)

1.  Используйте **отдельный номер Signal** для бота (рекомендуется).
2.  Установите `signal-cli` (требуется Java, если используется сборка JVM).
3.  Выберите один из путей настройки:
    -   **Путь A (QR-ссылка):** `signal-cli link -n "OpenClaw"` и отсканируйте QR-код в Signal.
    -   **Путь B (регистрация через SMS):** зарегистрируйте выделенный номер с капчей и SMS-верификацией.
4.  Настройте OpenClaw и перезапустите шлюз.
5.  Отправьте первое личное сообщение и подтвердите спаривание (`openclaw pairing approve signal <КОД>`).

Минимальная конфигурация:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Справочник по полям:

| Поле | Описание |
| --- | --- |
| `account` | Номер телефона бота в формате E.164 (`+15551234567`) |
| `cliPath` | Путь к `signal-cli` (`signal-cli`, если он в `PATH`) |
| `dmPolicy` | Политика доступа к личным сообщениям (рекомендуется `pairing`) |
| `allowFrom` | Номера телефонов или значения `uuid:`, которым разрешено отправлять личные сообщения |

## Что это такое

-   Канал Signal через `signal-cli` (не встроенная библиотека libsignal).
-   Детерминированная маршрутизация: ответы всегда возвращаются в Signal.
-   Личные сообщения используют основную сессию агента; группы изолированы (`agent::signal:group:`).

## Запись конфигурации

По умолчанию Signal разрешено записывать обновления конфигурации, инициированные командой `/config set|unset` (требуется `commands.config: true`). Отключите это с помощью:

```json
{
  channels: { signal: { configWrites: false } },
}
```

## Модель номера (важно)

-   Шлюз подключается к **устройству Signal** (учетной записи `signal-cli`).
-   Если вы запускаете бота на **вашем личном аккаунте Signal**, он будет игнорировать ваши собственные сообщения (защита от зацикливания).
-   Для сценария «Я пишу боту, и он отвечает» используйте **отдельный номер для бота**.

## Путь настройки A: привязка существующего аккаунта Signal (QR)

1.  Установите `signal-cli` (сборка JVM или нативная).
2.  Привяжите аккаунт бота:
    -   `signal-cli link -n "OpenClaw"`, затем отсканируйте QR-код в Signal.
3.  Настройте Signal и запустите шлюз.

Пример:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Поддержка нескольких аккаунтов: используйте `channels.signal.accounts` с конфигурацией для каждого аккаунта и опциональным `name`. См. [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) для общего шаблона.

## Путь настройки B: регистрация выделенного номера для бота (SMS, Linux)

Используйте этот путь, когда вам нужен выделенный номер для бота, а не привязка существующего аккаунта приложения Signal.

1.  Получите номер, способный принимать SMS (или голосовую верификацию для стационарных телефонов).
    -   Используйте выделенный номер для бота, чтобы избежать конфликтов аккаунтов/сессий.
2.  Установите `signal-cli` на хосте шлюза:

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Если вы используете сборку JVM (`signal-cli-${VERSION}.tar.gz`), сначала установите JRE 25+. Поддерживайте `signal-cli` в актуальном состоянии; в исходном коде отмечается, что старые версии могут перестать работать при изменении API серверов Signal.

3.  Зарегистрируйте и подтвердите номер:

```bash
signal-cli -a +<НОМЕР_БОТА> register
```

Если требуется капча:

1.  Откройте `https://signalcaptchas.org/registration/generate.html`.
2.  Пройдите капчу, скопируйте цель ссылки `signalcaptcha://...` из «Open Signal».
3.  По возможности запускайте регистрацию с того же внешнего IP-адреса, что и сессия браузера.
4.  Немедленно запустите регистрацию снова (токены капчи быстро истекают):

```bash
signal-cli -a +<НОМЕР_БОТА> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<НОМЕР_БОТА> verify <КОД_ПОДТВЕРЖДЕНИЯ>
```

4.  Настройте OpenClaw, перезапустите шлюз, проверьте канал:

```bash
# Если вы запускаете шлюз как пользовательский сервис systemd:
systemctl --user restart openclaw-gateway

# Затем проверьте:
openclaw doctor
openclaw channels status --probe
```

5.  Спарьте отправителя личных сообщений:
    -   Отправьте любое сообщение на номер бота.
    -   Подтвердите код на сервере: `openclaw pairing approve signal <КОД_СПАРИВАНИЯ>`.
    -   Сохраните номер бота как контакт на вашем телефоне, чтобы избежать «Неизвестный контакт».

Важно: регистрация аккаунта с номером телефона через `signal-cli` может деаутентифицировать основную сессию приложения Signal для этого номера. Предпочтительнее использовать выделенный номер для бота или режим QR-привязки, если вам нужно сохранить существующую настройку приложения на телефоне. Ссылки на исходный код:

-   README `signal-cli`: `https://github.com/AsamK/signal-cli`
-   Процесс с капчей: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   Процесс привязки: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Режим внешнего демона (httpUrl)

Если вы хотите управлять `signal-cli` самостоятельно (медленный холодный старт JVM, инициализация контейнера или общие CPU), запустите демон отдельно и укажите OpenClaw на него:

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Это пропускает авто-запуск и ожидание старта внутри OpenClaw. Для медленных стартов при авто-запуске установите `channels.signal.startupTimeoutMs`.

## Контроль доступа (личные сообщения + группы)

Личные сообщения:

-   По умолчанию: `channels.signal.dmPolicy = "pairing"`.
-   Неизвестные отправители получают код спаривания; сообщения игнорируются до подтверждения (коды истекают через 1 час).
-   Подтвердите через:
    -   `openclaw pairing list signal`
    -   `openclaw pairing approve signal <КОД>`
-   Спаривание — это стандартный обмен токенами для личных сообщений Signal. Подробности: [Спаривание](./pairing.md)
-   Отправители только по UUID (из `sourceUuid`) сохраняются как `uuid:` в `channels.signal.allowFrom`.

Группы:

-   `channels.signal.groupPolicy = open | allowlist | disabled`.
-   `channels.signal.groupAllowFrom` управляет тем, кто может инициировать действия в группах, когда установлен `allowlist`.
-   Примечание для времени выполнения: если `channels.signal` полностью отсутствует, среда выполнения возвращается к `groupPolicy="allowlist"` для проверок групп (даже если установлен `channels.defaults.groupPolicy`).

## Как это работает (поведение)

-   `signal-cli` работает как демон; шлюз читает события через SSE.
-   Входящие сообщения нормализуются в общий конверт канала.
-   Ответы всегда маршрутизируются обратно на тот же номер или группу.

## Медиа + ограничения

-   Исходящий текст разбивается на части по `channels.signal.textChunkLimit` (по умолчанию 4000).
-   Опциональное разбиение по переносам строк: установите `channels.signal.chunkMode="newline"` для разделения по пустым строкам (границам абзацев) перед разбиением по длине.
-   Поддерживаются вложения (base64, получаемые от `signal-cli`).
-   Ограничение на медиа по умолчанию: `channels.signal.mediaMaxMb` (по умолчанию 8).
-   Используйте `channels.signal.ignoreAttachments`, чтобы пропустить загрузку медиа.
-   Контекст истории группы использует `channels.signal.historyLimit` (или `channels.signal.accounts.*.historyLimit`), с откатом на `messages.groupChat.historyLimit`. Установите `0` для отключения (по умолчанию 50).

## Индикаторы набора текста + квитанции о прочтении

-   **Индикаторы набора текста**: OpenClaw отправляет сигналы набора текста через `signal-cli sendTyping` и обновляет их, пока выполняется ответ.
-   **Квитанции о прочтении**: когда `channels.signal.sendReadReceipts` равно true, OpenClaw пересылает квитанции о прочтении для разрешенных личных сообщений.
-   Signal-cli не предоставляет квитанции о прочтении для групп.

## Реакции (инструмент message)

-   Используйте `message action=react` с `channel=signal`.
-   Цели: отправитель в формате E.164 или UUID (используйте `uuid:` из вывода спаривания; также работает и голый UUID).
-   `messageId` — это временная метка Signal для сообщения, на которое вы реагируете.
-   Реакции в группах требуют `targetAuthor` или `targetAuthorUuid`.

Примеры:

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Конфигурация:

-   `channels.signal.actions.reactions`: включить/отключить действия с реакциями (по умолчанию true).
-   `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
    -   `off`/`ack` отключает реакции агента (инструмент `react` выдаст ошибку).
    -   `minimal`/`extensive` включает реакции агента и устанавливает уровень детализации.
-   Переопределения для каждого аккаунта: `channels.signal.accounts..actions.reactions`, `channels.signal.accounts..reactionLevel`.

## Цели доставки (CLI/cron)

-   Личные сообщения: `signal:+15551234567` (или просто E.164).
-   Личные сообщения по UUID: `uuid:` (или голый UUID).
-   Группы: `signal:group:`.
-   Имена пользователей: `username:` (если поддерживается вашим аккаунтом Signal).

## Устранение неполадок

Сначала выполните эту последовательность:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Затем при необходимости подтвердите состояние спаривания для личных сообщений:

```bash
openclaw pairing list signal
```

Распространенные проблемы:

-   Демон доступен, но нет ответов: проверьте настройки аккаунта/демона (`httpUrl`, `account`) и режим приема.
-   Личные сообщения игнорируются: отправитель ожидает подтверждения спаривания.
-   Сообщения в группах игнорируются: блокировка доставки из-за ограничений отправителя/упоминания в группе.
-   Ошибки валидации конфигурации после редактирования: выполните `openclaw doctor --fix`.
-   Signal отсутствует в диагностике: убедитесь, что `channels.signal.enabled: true`.

Дополнительные проверки:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Для процесса анализа: [/channels/troubleshooting](./troubleshooting.md).

## Примечания по безопасности

-   `signal-cli` хранит ключи аккаунта локально (обычно `~/.local/share/signal-cli/data/`).
-   Создавайте резервную копию состояния аккаунта Signal перед миграцией сервера или пересборкой.
-   Сохраняйте `channels.signal.dmPolicy: "pairing"`, если вы явно не хотите более широкого доступа к личным сообщениям.
-   SMS-верификация нужна только для процессов регистрации или восстановления, но потеря контроля над номером/аккаунтом может осложнить повторную регистрацию.

## Справочник по конфигурации (Signal)

Полная конфигурация: [Конфигурация](../gateway/configuration.md) Параметры провайдера:

-   `channels.signal.enabled`: включить/отключить запуск канала.
-   `channels.signal.account`: E.164 для аккаунта бота.
-   `channels.signal.cliPath`: путь к `signal-cli`.
-   `channels.signal.httpUrl`: полный URL демона (переопределяет хост/порт).
-   `channels.signal.httpHost`, `channels.signal.httpPort`: привязка демона (по умолчанию 127.0.0.1:8080).
-   `channels.signal.autoStart`: авто-запуск демона (по умолчанию true, если `httpUrl` не задан).
-   `channels.signal.startupTimeoutMs`: таймаут ожидания запуска в мс (макс. 120000).
-   `channels.signal.receiveMode`: `on-start | manual`.
-   `channels.signal.ignoreAttachments`: пропустить загрузку вложений.
-   `channels.signal.ignoreStories`: игнорировать истории от демона.
-   `channels.signal.sendReadReceipts`: пересылать квитанции о прочтении.
-   `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (по умолчанию: pairing).
-   `channels.signal.allowFrom`: белый список для личных сообщений (E.164 или `uuid:`). `open` требует `"*"`. В Signal нет имен пользователей; используйте идентификаторы телефона/UUID.
-   `channels.signal.groupPolicy`: `open | allowlist | disabled` (по умолчанию: allowlist).
-   `channels.signal.groupAllowFrom`: белый список отправителей в группах.
-   `channels.signal.historyLimit`: максимальное количество групповых сообщений, включаемых в контекст (0 отключает).
-   `channels.signal.dmHistoryLimit`: ограничение истории личных сообщений в ходах пользователя. Переопределения для каждого пользователя: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
-   `channels.signal.textChunkLimit`: размер чанка для исходящих сообщений (символов).
-   `channels.signal.chunkMode`: `length` (по умолчанию) или `newline` для разделения по пустым строкам (границам абзацев) перед разбиением по длине.
-   `channels.signal.mediaMaxMb`: ограничение на входящие/исходящие медиа (МБ).

Связанные глобальные параметры:

-   `agents.list[].groupChat.mentionPatterns` (Signal не поддерживает нативные упоминания).
-   `messages.groupChat.mentionPatterns` (глобальный запасной вариант).
-   `messages.responsePrefix`.

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)