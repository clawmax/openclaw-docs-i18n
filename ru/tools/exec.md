title: "Документация по инструменту OpenClaw Exec для выполнения shell-команд"
description: "Узнайте, как использовать инструмент OpenClaw Exec для выполнения shell-команд в foreground или background, настройки безопасности, управления подтверждениями и установки переопределений сессии."
keywords: ["openclaw exec", "выполнение shell-команд", "фоновый процесс", "безопасность sandbox", "подтверждения exec", "выполнение на хосте", "инструмент командной строки", "автоматизация рабочего пространства"]
---

  Встроенные инструменты

  
# Инструмент Exec

Запускает shell-команды в рабочем пространстве. Поддерживает выполнение в foreground + background через `process`. Если `process` запрещён, `exec` выполняется синхронно и игнорирует `yieldMs`/`background`. Фоновые сессии ограничены агентом; `process` видит только сессии от того же агента.

## Параметры

-   `command` (обязательный)
-   `workdir` (по умолчанию cwd)
-   `env` (переопределения ключ/значение)
-   `yieldMs` (по умолчанию 10000): автоматический переход в background после задержки
-   `background` (bool): немедленный переход в background
-   `timeout` (секунды, по умолчанию 1800): завершение по истечении времени
-   `pty` (bool): запускать в псевдотерминале, когда доступно (CLI только для TTY, coding agents, терминальные UI)
-   `host` (`sandbox | gateway | node`): где выполнять
-   `security` (`deny | allowlist | full`): режим принудительного применения для `gateway`/`node`
-   `ask` (`off | on-miss | always`): запросы подтверждения для `gateway`/`node`
-   `node` (строка): id/имя узла для `host=node`
-   `elevated` (bool): запросить привилегированный режим (хост gateway); `security=full` принудительно устанавливается только когда elevated разрешается в `full`

Примечания:

-   `host` по умолчанию `sandbox`.
-   `elevated` игнорируется, когда sandboxing выключен (exec уже выполняется на хосте).
-   Подтверждения для `gateway`/`node` контролируются файлом `~/.openclaw/exec-approvals.json`.
-   `node` требует подключённого узла (приложение companion или headless node host).
-   Если доступно несколько узлов, установите `exec.node` или `tools.exec.node` для выбора одного.
-   На хостах не-Windows exec использует `SHELL`, если он установлен; если `SHELL` — `fish`, он предпочитает `bash` (или `sh`) из `PATH`, чтобы избежать несовместимых с fish скриптов, затем возвращается к `SHELL`, если ни один из них не существует.
-   На хостах Windows exec предпочитает обнаружение PowerShell 7 (`pwsh`) (Program Files, ProgramW6432, затем PATH), затем возвращается к Windows PowerShell 5.1.
-   Выполнение на хосте (`gateway`/`node`) отклоняет `env.PATH` и переопределения загрузчика (`LD_*`/`DYLD_*`), чтобы предотвратить подмену бинарников или внедрение кода.
-   OpenClaw устанавливает `OPENCLAW_SHELL=exec` в окружении запускаемой команды (включая PTY и выполнение в sandbox), чтобы правила shell/профиля могли обнаруживать контекст инструмента exec.
-   Важно: sandboxing **по умолчанию выключен**. Если sandboxing выключен и явно настроен/запрошен `host=sandbox`, exec теперь завершается с ошибкой (fail closed) вместо тихого выполнения на хосте gateway. Включите sandboxing или используйте `host=gateway` с подтверждениями.
-   Предварительные проверки скриптов (для распространённых ошибок синтаксиса shell в Python/Node) проверяют только файлы внутри границы эффективного `workdir`. Если путь к скрипту разрешается вне `workdir`, предварительная проверка для этого файла пропускается.

## Конфигурация

