

  Команды CLI

  
# hooks

Управление хуками агента (автоматизации на основе событий для команд, таких как `/new`, `/reset`, и запуск шлюза). Связанное:

-   Хуки: [Хуки](../automation/hooks.md)
-   Хуки плагинов: [Плагины](../tools/plugin.md#plugin-hooks)

## Список всех хуков

```bash
openclaw hooks list
```

Вывести список всех обнаруженных хуков из рабочих, управляемых и встроенных директорий. **Опции:**

-   `--eligible`: Показать только подходящие хуки (требования выполнены)
-   `--json`: Вывод в формате JSON
-   `-v, --verbose`: Показать подробную информацию, включая отсутствующие требования

**Пример вывода:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Запуск BOOT.md при старте шлюза
  📎 bootstrap-extra-files ✓ - Внедрение дополнительных файлов начальной загрузки рабочей области во время начальной загрузки агента
  📝 command-logger ✓ - Логирование всех событий команд в централизованный файл аудита
  💾 session-memory ✓ - Сохранение контекста сессии в память при выполнении команды /new
```

**Пример (подробный):**

```bash
openclaw hooks list --verbose
```

Показывает отсутствующие требования для неподходящих хуков. **Пример (JSON):**

```bash
openclaw hooks list --json
```

Возвращает структурированный JSON для программного использования.

## Получить информацию о хуке

```bash
openclaw hooks info <name>
```

Показать подробную информацию о конкретном хуке. **Аргументы:**

-   ``: Имя хука (например, `session-memory`)

**Опции:**

-   `--json`: Вывод в формате JSON

**Пример:**

```bash
openclaw hooks info session-memory
```

**Вывод:**

```
💾 session-memory ✓ Ready

Сохранение контекста сессии в память при выполнении команды /new

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/automation/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Проверить пригодность хуков

```bash
openclaw hooks check
```

Показать сводку статуса пригодности хуков (сколько готово vs. не готово). **Опции:**

-   `--json`: Вывод в формате JSON

**Пример вывода:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Включить хук

```bash
openclaw hooks enable <name>
```

Включить конкретный хук, добавив его в ваш конфиг (`~/.openclaw/config.json`). **Примечание:** Хуки, управляемые плагинами, показывают `plugin:` в `openclaw hooks list` и не могут быть включены/отключены здесь. Включите/отключите плагин вместо этого. **Аргументы:**

-   ``: Имя хука (например, `session-memory`)

**Пример:**

```bash
openclaw hooks enable session-memory
```

**Вывод:**

```
✓ Включен хук: 💾 session-memory
```

**Что делает:**

-   Проверяет, существует ли хук и подходит ли он
-   Обновляет `hooks.internal.entries..enabled = true` в вашем конфиге
-   Сохраняет конфиг на диск

**После включения:**

-   Перезапустите шлюз, чтобы хуки перезагрузились (перезапуск приложения в строке меню на macOS или перезапуск процесса шлюза в разработке).

## Отключить хук

```bash
openclaw hooks disable <name>
```

Отключить конкретный хук, обновив ваш конфиг. **Аргументы:**

-   ``: Имя хука (например, `command-logger`)

**Пример:**

```bash
openclaw hooks disable command-logger
```

**Вывод:**

```
⏸ Отключен хук: 📝 command-logger
```

**После отключения:**

-   Перезапустите шлюз, чтобы хуки перезагрузились

## Установить хуки

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

Установить пакет хуков из локальной папки/архива или npm. Спецификации npm — **только из реестра** (имя пакета + опционально **точная версия** или **dist-tag**). Спецификации Git/URL/файл и диапазоны semver отклоняются. Установка зависимостей выполняется с `--ignore-scripts` для безопасности. Простые спецификации и `@latest` остаются на стабильной ветке. Если npm разрешает любую из них в предрелизную версию, OpenClaw останавливается и просит вас явно согласиться с помощью тега предрелиза, такого как `@beta`/`@rc`, или точной предрелизной версии. **Что делает:**

-   Копирует пакет хуков в `~/.openclaw/hooks/`
-   Включает установленные хуки в `hooks.internal.entries.*`
-   Записывает установку в `hooks.internal.installs`

**Опции:**

-   `-l, --link`: Связать локальную директорию вместо копирования (добавляет её в `hooks.internal.load.extraDirs`)
-   `--pin`: Записывать npm-установки как точные разрешённые `name@version` в `hooks.internal.installs`

**Поддерживаемые архивы:** `.zip`, `.tgz`, `.tar.gz`, `.tar` **Примеры:**

```bash
# Локальная директория
openclaw hooks install ./my-hook-pack

# Локальный архив
openclaw hooks install ./my-hook-pack.zip

# Пакет NPM
openclaw hooks install @openclaw/my-hook-pack

# Связать локальную директорию без копирования
openclaw hooks install -l ./my-hook-pack
```

## Обновить хуки

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Обновить установленные пакеты хуков (только npm-установки). **Опции:**

-   `--all`: Обновить все отслеживаемые пакеты хуков
-   `--dry-run`: Показать, что изменится, без записи

Когда сохранённый хеш целостности существует и хеш полученного артефакта изменяется, OpenClaw выводит предупреждение и запрашивает подтверждение перед продолжением. Используйте глобальный `--yes`, чтобы обойти запросы в CI/неинтерактивных запусках.

## Встроенные хуки

### session-memory

Сохраняет контекст сессии в память при выполнении команды `/new`. **Включить:**

```bash
openclaw hooks enable session-memory
```

**Вывод:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **См.:** [документация session-memory](../automation/hooks.md#session-memory)

### bootstrap-extra-files

Внедряет дополнительные файлы начальной загрузки (например, локальные для монорепозитория `AGENTS.md` / `TOOLS.md`) во время `agent:bootstrap`. **Включить:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**См.:** [документация bootstrap-extra-files](../automation/hooks.md#bootstrap-extra-files)

### command-logger

Логирует все события команд в централизованный файл аудита. **Включить:**

```bash
openclaw hooks enable command-logger
```

**Вывод:** `~/.openclaw/logs/commands.log` **Просмотр логов:**

```bash
# Последние команды
tail -n 20 ~/.openclaw/logs/commands.log

# Красивый вывод
cat ~/.openclaw/logs/commands.log | jq .

# Фильтр по действию
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**См.:** [документация command-logger](../automation/hooks.md#command-logger)

### boot-md

Запускает `BOOT.md` при старте шлюза (после запуска каналов). **События**: `gateway:startup` **Включить**:

```bash
openclaw hooks enable boot-md
```

**См.:** [документация boot-md](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)