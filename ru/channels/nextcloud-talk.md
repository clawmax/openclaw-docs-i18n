title: "Руководство по установке и настройке плагина OpenClaw для Nextcloud Talk"
description: "Узнайте, как установить и настроить бота OpenClaw для Nextcloud Talk. Поддерживаются личные сообщения, групповые комнаты, реакции и разметка markdown."
keywords: ["nextcloud talk", "плагин openclaw", "интеграция чат-бота", "вебхук-бот", "личные сообщения", "групповые комнаты", "настройка бота", "платформа обмена сообщениями"]
---

  Платформы обмена сообщениями

  
# Nextcloud Talk

Статус: поддерживается через плагин (вебхук-бот). Поддерживаются личные сообщения, комнаты, реакции и сообщения в формате markdown.

## Требуется плагин

Nextcloud Talk поставляется как плагин и не входит в базовую установку. Установите через CLI (npm registry):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Локальная установка (при запуске из git-репозитория):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Если вы выберете Nextcloud Talk во время настройки/онбординга и будет обнаружен git-репозиторий, OpenClaw автоматически предложит путь для локальной установки. Подробнее: [Плагины](../tools/plugin.md)

## Быстрая настройка (для начинающих)

1.  Установите плагин Nextcloud Talk.
2.  На вашем сервере Nextcloud создайте бота:
    
    Копировать
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  Включите бота в настройках целевой комнаты.
4.  Настройте OpenClaw:
    -   Конфигурация: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   Или переменная окружения: `NEXTCLOUD_TALK_BOT_SECRET` (только для аккаунта по умолчанию)
5.  Перезапустите шлюз (или завершите онбординг).

Минимальная конфигурация:

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## Примечания

-   Боты не могут инициировать личные сообщения. Пользователь должен первым написать боту.
-   URL вебхука должен быть доступен для Шлюза; установите `webhookPublicUrl`, если находитесь за прокси.
-   Загрузка медиафайлов не поддерживается API бота; медиа отправляются в виде URL-адресов.
-   Полезная нагрузка вебхука не различает личные сообщения и комнаты; установите `apiUser` + `apiPassword`, чтобы включить определение типа комнаты (иначе личные сообщения обрабатываются как комнаты).

## Контроль доступа (личные сообщения)

-   По умолчанию: `channels.nextcloud-talk.dmPolicy = "pairing"`. Неизвестным отправителям выдается код сопряжения.
-   Подтверждение через:
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   Открытые личные сообщения: `channels.nextcloud-talk.dmPolicy="open"` плюс `channels.nextcloud-talk.allowFrom=["*"]`.
-   `allowFrom` сопоставляется только с идентификаторами пользователей Nextcloud; отображаемые имена игнорируются.

## Комнаты (группы)

-   По умолчанию: `channels.nextcloud-talk.groupPolicy = "allowlist"` (с упоминанием).
-   Добавьте комнаты в разрешительный список с помощью `channels.nextcloud-talk.rooms`:

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   Чтобы запретить все комнаты, оставьте разрешительный список пустым или установите `channels.nextcloud-talk.groupPolicy="disabled"`.

## Возможности

| Функция | Статус |
| --- | --- |
| Личные сообщения | Поддерживается |
| Комнаты | Поддерживается |
| Ветки обсуждений | Не поддерживается |
| Медиафайлы | Только URL |
| Реакции | Поддерживается |
| Нативные команды | Не поддерживается |

## Справочник по конфигурации (Nextcloud Talk)

Полная конфигурация: [Конфигурация](../gateway/configuration.md) Параметры провайдера:

-   `channels.nextcloud-talk.enabled`: включить/отключить запуск канала.
-   `channels.nextcloud-talk.baseUrl`: URL экземпляра Nextcloud.
-   `channels.nextcloud-talk.botSecret`: общий секрет бота.
-   `channels.nextcloud-talk.botSecretFile`: путь к файлу с секретом.
-   `channels.nextcloud-talk.apiUser`: пользователь API для определения комнат (обнаружение личных сообщений).
-   `channels.nextcloud-talk.apiPassword`: пароль API/приложения для определения комнат.
-   `channels.nextcloud-talk.apiPasswordFile`: путь к файлу с паролем API.
-   `channels.nextcloud-talk.webhookPort`: порт прослушивания вебхука (по умолчанию: 8788).
-   `channels.nextcloud-talk.webhookHost`: хост вебхука (по умолчанию: 0.0.0.0).
-   `channels.nextcloud-talk.webhookPath`: путь вебхука (по умолчанию: /nextcloud-talk-webhook).
-   `channels.nextcloud-talk.webhookPublicUrl`: внешне доступный URL вебхука.
-   `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
-   `channels.nextcloud-talk.allowFrom`: разрешительный список для личных сообщений (идентификаторы пользователей). Для `open` требуется `"*"`.
-   `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
-   `channels.nextcloud-talk.groupAllowFrom`: разрешительный список для групп (идентификаторы пользователей).
-   `channels.nextcloud-talk.rooms`: настройки и разрешительный список для каждой комнаты.
-   `channels.nextcloud-talk.historyLimit`: ограничение истории группы (0 отключает).
-   `channels.nextcloud-talk.dmHistoryLimit`: ограничение истории личных сообщений (0 отключает).
-   `channels.nextcloud-talk.dms`: переопределения для каждого личного чата (historyLimit).
-   `channels.nextcloud-talk.textChunkLimit`: размер фрагмента исходящего текста (символы).
-   `channels.nextcloud-talk.chunkMode`: `length` (по умолчанию) или `newline` для разделения по пустым строкам (границам абзацев) перед разделением по длине.
-   `channels.nextcloud-talk.blockStreaming`: отключить потоковую передачу блоков для этого канала.
-   `channels.nextcloud-talk.blockStreamingCoalesce`: настройка объединения потоковой передачи блоков.
-   `channels.nextcloud-talk.mediaMaxMb`: ограничение на входящие медиафайлы (МБ).

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)