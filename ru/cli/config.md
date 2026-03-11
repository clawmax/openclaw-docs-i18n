

  Команды CLI

  
# config

Вспомогательные команды для конфигурации: получить/установить/удалить/проверить значения по пути и вывести путь к активному файлу конфигурации. Запуск без подкоманды открывает мастер настройки (аналогично `openclaw configure`).

## Примеры

```bash
openclaw config file
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
openclaw config validate
openclaw config validate --json
```

## Пути

Пути используют точечную или скобочную нотацию:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Используйте индекс списка агентов для указания на конкретного агента:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## Значения

Значения, по возможности, парсятся как JSON5; в противном случае они обрабатываются как строки. Используйте `--strict-json` для обязательного парсинга JSON5. `--json` остаётся поддерживаемым устаревшим алиасом.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

## Подкоманды

-   `config file`: Вывести путь к активному файлу конфигурации (разрешённый из `OPENCLAW_CONFIG_PATH` или расположение по умолчанию).

Перезапустите шлюз после редактирования.

## Validate

Проверить текущую конфигурацию на соответствие активной схеме без запуска шлюза.

```bash
openclaw config validate
openclaw config validate --json
```

[completion](./completion.md)[configure](./configure.md)

---