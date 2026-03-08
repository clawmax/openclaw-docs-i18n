title: "Маршрутизация и настройка нескольких агентов в OpenClaw Gateway"
description: "Узнайте, как настроить несколько изолированных ИИ-агентов с отдельными рабочими пространствами, учётными записями каналов и правилами маршрутизации в OpenClaw для управления различными персонами и сессиями."
keywords: ["маршрутизация нескольких агентов", "агенты openclaw", "изолированные агенты", "привязки каналов", "рабочее пространство агента", "управление сессиями", "маршрутизация whatsapp", "агенты бота discord"]
---

  Мульти-агент

  
# Маршрутизация нескольких агентов

Цель: несколько *изолированных* агентов (отдельное рабочее пространство + `agentDir` + сессии), а также несколько учётных записей каналов (например, два WhatsApp) в одном работающем Gateway. Входящие сообщения маршрутизируются к агенту через привязки.

## Что такое «один агент»?

**Агент** — это полностью автономный «мозг» со своим:

-   **Рабочим пространством** (файлы, AGENTS.md/SOUL.md/USER.md, локальные заметки, правила персоны).
-   **Директорией состояния** (`agentDir`) для профилей аутентификации, реестра моделей и конфигурации на агента.
-   **Хранилищем сессий** (история чатов + состояние маршрутизации) в `~/.openclaw/agents//sessions`.

