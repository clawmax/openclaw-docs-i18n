title: "Контрольный список выпуска OpenClaw и руководство по публикации версий"
description: "Пошаговое руководство по выпуску и публикации версий OpenClaw. Узнайте о предварительных проверках, процессе сборки, публикации в npm и распространении приложения для macOS."
keywords: ["выпуск openclaw", "контрольный список выпуска", "публикация npm", "версионирование", "sparkle appcast", "распространение приложения macos", "список изменений", "публикация плагинов"]
---

  Примечания к выпуску

  
# Контрольный список выпуска

Используйте `pnpm` (Node 22+) из корня репозитория. Перед тегированием/публикацией убедитесь, что рабочее дерево чистое.

## Триггер оператора

Когда оператор говорит «выпустить», немедленно выполните эту предварительную проверку (без лишних вопросов, если нет блокировок):

-   Прочтите этот документ и `docs/platforms/mac/release.md`.
-   Загрузите переменные окружения из `~/.profile` и убедитесь, что `SPARKLE_PRIVATE_KEY_FILE` + переменные App Store Connect установлены (SPARKLE\_PRIVATE\_KEY\_FILE должен находиться в `~/.profile`).
-   При необходимости используйте ключи Sparkle из `~/Library/CloudStorage/Dropbox/Backup/Sparkle`.

1.  **Версия и метаданные**