-   `tools.exec.notifyOnExit` (по умолчанию: true): если true, фоновые сессии exec ставят в очередь системное событие и запрашивают heartbeat при выходе.
-   `tools.exec.approvalRunningNoticeMs` (по умолчанию: 10000): отправляет одно уведомление «running», когда exec, требующий подтверждения, выполняется дольше этого времени (0 отключает).
-   `tools.exec.host` (по умолчанию: `sandbox`)
-   `tools.exec.security` (по умолчанию: `deny` для sandbox, `allowlist` для gateway + node, если не установлено)
-   `tools.exec.ask` (по умолчанию: `on-miss`)
-   `tools.exec.node` (по умолчанию: не установлено)
-   `tools.exec.pathPrepend`: список директорий для добавления в начало `PATH` для запусков exec (только gateway + sandbox).
-   `tools.exec.safeBins`: безопасные бинарники только для stdin, которые могут выполняться без явных записей в allowlist. Подробности поведения см. в [Safe bins](./exec-approvals.md#safe-bins-stdin-only).
-   `tools.exec.safeBinTrustedDirs`: дополнительные явно указанные директории, которым доверяют для проверок путей `safeBins`. Элементы `PATH` никогда не доверяются автоматически. Встроенные значения по умолчанию: `/bin` и `/usr/bin`.
-   `tools.exec.safeBinProfiles`: опциональная пользовательская политика argv для каждого безопасного бинарника (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

Пример:

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### Обработка PATH

-   `host=gateway`: объединяет `PATH` вашего login-shell в окружение exec. Переопределения `env.PATH` отклоняются для выполнения на хосте. Сам демон всё ещё работает с минимальным `PATH`:
    -   macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
    -   Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
-   `host=sandbox`: запускает `sh -lc` (login shell) внутри контейнера, поэтому `/etc/profile` может сбросить `PATH`. OpenClaw добавляет `env.PATH` после загрузки профиля через внутреннюю переменную окружения (без интерполяции shell); `tools.exec.pathPrepend` также применяется здесь.
-   `host=node`: только неперекрытые переопределения env, которые вы передаёте, отправляются на узел. Переопределения `env.PATH` отклоняются для выполнения на хосте и игнорируются узлами. Если вам нужны дополнительные записи PATH на узле, настройте окружение службы узла (systemd/launchd) или установите инструменты в стандартные расположения.

Привязка узла к агенту (используйте индекс в списке агентов в конфигурации):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

UI управления: вкладка Nodes включает небольшую панель «Exec node binding» для тех же настроек.

## Переопределения сессии (/exec)

Используйте `/exec` для установки **значений по умолчанию для сессии** для `host`, `security`, `ask` и `node`. Отправьте `/exec` без аргументов, чтобы показать текущие значения. Пример:

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Модель авторизации

`/exec` учитывается только для **авторизованных отправителей** (allowlists каналов/спаривание плюс `commands.useAccessGroups`). Он обновляет **только состояние сессии** и не записывает конфигурацию. Чтобы полностью отключить exec, запретите его через политику инструментов (`tools.deny: ["exec"]` или на агента). Подтверждения хоста всё ещё применяются, если вы явно не установите `security=full` и `ask=off`.

## Подтверждения Exec (companion app / node host)

Агенты в sandbox могут требовать подтверждения на запрос перед запуском `exec` на хосте gateway или узле. См. [Exec approvals](./exec-approvals.md) для политики, allowlist и UI потока. Когда требуются подтверждения, инструмент exec возвращается немедленно со статусом `status: "approval-pending"` и id подтверждения. После одобрения (или отказа / тайм-аута) Gateway отправляет системные события (`Exec finished` / `Exec denied`). Если команда всё ещё выполняется после `tools.exec.approvalRunningNoticeMs`, отправляется одно уведомление `Exec running`.

## Allowlist + безопасные бинарники

Принудительное применение ручного allowlist сопоставляет **только разрешённые пути к бинарникам** (без сопоставления по базовому имени). Когда `security=allowlist`, shell-команды автоматически разрешаются только если каждый сегмент пайплайна находится в allowlist или является безопасным бинарником. Цепочки (`;`, `&&`, `||`) и перенаправления отклоняются в режиме allowlist, если каждый сегмент верхнего уровня не удовлетворяет allowlist (включая безопасные бинарники). Перенаправления остаются неподдерживаемыми. `autoAllowSkills` — это отдельный удобный путь в подтверждениях exec. Он не то же самое, что ручные записи в allowlist путей. Для строгого явного доверия держите `autoAllowSkills` отключённым. Используйте два контрола для разных задач:

-   `tools.exec.safeBins`: небольшие фильтры потоков только для stdin.
-   `tools.exec.safeBinTrustedDirs`: явные дополнительные доверенные директории для путей исполняемых файлов safe-bin.
-   `tools.exec.safeBinProfiles`: явная политика argv для пользовательских безопасных бинарников.
-   allowlist: явное доверие для путей исполняемых файлов.

Не рассматривайте `safeBins` как общий allowlist и не добавляйте бинарники интерпретаторов/сред выполнения (например, `python3`, `node`, `ruby`, `bash`). Если они вам нужны, используйте явные записи в allowlist и держите запросы подтверждения включёнными. `openclaw security audit` предупреждает, когда записи `safeBins` для интерпретаторов/сред выполнения отсутствуют в явных профилях, и `openclaw doctor --fix` может создать недостающие пользовательские записи `safeBinProfiles`. Полные детали политики и примеры см. в [Exec approvals](./exec-approvals.md#safe-bins-stdin-only) и [Safe bins versus allowlist](./exec-approvals.md#safe-bins-versus-allowlist).

## Примеры

Foreground:

```json
{ "tool": "exec", "command": "ls -la" }
```

Background + опрос:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Отправка клавиш (в стиле tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Submit (отправка только CR):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Paste (по умолчанию в скобках):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (экспериментальный)

`apply_patch` — это под-инструмент `exec` для структурированного редактирования нескольких файлов. Включите его явно:

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Примечания:

-   Доступно только для моделей OpenAI/OpenAI Codex.
-   Политика инструментов всё ещё применяется; `allow: ["exec"]` неявно разрешает `apply_patch`.
-   Конфигурация находится в `tools.exec.applyPatch`.
-   `tools.exec.applyPatch.workspaceOnly` по умолчанию `true` (ограничено рабочим пространством). Установите `false` только если вы намеренно хотите, чтобы `apply_patch` записывал/удалял вне директории рабочего пространства.

[Привилегированный режим](./elevated.md)[Подтверждения Exec](./exec-approvals.md)