

  Координация агентов

  
# Песочница и инструменты для нескольких агентов

## Обзор

Каждый агент в настройке с несколькими агентами теперь может иметь свою собственную:

-   **Конфигурацию песочницы** (`agents.list[].sandbox` переопределяет `agents.defaults.sandbox`)
-   **Ограничения инструментов** (`tools.allow` / `tools.deny`, плюс `agents.list[].tools`)

Это позволяет запускать несколько агентов с разными профилями безопасности:

-   Персональный помощник с полным доступом
-   Семейные/рабочие агенты с ограниченными инструментами
-   Публичные агенты в песочницах

`setupCommand` относится к `sandbox.docker` (глобально или для каждого агента) и выполняется один раз при создании контейнера. Аутентификация для каждого агента: каждый агент читает из своего собственного хранилища аутентификации `agentDir` по пути:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Учетные данные **не** передаются между агентами. Никогда не используйте один и тот же `agentDir` для разных агентов. Если вы хотите поделиться учетными данными, скопируйте `auth-profiles.json` в `agentDir` другого агента. Чтобы узнать, как работает песочница во время выполнения, см. [Песочница](../gateway/sandboxing.md). Для отладки вопроса "почему это заблокировано?" см. [Песочница vs Политика инструментов vs Повышенные права](../gateway/sandbox-vs-tool-policy-vs-elevated.md) и `openclaw sandbox explain`.

* * *

## Примеры конфигурации

### Пример 1: Персональный + Ограниченный семейный агент

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Персональный помощник",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Семейный бот",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Результат:**

-   Агент `main`: Работает на хосте, полный доступ к инструментам
-   Агент `family`: Работает в Docker (один контейнер на агента), только инструмент `read`

* * *

### Пример 2: Рабочий агент с общей песочницей

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### Пример 2b: Глобальный профиль для программирования + агент только для обмена сообщениями

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Результат:**

-   агенты по умолчанию получают инструменты для программирования
-   агент `support` работает только с сообщениями (+ инструмент Slack)

* * *

### Пример 3: Разные режимы песочницы для каждого агента

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // Глобальное значение по умолчанию
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // Переопределение: main никогда не помещается в песочницу
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // Переопределение: public всегда в песочнице
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## Приоритет конфигурации

Когда существуют как глобальные (`agents.defaults.*`), так и специфичные для агента (`agents.list[].*`) конфигурации:

### Конфигурация песочницы

Настройки для конкретного агента переопределяют глобальные:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Примечания:**

-   `agents.list[].sandbox.{docker,browser,prune}.*` переопределяет `agents.defaults.sandbox.{docker,browser,prune}.*` для этого агента (игнорируется, когда область действия песочницы разрешается в `"shared"`).

### Ограничения инструментов

Порядок фильтрации следующий:

1.  **Профиль инструментов** (`tools.profile` или `agents.list[].tools.profile`)
2.  **Профиль инструментов провайдера** (`tools.byProvider[provider].profile` или `agents.list[].tools.byProvider[provider].profile`)
3.  **Глобальная политика инструментов** (`tools.allow` / `tools.deny`)
4.  **Политика инструментов провайдера** (`tools.byProvider[provider].allow/deny`)
5.  **Политика инструментов для конкретного агента** (`agents.list[].tools.allow/deny`)
6.  **Политика провайдера для агента** (`agents.list[].tools.byProvider[provider].allow/deny`)
7.  **Политика инструментов песочницы** (`tools.sandbox.tools` или `agents.list[].tools.sandbox.tools`)
8.  **Политика инструментов подчиненных агентов** (`tools.subagents.tools`, если применимо)

Каждый уровень может дополнительно ограничивать инструменты, но не может вернуть инструменты, запрещенные на более ранних уровнях. Если задан `agents.list[].tools.sandbox.tools`, он заменяет `tools.sandbox.tools` для этого агента. Если задан `agents.list[].tools.profile`, он переопределяет `tools.profile` для этого агента. Ключи инструментов провайдера принимают либо `provider` (например, `google-antigravity`), либо `provider/model` (например, `openai/gpt-5.2`).

