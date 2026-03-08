

  Команды CLI

  
# Sandbox CLI

Управляйте контейнерами песочницы на основе Docker для изолированного выполнения агентов.

## Обзор

OpenClaw может запускать агентов в изолированных контейнерах Docker для безопасности. Команды `sandbox` помогают управлять этими контейнерами, особенно после обновлений или изменений конфигурации.

## Команды

### openclaw sandbox explain

Проверьте **эффективный** режим/область/доступ к рабочему пространству песочницы, политику инструментов песочницы и повышенные гейты (с путями ключей конфигурации для исправления).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

Вывести список всех контейнеров песочницы с их статусом и конфигурацией.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Список только контейнеров браузера
openclaw sandbox list --json     # Вывод в формате JSON
```

**В выводе содержится:**

-   Имя контейнера и статус (работает/остановлен)
-   Образ Docker и соответствует ли он конфигурации
-   Возраст (время с момента создания)
-   Время простоя (время с последнего использования)
-   Связанная сессия/агент

### openclaw sandbox recreate

Удалить контейнеры песочницы, чтобы принудительно пересоздать их с обновленными образами/конфигурацией.

```bash
openclaw sandbox recreate --all                # Пересоздать все контейнеры
openclaw sandbox recreate --session main       # Конкретная сессия
openclaw sandbox recreate --agent mybot        # Конкретный агент
openclaw sandbox recreate --browser            # Только контейнеры браузера
openclaw sandbox recreate --all --force        # Пропустить подтверждение
```

**Опции:**

-   `--all`: Пересоздать все контейнеры песочницы
-   `--session `: Пересоздать контейнер для конкретной сессии
-   `--agent `: Пересоздать контейнеры для конкретного агента
-   `--browser`: Пересоздать только контейнеры браузера
-   `--force`: Пропустить запрос подтверждения

**Важно:** Контейнеры автоматически пересоздаются при следующем использовании агента.

## Примеры использования

### После обновления образов Docker

```bash
# Загрузить новый образ
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Обновить конфигурацию для использования нового образа
# Отредактировать config: agents.defaults.sandbox.docker.image (или agents.list[].sandbox.docker.image)

# Пересоздать контейнеры
openclaw sandbox recreate --all
```

### После изменения конфигурации песочницы

```bash
# Отредактировать config: agents.defaults.sandbox.* (или agents.list[].sandbox.*)

# Пересоздать для применения новой конфигурации
openclaw sandbox recreate --all
```

### После изменения setupCommand

```bash
openclaw sandbox recreate --all
# или только для одного агента:
openclaw sandbox recreate --agent family
```

### Только для конкретного агента

```bash
# Обновить контейнеры только одного агента
openclaw sandbox recreate --agent alfred
```

## Зачем это нужно?

**Проблема:** Когда вы обновляете образы Docker или конфигурацию песочницы:

-   Существующие контейнеры продолжают работать со старыми настройками
-   Контейнеры удаляются только после 24 часов бездействия
-   Регулярно используемые агенты сохраняют старые контейнеры работающими неограниченно долго

**Решение:** Используйте `openclaw sandbox recreate`, чтобы принудительно удалить старые контейнеры. Они будут автоматически пересозданы с текущими настройками при следующей необходимости. Совет: предпочитайте `openclaw sandbox recreate` ручному `docker rm`. Это использует соглашение об именовании контейнеров Шлюза и избегает несоответствий при изменении ключей области/сессии.

## Конфигурация

Настройки песочницы находятся в `~/.openclaw/openclaw.json` в разделе `agents.defaults.sandbox` (переопределения для конкретного агента указываются в `agents.list[].sandbox`):

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... дополнительные опции Docker
        },
        "prune": {
          "idleHours": 24, // Автоудаление после 24 часов простоя
          "maxAgeDays": 7, // Автоудаление после 7 дней
        },
      },
    },
  },
}
```

## Смотрите также

-   [Документация по песочнице](../gateway/sandboxing.md)
-   [Конфигурация агента](../concepts/agent-workspace.md)
-   [Команда Doctor](../gateway/doctor.md) - Проверка настройки песочницы

[reset](./reset.md)[secrets](./secrets.md)

---