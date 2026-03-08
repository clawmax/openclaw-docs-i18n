

  Aplicación complementaria para macOS

  
# Habilidades

La aplicación de macOS expone las habilidades de OpenClaw a través del gateway; no analiza las habilidades localmente.

## Fuente de datos

-   `skills.status` (gateway) devuelve todas las habilidades junto con su elegibilidad y requisitos faltantes (incluyendo bloqueos de lista de permitidos para habilidades empaquetadas).
-   Los requisitos se derivan de `metadata.openclaw.requires` en cada `SKILL.md`.

## Acciones de instalación

-   `metadata.openclaw.install` define las opciones de instalación (brew/node/go/uv).
-   La aplicación llama a `skills.install` para ejecutar los instaladores en el host del gateway.
-   El gateway expone solo un instalador preferido cuando se proporcionan múltiples (brew cuando está disponible, de lo contrario el gestor de node desde `skills.install`, por defecto npm).

## Variables de entorno / Claves API

-   La aplicación almacena las claves en `~/.openclaw/openclaw.json` bajo `skills.entries.`.
-   `skills.update` actualiza parcialmente `enabled`, `apiKey` y `env`.

## Modo remoto

-   La instalación y las actualizaciones de configuración ocurren en el host del gateway (no en el Mac local).

[IPC de macOS](./xpc.md)[Puente Peekaboo](./peekaboo.md)

---