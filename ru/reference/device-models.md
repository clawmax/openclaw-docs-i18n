title: "База данных моделей устройств OpenClaw для идентификаторов устройств Apple"
description: "Узнайте, как OpenClaw сопоставляет идентификаторы устройств Apple с удобочитаемыми именами. Найдите источник данных и шаги для обновления базы данных моделей iOS и macOS."
keywords: ["идентификаторы устройств apple", "база данных моделей устройств", "идентификаторы моделей ios", "идентификаторы моделей macos", "сопоставление устройств openclaw", "модели устройств rpc", "имена устройств api", "сопоставление идентификаторов моделей apple"]
---

  RPC и API

  
# База данных моделей устройств

Приложение-компаньон для macOS отображает удобочитаемые названия моделей устройств Apple в интерфейсе **Instances**, сопоставляя идентификаторы моделей Apple (например, `iPad16,6`, `Mac16,6`) с человеко-читаемыми именами. Это сопоставление поставляется в виде JSON по пути:

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## Источник данных

В настоящее время мы используем сопоставление из репозитория с лицензией MIT:

-   `kyle-seongwoo-jun/apple-device-identifiers`

Для обеспечения детерминированности сборок файлы JSON привязаны к конкретным коммитам из исходного репозитория (записано в `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## Обновление базы данных

1.  Выберите коммиты из исходного репозитория, к которым хотите привязаться (один для iOS, один для macOS).
2.  Обновите хэши коммитов в файле `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3.  Загрузите JSON-файлы заново, привязавшись к выбранным коммитам:

```
IOS_COMMIT="<хэш коммита для ios-device-identifiers.json>"
MAC_COMMIT="<хэш коммита для mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  Убедитесь, что файл `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` по-прежнему соответствует лицензии из исходного репозитория (замените его, если лицензия изменилась).
5.  Убедитесь, что приложение для macOS собирается без ошибок (без предупреждений):

```bash
swift build --package-path apps/macos
```

[Адаптеры RPC](./rpc.md)[AGENTS.md по умолчанию](./AGENTS.default.md)