### Группы инструментов (сокращения)

Политики инструментов (глобальные, для агента, для песочницы) поддерживают записи `group:*`, которые раскрываются в несколько конкретных инструментов:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: все встроенные инструменты OpenClaw (исключает плагины провайдеров)

### Режим повышенных прав

`tools.elevated` — это глобальная базовая линия (список разрешений на основе отправителя). `agents.list[].tools.elevated` может дополнительно ограничивать повышенные права для конкретных агентов (должны разрешать оба). Паттерны снижения рисков:

-   Запретить `exec` для ненадежных агентов (`agents.list[].tools.deny: ["exec"]`)
-   Избегайте добавления отправителей, которые маршрутизируются к ограниченным агентам, в белый список
-   Отключите повышенные права глобально (`tools.elevated.enabled: false`), если вам нужна только работа в песочнице
-   Отключите повышенные права для конкретного агента (`agents.list[].tools.elevated.enabled: false`) для чувствительных профилей

* * *

## Миграция с одного агента

**До (один агент):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**После (несколько агентов с разными профилями):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Устаревшие конфигурации `agent.*` мигрируются с помощью `openclaw doctor`; в дальнейшем предпочтительнее использовать `agents.defaults` + `agents.list`.

* * *

## Примеры ограничения инструментов

### Агент только для чтения

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### Агент для безопасного выполнения (без модификации файлов)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### Агент только для общения

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## Распространенная ошибка: "non-main"

`agents.defaults.sandbox.mode: "non-main"` основан на `session.mainKey` (по умолчанию `"main"`), а не на идентификаторе агента. Сессии групп/каналов всегда получают свои собственные ключи, поэтому они рассматриваются как non-main и будут помещены в песочницу. Если вы хотите, чтобы агент никогда не помещался в песочницу, установите `agents.list[].sandbox.mode: "off"`.

* * *

## Тестирование

После настройки песочницы и инструментов для нескольких агентов:

1.  **Проверьте разрешение агента:**
    
    Копировать
    
    ```bash
    openclaw agents list --bindings
    ```
    
2.  **Проверьте контейнеры песочницы:**
    
    Копировать
    
    ```bash
    docker ps --filter "name=openclaw-sbx-"
    ```
    
3.  **Проверьте ограничения инструментов:**
    -   Отправьте сообщение, требующее использования ограниченных инструментов
    -   Убедитесь, что агент не может использовать запрещенные инструменты
4.  **Мониторьте логи:**
    
    Копировать
    
    ```bash
    tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
    ```
    

* * *

## Устранение неполадок

### Агент не помещен в песочницу, несмотря на mode: "all"

-   Проверьте, есть ли глобальная настройка `agents.defaults.sandbox.mode`, которая переопределяет её
-   Конфигурация для конкретного агента имеет приоритет, поэтому установите `agents.list[].sandbox.mode: "all"`

### Инструменты все еще доступны, несмотря на список запретов

-   Проверьте порядок фильтрации инструментов: глобальный → агент → песочница → подчиненный агент
-   Каждый уровень может только дополнительно ограничивать, но не возвращать доступ
-   Проверьте с помощью логов: `[tools] filtering tools for agent:${agentId}`

### Контейнер не изолирован для каждого агента

-   Установите `scope: "agent"` в конфигурации песочницы для конкретного агента
-   По умолчанию используется `"session"`, который создает один контейнер на сессию

* * *

## Смотрите также

-   [Маршрутизация нескольких агентов](../concepts/multi-agent.md)
-   [Конфигурация песочницы](../gateway/configuration.md#agentsdefaults-sandbox)
-   [Управление сессиями](../concepts/session.md)

[ACP Агенты](./acp-agents.md)[Создание навыков](./creating-skills.md)