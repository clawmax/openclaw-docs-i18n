

  Медиа и устройства

  
# Устранение неполадок узлов

Используйте эту страницу, когда узел виден в статусе, но инструменты узла не работают.

## Лестница команд

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Затем выполните проверки, специфичные для узла:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Признаки исправной работы:

-   Узел подключен и сопряжен для роли `node`.
-   `nodes describe` включает возможность, которую вы вызываете.
-   Одобрения выполнения показывают ожидаемый режим/белый список.

## Требования к работе в фоне

`canvas.*`, `camera.*` и `screen.*` работают только в фоновом режиме на узлах iOS/Android. Быстрая проверка и исправление:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

Если вы видите `NODE_BACKGROUND_UNAVAILABLE`, переведите приложение узла на передний план и повторите попытку.

## Матрица разрешений

| Возможность | iOS | Android | Приложение узла macOS | Типичный код ошибки |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | Камера (+ микрофон для аудио в клипе) | Камера (+ микрофон для аудио в клипе) | Камера (+ микрофон для аудио в клипе) | `*_PERMISSION_REQUIRED` |
| `screen.record` | Запись экрана (+ микрофон опционально) | Запрос на захват экрана (+ микрофон опционально) | Запись экрана | `*_PERMISSION_REQUIRED` |
| `location.get` | При использовании или Всегда (зависит от режима) | Определение местоположения на переднем/заднем плане в зависимости от режима | Разрешение на определение местоположения | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | н/д (путь к хосту узла) | н/д (путь к хосту узла) | Требуются одобрения выполнения | `SYSTEM_RUN_DENIED` |

## Сопряжение устройств vs одобрения выполнения

Это разные этапы контроля:

1.  **Сопряжение устройств**: может ли этот узел подключиться к шлюзу?
2.  **Одобрения выполнения**: может ли этот узел выполнить конкретную shell-команду?

Быстрые проверки:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

Если сопряжение отсутствует, сначала одобрите устройство узла. Если сопряжение в порядке, но `system.run` не работает, исправьте одобрения выполнения/белый список.

## Распространенные коды ошибок узла

-   `NODE_BACKGROUND_UNAVAILABLE` → приложение работает в фоне; переведите его на передний план.
-   `CAMERA_DISABLED` → переключатель камеры отключен в настройках узла.
-   `*_PERMISSION_REQUIRED` → разрешение ОС отсутствует/отклонено.
-   `LOCATION_DISABLED` → режим определения местоположения выключен.
-   `LOCATION_PERMISSION_REQUIRED` → запрошенный режим определения местоположения не предоставлен.
-   `LOCATION_BACKGROUND_UNAVAILABLE` → приложение работает в фоне, но есть только разрешение "При использовании".
-   `SYSTEM_RUN_DENIED: approval required` → запрос на выполнение требует явного одобрения.
-   `SYSTEM_RUN_DENIED: allowlist miss` → команда заблокирована режимом белого списка. На хостах узлов Windows формы оболочки-обертки, такие как `cmd.exe /c ...`, рассматриваются как промахи белого списка в режиме белого списка, если они не одобрены через процесс запроса.

## Цикл быстрого восстановления

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Если проблема не устранена:

-   Повторно одобрите сопряжение устройства.
-   Снова откройте приложение узла (на переднем плане).
-   Повторно предоставьте разрешения ОС.
-   Пересоздайте/скорректируйте политику одобрения выполнения.

Связанные разделы:

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[Узлы](../nodes.md)[Анализ медиа](./media-understanding.md)