-   [ ]  Увеличьте версию в `package.json` (например, `2026.1.29`).
-   [ ]  Запустите `pnpm plugins:sync`, чтобы синхронизировать версии пакетов расширений и списки изменений.
-   [ ]  Обновите строки CLI/версии в [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) и пользовательский агент Baileys в [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
-   [ ]  Подтвердите метаданные пакета (name, description, repository, keywords, license) и что `bin` указывает на [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) для `openclaw`.
-   [ ]  Если зависимости изменились, запустите `pnpm install`, чтобы `pnpm-lock.yaml` был актуальным.

2.  **Сборка и артефакты**

-   [ ]  Если изменились входные данные A2UI, запустите `pnpm canvas:a2ui:bundle` и закоммитьте обновленный [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
-   [ ]  `pnpm run build` (перегенерирует `dist/`).
-   [ ]  Убедитесь, что npm-пакет `files` включает все необходимые папки `dist/*` (особенно `dist/node-host/**` и `dist/acp/**` для headless node + ACP CLI).
-   [ ]  Подтвердите, что `dist/build-info.json` существует и содержит ожидаемый хэш `commit` (баннер CLI использует это для установок через npm).
-   [ ]  Опционально: `npm pack --pack-destination /tmp` после сборки; проверьте содержимое tarball и держите его под рукой для выпуска на GitHub (**не** коммитьте его).

3.  **Список изменений и документация**

-   [ ]  Обновите `CHANGELOG.md` с пользовательскими ключевыми моментами (создайте файл, если отсутствует); сохраняйте записи строго в порядке убывания версий.
-   [ ]  Убедитесь, что примеры/флаги в README соответствуют текущему поведению CLI (особенно новым командам или опциям).

4.  **Проверка**

-   [ ]  `pnpm build`
-   [ ]  `pnpm check`
-   [ ]  `pnpm test` (или `pnpm test:coverage`, если нужен вывод покрытия)
-   [ ]  `pnpm release:check` (проверяет содержимое npm pack)
-   [ ]  `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (Docker smoke-тест установки, быстрый путь; обязателен перед выпуском)
    -   Если известно, что предыдущий npm-релиз сломан, установите `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<последняя-рабочая-версия>` или `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` для шага предустановки.
-   [ ]  (Опционально) Полный smoke-тест установщика (добавляет покрытие non-root + CLI): `pnpm test:install:smoke`
-   [ ]  (Опционально) E2E-тест установщика (Docker, запускает `curl -fsSL https://openclaw.ai/install.sh | bash`, выполняет онбординг, затем запускает реальные вызовы инструментов):
    -   `pnpm test:install:e2e:openai` (требуется `OPENAI_API_KEY`)
    -   `pnpm test:install:e2e:anthropic` (требуется `ANTHROPIC_API_KEY`)
    -   `pnpm test:install:e2e` (требуются оба ключа; запускает обоих провайдеров)
-   [ ]  (Опционально) Выборочно проверьте веб-шлюз, если ваши изменения затрагивают пути отправки/получения.

5.  **Приложение для macOS (Sparkle)**

-   [ ]  Соберите + подпишите приложение для macOS, затем запакуйте его в zip для распространения.
-   [ ]  Сгенерируйте Sparkle appcast (HTML-заметки через [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) и обновите `appcast.xml`.
-   [ ]  Держите zip-архив приложения (и опциональный dSYM zip) готовыми для прикрепления к выпуску на GitHub.
-   [ ]  Следуйте [инструкции по выпуску для macOS](../platforms/mac/release.md) для точных команд и необходимых переменных окружения.
    -   `APP_BUILD` должен быть числовым и монотонным (без `-beta`), чтобы Sparkle корректно сравнивал версии.
    -   При нотаризации используйте профиль связки ключей `openclaw-notary`, созданный из переменных окружения API App Store Connect (см. [macOS release](../platforms/mac/release.md)).

6.  **Публикация (npm)**

-   [ ]  Убедитесь, что статус git чист; при необходимости закоммитьте и запушьте.
-   [ ]  `npm login` (проверьте 2FA), если нужно.
-   [ ]  `npm publish --access public` (используйте `--tag beta` для пре-релизов).
-   [ ]  Проверьте реестр: `npm view openclaw version`, `npm view openclaw dist-tags` и `npx -y openclaw@X.Y.Z --version` (или `--help`).

### Устранение неполадок (заметки с выпуска 2.0.0-beta2)

-   **`npm pack/publish` зависает или создает огромный tarball**: бандл приложения macOS в `dist/OpenClaw.app` (и релизные zip-архивы) попадают в пакет. Исправьте, указав разрешенное содержимое для публикации через `files` в `package.json` (включите подкаталоги dist, docs, skills; исключите бандлы приложений). Подтвердите с помощью `npm pack --dry-run`, что `dist/OpenClaw.app` не указан.
-   **Цикл веб-аутентификации npm для dist-tags**: используйте устаревшую аутентификацию для получения запроса OTP:
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **Проверка `npx` завершается с `ECOMPROMISED: Lock compromised`**: повторите с чистым кэшем:
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **Тег нужно переназначить после позднего исправления**: принудительно обновите и запушьте тег, затем убедитесь, что ассеты выпуска на GitHub все еще соответствуют:
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **Выпуск на GitHub + appcast**

-   [ ]  Создайте и запушьте тег: `git tag vX.Y.Z && git push origin vX.Y.Z` (или `git push --tags`).
-   [ ]  Создайте/обновите выпуск на GitHub для `vX.Y.Z` с **заголовком `openclaw X.Y.Z`** (а не просто тегом); тело должно содержать **полный** раздел списка изменений для этой версии (Основное + Изменения + Исправления), встроенный (не просто ссылки), и **не должно повторять заголовок внутри тела**.
-   [ ]  Прикрепите артефакты: tarball `npm pack` (опционально), `OpenClaw-X.Y.Z.zip` и `OpenClaw-X.Y.Z.dSYM.zip` (если сгенерирован).
-   [ ]  Закоммитьте обновленный `appcast.xml` и запушьте его (Sparkle берет данные из main).
-   [ ]  Из чистого временного каталога (без `package.json`) запустите `npx -y openclaw@X.Y.Z send --help`, чтобы подтвердить работу точек входа установки/CLI.
-   [ ]  Анонсируйте/поделитесь примечаниями к выпуску.

## Область публикации плагинов (npm)

Мы публикуем **только существующие npm-плагины** под областью `@openclaw/*`. Встроенные плагины, которых нет в npm, остаются **только на диске** (все еще поставляются в `extensions/**`). Процесс для получения списка:

1.  `npm search @openclaw --json` и запишите имена пакетов.
2.  Сравните с именами в `extensions/*/package.json`.
3.  Публикуйте только **пересечение** (уже находящееся в npm).

Текущий список npm-плагинов (обновляйте по мере необходимости):

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

Примечания к выпуску также должны указывать **новые опциональные встроенные плагины**, которые **не включены по умолчанию** (пример: `tlon`).

[Благодарности](./credits.md)[Тесты](./test.md)