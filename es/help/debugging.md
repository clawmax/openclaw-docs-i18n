

  Entorno y depuración

  
# Depuración

Esta página cubre ayudas de depuración para la salida en streaming, especialmente cuando un proveedor mezcla razonamiento con texto normal.

## Anulaciones de depuración en tiempo de ejecución

Usa `/debug` en el chat para establecer anulaciones de configuración **solo en tiempo de ejecución** (en memoria, no en disco). `/debug` está deshabilitado por defecto; habilítalo con `commands.debug: true`. Esto es útil cuando necesitas alternar configuraciones oscuras sin editar `openclaw.json`. Ejemplos:

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` borra todas las anulaciones y vuelve a la configuración en disco.

## Modo de observación del Gateway

Para una iteración rápida, ejecuta el gateway bajo el observador de archivos:

```bash
pnpm gateway:watch
```

Esto se corresponde con:

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

Añade cualquier bandera CLI del gateway después de `gateway:watch` y se pasarán en cada reinicio.

## Perfil de desarrollo + gateway de desarrollo (—dev)

Usa el perfil de desarrollo para aislar el estado y crear una configuración segura y desechable para depurar. Hay **dos** banderas `--dev`:

-   **`--dev` global (perfil):** aísla el estado bajo `~/.openclaw-dev` y establece el puerto del gateway por defecto en `19001` (los puertos derivados se desplazan con él).
-   **`gateway --dev`: le dice al Gateway que cree automáticamente una configuración y espacio de trabajo por defecto** cuando falten (y omita BOOTSTRAP.md).

Flujo recomendado (perfil de desarrollo + arranque de desarrollo):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Si aún no tienes una instalación global, ejecuta la CLI mediante `pnpm openclaw ...`. Lo que esto hace:

1.  **Aislamiento del perfil** (`--dev` global)
    -   `OPENCLAW_PROFILE=dev`
    -   `OPENCLAW_STATE_DIR=~/.openclaw-dev`
    -   `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
    -   `OPENCLAW_GATEWAY_PORT=19001` (navegador/canvas se desplazan en consecuencia)
2.  **Arranque de desarrollo** (`gateway --dev`)
    -   Escribe una configuración mínima si falta (`gateway.mode=local`, enlaza a loopback).
    -   Establece `agent.workspace` al espacio de trabajo de desarrollo.
    -   Establece `agent.skipBootstrap=true` (sin BOOTSTRAP.md).
    -   Si faltan, siembra los archivos del espacio de trabajo: `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
    -   Identidad por defecto: **C3‑PO** (droide de protocolo).
    -   Omite los proveedores de canales en modo desarrollo (`OPENCLAW_SKIP_CHANNELS=1`).

Flujo de reinicio (comienzo desde cero):

```bash
pnpm gateway:dev:reset
```

Nota: `--dev` es una bandera de perfil **global** y algunos ejecutores la consumen. Si necesitas especificarla explícitamente, usa la forma de variable de entorno:

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` borra la configuración, credenciales, sesiones y el espacio de trabajo de desarrollo (usando `trash`, no `rm`), luego recrea la configuración de desarrollo por defecto. Consejo: si ya hay un gateway no‑de‑desarrollo en ejecución (launchd/systemd), detenlo primero:

```bash
openclaw gateway stop
```

## Registro de flujo sin procesar (OpenClaw)

OpenClaw puede registrar el **flujo sin procesar del asistente** antes de cualquier filtrado/formateo. Esta es la mejor manera de ver si el razonamiento llega como deltas de texto plano (o como bloques de pensamiento separados). Habilítalo mediante CLI:

```bash
pnpm gateway:watch --raw-stream
```

Anulación de ruta opcional:

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variables de entorno equivalentes:

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Archivo por defecto: `~/.openclaw/logs/raw-stream.jsonl`

## Registro de fragmentos sin procesar (pi-mono)

Para capturar **fragmentos crudos compatibles con OpenAI** antes de que se analicen en bloques, pi-mono expone un registrador separado:

```
PI_RAW_STREAM=1
```

Ruta opcional:

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Archivo por defecto: `~/.pi-mono/logs/raw-openai-completions.jsonl`

> Nota: esto solo lo emiten los procesos que usan el proveedor `openai-completions` de pi-mono.

## Notas de seguridad

-   Los registros de flujo sin procesar pueden incluir prompts completos, salida de herramientas y datos del usuario.
-   Mantén los registros locales y elimínalos después de depurar.
-   Si compartes registros, elimina primero los secretos y la información personal identificable (PII).

[Variables de Entorno](./environment.md)[Pruebas](./testing.md)

---