Профили аутентификации **на каждого агента**. Каждый агент читает из своего:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Основные учётные данные агента **не** передаются автоматически. Никогда не используйте один `agentDir` для нескольких агентов (это вызывает конфликты аутентификации/сессий). Если вы хотите поделиться учётными данными, скопируйте `auth-profiles.json` в `agentDir` другого агента. Навыки настраиваются на каждого агента через папку `skills/` каждого рабочего пространства, при этом общие навыки доступны из `~/.openclaw/skills`. См. [Навыки: на агента vs общие](../tools/skills.md#per-agent-vs-shared-skills). Gateway может размещать **одного агента** (по умолчанию) или **многих агентов** параллельно. **Примечание о рабочем пространстве:** рабочее пространство каждого агента — это **текущая рабочая директория по умолчанию**, а не жёсткая песочница. Относительные пути разрешаются внутри рабочего пространства, но абсолютные пути могут достигать других мест на хосте, если не включена песочница. См. [Песочница](../gateway/sandboxing.md).

## Пути (быстрая карта)

-   Конфиг: `~/.openclaw/openclaw.json` (или `OPENCLAW_CONFIG_PATH`)
-   Директория состояния: `~/.openclaw` (или `OPENCLAW_STATE_DIR`)
-   Рабочее пространство: `~/.openclaw/workspace` (или `~/.openclaw/workspace-`)
-   Директория агента: `~/.openclaw/agents//agent` (или `agents.list[].agentDir`)
-   Сессии: `~/.openclaw/agents//sessions`

### Режим одного агента (по умолчанию)

Если вы ничего не делаете, OpenClaw запускает одного агента:

-   `agentId` по умолчанию — **`main`**.
-   Сессии ключуются как `agent:main:`.
-   Рабочее пространство по умолчанию: `~/.openclaw/workspace` (или `~/.openclaw/workspace-` при установке `OPENCLAW_PROFILE`).
-   Состояние по умолчанию: `~/.openclaw/agents/main/agent`.

## Помощник по агентам

Используйте мастер агентов, чтобы добавить нового изолированного агента:

```bash
openclaw agents add work
```

Затем добавьте `bindings` (или позвольте мастеру сделать это) для маршрутизации входящих сообщений. Проверьте:

```bash
openclaw agents list --bindings
```

## Быстрый старт

### Шаг 1: Создайте рабочее пространство для каждого агента

Используйте мастер или создайте рабочие пространства вручную:

```bash
openclaw agents add coding
openclaw agents add social
```

Каждый агент получает своё рабочее пространство с `SOUL.md`, `AGENTS.md` и опционально `USER.md`, а также выделенный `agentDir` и хранилище сессий в `~/.openclaw/agents/`.

### Шаг 2: Создайте учётные записи каналов

Создайте по одной учётной записи на каждого агента на предпочитаемых каналах:

-   Discord: один бот на агента, включите Message Content Intent, скопируйте каждый токен.
-   Telegram: один бот на агента через BotFather, скопируйте каждый токен.
-   WhatsApp: привяжите каждый номер телефона к каждой учётной записи.

```bash
openclaw channels login --channel whatsapp --account work
```

См. руководства по каналам: [Discord](../channels/discord.md), [Telegram](../channels/telegram.md), [WhatsApp](../channels/whatsapp.md).

### Шаг 3: Добавьте агентов, учётные записи и привязки

Добавьте агентов в `agents.list`, учётные записи каналов в `channels..accounts` и свяжите их с помощью `bindings` (примеры ниже).

### Шаг 4: Перезапустите и проверьте

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## Несколько агентов = несколько людей, несколько личностей

При использовании **нескольких агентов** каждый `agentId` становится **полностью изолированной персоной**:

-   **Разные номера телефонов/учётные записи** (на канал `accountId`).
-   **Разные личности** (файлы рабочего пространства на агента, такие как `AGENTS.md` и `SOUL.md`).
-   **Отдельная аутентификация + сессии** (без перекрёстных помех, если явно не включено).

Это позволяет **нескольким людям** использовать один сервер Gateway, сохраняя свои ИИ-«мозги» и данные изолированными.

## Один номер WhatsApp, несколько людей (разделение ЛС)

Вы можете направлять **разные ЛС WhatsApp** разным агентам, оставаясь на **одной учётной записи WhatsApp**. Сопоставляйте по E.164 отправителя (например, `+15551234567`) с `peer.kind: "direct"`. Ответы всё равно будут приходить с того же номера WhatsApp (без идентичности отправителя на агента). Важная деталь: прямые чаты сводятся к **основному ключу сессии** агента, поэтому для истинной изоляции требуется **один агент на человека**. Пример:

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

Примечания:

-   Контроль доступа к ЛС является **глобальным на учётную запись WhatsApp** (спаривание/белый список), а не на агента.
-   Для общих групп привяжите группу к одному агенту или используйте [Группы трансляции](../channels/broadcast-groups.md).

## Правила маршрутизации (как сообщения выбирают агента)

Привязки **детерминированы**, и **побеждает наиболее специфичная**:

1.  Совпадение `peer` (точный id ЛС/группы/канала)
2.  Совпадение `parentPeer` (наследование ветки)
3.  `guildId + roles` (маршрутизация по ролям Discord)
4.  `guildId` (Discord)
5.  `teamId` (Slack)
6.  Совпадение `accountId` для канала
7.  Совпадение на уровне канала (`accountId: "*"`)
8.  Возврат к агенту по умолчанию (`agents.list[].default`, иначе первая запись в списке, по умолчанию: `main`)

Если несколько привязок совпадают на одном уровне, побеждает первая в порядке конфигурации. Если привязка задаёт несколько полей совпадения (например, `peer` + `guildId`), все указанные поля обязательны (семантика `И`). Важная деталь области действия учётной записи:

-   Привязка, опускающая `accountId`, соответствует только учётной записи по умолчанию.
-   Используйте `accountId: "*"` для глобального возврата на канале для всех учётных записей.
-   Если позже вы добавите ту же привязку для того же агента с явным id учётной записи, OpenClaw обновит существующую привязку только для канала до области действия учётной записи вместо её дублирования.

## Несколько учётных записей / номеров телефона

Каналы, поддерживающие **несколько учётных записей** (например, WhatsApp), используют `accountId` для идентификации каждого входа. Каждый `accountId` может быть направлен к разному агенту, поэтому один сервер может размещать несколько номеров телефона без смешивания сессий. Если вы хотите, чтобы учётная запись по умолчанию на канале использовалась, когда `accountId` опущен, установите `channels..defaultAccount` (опционально). Если не установлено, OpenClaw возвращается к `default`, если он присутствует, иначе к первой настроенной учётной записи (отсортированной). Распространённые каналы, поддерживающие эту схему, включают:

-   `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
-   `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
-   `bluebubbles`, `zalo`, `zalouser`, `nostr`, `feishu`

## Концепции

-   `agentId`: один «мозг» (рабочее пространство, аутентификация на агента, хранилище сессий на агента).
-   `accountId`: один экземпляр учётной записи канала (например, учётная запись WhatsApp `"personal"` vs `"biz"`).
-   `binding`: направляет входящие сообщения к `agentId` по `(channel, accountId, peer)` и опционально guild/team id.
-   Прямые чаты сводятся к `agent::` («основной» на агента; `session.mainKey`).

## Примеры для платформ

### Боты Discord на агента

Каждая учётная запись бота Discord соответствует уникальному `accountId`. Привяжите каждую учётную запись к агенту и поддерживайте белые списки на бота.

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

Примечания:

-   Пригласите каждого бота на сервер и включите Message Content Intent.
-   Токены находятся в `channels.discord.accounts..token` (учётная запись по умолчанию может использовать `DISCORD_BOT_TOKEN`).

### Боты Telegram на агента

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

Примечания:

-   Создайте по одному боту на агента через BotFather и скопируйте каждый токен.
-   Токены находятся в `channels.telegram.accounts..botToken` (учётная запись по умолчанию может использовать `TELEGRAM_BOT_TOKEN`).

### Номера WhatsApp на агента

Привяжите каждую учётную запись перед запуском шлюза:

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Детерминированная маршрутизация: побеждает первое совпадение (самое специфичное первым).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Опциональное переопределение на конкретного собеседника (пример: отправить конкретную группу агенту work).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Выключено по умолчанию: обмен сообщениями между агентами должен быть явно включён + добавлен в белый список.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Опциональное переопределение. По умолчанию: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Опциональное переопределение. По умолчанию: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## Пример: ежедневный чат WhatsApp + глубокая работа в Telegram

Разделение по каналу: направляйте WhatsApp к быстрому повседневному агенту, а Telegram — к агенту Opus.

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Примечания:

-   Если у вас несколько учётных записей на канале, добавьте `accountId` в привязку (например, `{ channel: "whatsapp", accountId: "personal" }`).
-   Чтобы направить одно ЛС/группу к Opus, оставив остальное на chat, добавьте привязку `match.peer` для этого собеседника; совпадения по собеседнику всегда побеждают над правилами на весь канал.

## Пример: тот же канал, один собеседник к Opus

Оставьте WhatsApp на быстром агенте, но направьте одно ЛС к Opus:

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Привязки по собеседнику всегда побеждают, поэтому держите их выше правила на весь канал.

## Семейный агент, привязанный к группе WhatsApp

Привяжите выделенного семейного агента к одной группе WhatsApp с упоминанием-затвором и более строгой политикой инструментов:

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Примечания:

-   Списки разрешения/запрета инструментов относятся к **инструментам**, а не навыкам. Если навыку нужно запустить бинарный файл, убедитесь, что `exec` разрешён и бинарный файл существует в песочнице.
-   Для более строгого затвора установите `agents.list[].groupChat.mentionPatterns` и держите белые списки групп включёнными для канала.

## Песочница и конфигурация инструментов на агента

Начиная с v2026.1.6, каждый агент может иметь свою собственную песочницу и ограничения инструментов:

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // Без песочницы для личного агента
        },
        // Без ограничений инструментов - все инструменты доступны
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Всегда в песочнице
          scope: "agent",  // Один контейнер на агента
          docker: {
            // Опциональная однократная настройка после создания контейнера
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Только инструмент read
          deny: ["exec", "write", "edit", "apply_patch"],    // Запретить другие
        },
      },
    ],
  },
}
```

Примечание: `setupCommand` находится в `sandbox.docker` и запускается один раз при создании контейнера. Переопределения `sandbox.docker.*` на агента игнорируются, когда разрешённая область — `"shared"`. **Преимущества:**

-   **Изоляция безопасности**: Ограничьте инструменты для ненадёжных агентов
-   **Контроль ресурсов**: Песочница для конкретных агентов, оставляя других на хосте
-   **Гибкие политики**: Разные разрешения на агента

Примечание: `tools.elevated` является **глобальным** и основан на отправителе; он не настраивается на агента. Если вам нужны границы на агента, используйте `agents.list[].tools`, чтобы запретить `exec`. Для таргетинга групп используйте `agents.list[].groupChat.mentionPatterns`, чтобы @упоминания чётко соответствовали целевому агенту. См. [Песочница и инструменты для нескольких агентов](../tools/multi-agent-sandbox-tools.md) для подробных примеров.

[Компактизация](./compaction.md)[Присутствие](./presence.md)