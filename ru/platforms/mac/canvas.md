title: "Документация и руководство по использованию приложения OpenClaw Canvas для macOS"
description: "Узнайте, как использовать панель OpenClaw Canvas на macOS для рабочих пространств HTML/CSS/JS и A2UI. Руководство охватывает хранение файлов, API агента, CLI-команды и безопасность."
keywords: ["openclaw canvas", "панель canvas macos", "интеграция a2ui", "api агента", "wkwebview", "рабочее пространство canvas", "документация openclaw", "cli команды canvas"]
---

  Сопутствующее приложение для macOS

  
# Canvas

Приложение для macOS встраивает управляемую агентом **панель Canvas**, используя `WKWebView`. Это легковесное визуальное рабочее пространство для HTML/CSS/JS, A2UI и небольших интерактивных UI-поверхностей.

## Где находится Canvas

Состояние Canvas хранится в Application Support:

-   `~/Library/Application Support/OpenClaw/canvas//...`

Панель Canvas обслуживает эти файлы через **пользовательскую URL-схему**:

-   `openclaw-canvas:///`

Примеры:

-   `openclaw-canvas://main/` → `/main/index.html`
-   `openclaw-canvas://main/assets/app.css` → `/main/assets/app.css`
-   `openclaw-canvas://main/widgets/todo/` → `/main/widgets/todo/index.html`

Если в корне нет `index.html`, приложение показывает **встроенную страницу-шаблон**.

## Поведение панели

-   Безрамочная, изменяемая панель, закрепленная рядом со строкой меню (или курсором мыши).
-   Запоминает размер и положение для каждой сессии.
-   Автоматически перезагружается при изменении локальных файлов canvas.
-   Одновременно видна только одна панель Canvas (сессия переключается по необходимости).

Canvas можно отключить в Настройках → **Разрешить Canvas**. Когда отключено, команды узла canvas возвращают `CANVAS_DISABLED`.

## API-поверхность агента

Canvas доступен через **WebSocket шлюза (Gateway)**, поэтому агент может:

-   показывать/скрывать панель
-   переходить по пути или URL
-   выполнять JavaScript
-   захватывать снимок изображения

Примеры CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Примечания:

-   `canvas.navigate` принимает **локальные пути canvas**, URL `http(s)` и URL `file://`.
-   Если передать `"/"`, Canvas показывает локальный шаблон или `index.html`.

## A2UI в Canvas

A2UI размещается на хосте Canvas шлюза (Gateway) и отображается внутри панели Canvas. Когда шлюз объявляет хост Canvas, приложение macOS автоматически переходит на страницу хоста A2UI при первом открытии. URL хоста A2UI по умолчанию:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### Команды A2UI (v0.8)

Canvas в настоящее время принимает сообщения сервер→клиент **A2UI v0.8**:

-   `beginRendering`
-   `surfaceUpdate`
-   `dataModelUpdate`
-   `deleteSurface`

`createSurface` (v0.9) не поддерживается. Пример CLI:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Быстрая проверка:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## Запуск агента из Canvas

Canvas может запускать новые выполнения агента через глубокие ссылки (deep links):

-   `openclaw://agent?...`

Пример (в JS):

```bash
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

Приложение запрашивает подтверждение, если не предоставлен действительный ключ.

## Примечания по безопасности

-   Схема Canvas блокирует обход директорий; файлы должны находиться в корне сессии.
-   Локальное содержимое Canvas использует пользовательскую схему (не требуется сервер локальной петли).
-   Внешние URL `http(s)` разрешены только при явной навигации.

[WebChat](./webchat.md)[Жизненный цикл шлюза (Gateway)](./child-process.md)

---