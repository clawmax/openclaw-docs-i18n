title: "Руководство по подписанию и распространению релизных сборок OpenClaw для macOS"
description: "Узнайте, как собрать, подписать, нотаризовать и опубликовать релизные версии приложения OpenClaw для macOS с автообновлениями Sparkle. Включает предварительные требования и скрипты упаковки."
keywords: ["релиз macos", "обновления sparkle", "подпись developer id", "нотаризация приложения", "генерация appcast", "распространение macos", "упаковка mac приложения", "автоматизация релиза"]
---

  Компаньон-приложение для macOS

  
# Релиз для macOS

Это приложение теперь поставляется с автообновлениями Sparkle. Релизные сборки должны быть подписаны Developer ID, заархивированы и опубликованы с подписанной записью в appcast.

## Предварительные требования

-   Установлен сертификат Developer ID Application (пример: `Developer ID Application:  ()`).
-   Путь к приватному ключу Sparkle задан в переменной окружения как `SPARKLE_PRIVATE_KEY_FILE` (путь к вашему приватному ключу ed25519 для Sparkle; публичный ключ встроен в Info.plist). Если он отсутствует, проверьте `~/.profile`.
-   Учётные данные для нотаризации (профиль Keychain или API-ключ) для `xcrun notarytool`, если требуется распространение DMG/zip, безопасное для Gatekeeper.
    -   Мы используем профиль Keychain с именем `openclaw-notary`, созданный из переменных окружения API-ключа App Store Connect в вашем профиле оболочки:
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   Установлены зависимости `pnpm` (`pnpm install --config.node-linker=hoisted`).
-   Инструменты Sparkle автоматически загружаются через SwiftPM в `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast` и т.д.).

## Сборка и упаковка

Примечания:

-   `APP_BUILD` соответствует `CFBundleVersion`/`sparkle:version`; сохраняйте его числовым и монотонным (без `-beta`), иначе Sparkle будет сравнивать его как равный.
-   Если `APP_BUILD` опущен, `scripts/package-mac-app.sh` вычисляет безопасное для Sparkle значение по умолчанию из `APP_VERSION` (`YYYYMMDDNN`: стабильные сборки используют по умолчанию `90`, пререлизы используют суффикс для определения ветки) и использует большее из этого значения и количества коммитов git.
-   Вы всё ещё можете явно переопределить `APP_BUILD`, когда для релизной инженерии требуется конкретное монотонное значение.
-   По умолчанию используется текущая архитектура (`$(uname -m)`). Для релизных/универсальных сборок установите `BUILD_ARCHS="arm64 x86_64"` (или `BUILD_ARCHS=all`).
-   Используйте `scripts/package-mac-dist.sh` для релизных артефактов (zip + DMG + нотаризация). Используйте `scripts/package-mac-app.sh` для локальной/разработческой упаковки.

```bash
# Из корня репозитория; задайте идентификаторы релиза, чтобы включить ленту Sparkle.
# APP_BUILD должен быть числовым и монотонным для сравнения в Sparkle.
# По умолчанию вычисляется автоматически из APP_VERSION, если опущено.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip для распространения (включает ресурсные вилки для поддержки дельта-обновлений Sparkle)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# Опционально: также создайте стилизованный DMG для пользователей (перетащить в /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# Рекомендуется: сборка + нотаризация/скрепление zip + DMG
# Сначала создайте профиль Keychain один раз:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Опционально: приложите dSYM к релизу
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## Запись в Appcast

Используйте генератор примечаний к релизу, чтобы Sparkle отображал форматированные HTML-примечания:

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Генерирует HTML-примечания к релизу из `CHANGELOG.md` (через [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) и встраивает их в запись appcast. Зафиксируйте обновлённый `appcast.xml` вместе с релизными артефактами (zip + dSYM) при публикации.

## Публикация и проверка

-   Загрузите `OpenClaw-2026.3.7.zip` (и `OpenClaw-2026.3.7.dSYM.zip`) в релиз GitHub для тега `v2026.3.7`.
-   Убедитесь, что сырой URL appcast соответствует встроенной ленте: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
-   Проверки:
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` возвращает 200.
    -   `curl -I ` возвращает 200 после загрузки артефактов.
    -   На предыдущей публичной сборке запустите «Проверить обновления…» на вкладке About и убедитесь, что Sparkle чисто устанавливает новую сборку.

Критерий завершения: подписанное приложение и appcast опубликованы, процесс обновления работает с более старой установленной версии, и релизные артефакты прикреплены к релизу на GitHub.

[Подпись для macOS](./signing.md)[Шлюз на macOS](./bundled-gateway.md)