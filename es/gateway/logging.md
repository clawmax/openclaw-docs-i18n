

  Configuración y operaciones

  
# Registro (Logging)

Para una descripción general orientada al usuario (CLI + Control UI + configuración), consulta [/logging](../logging.md). OpenClaw tiene dos "superficies" de registro:

-   **Salida de consola** (lo que ves en la terminal / Debug UI).
-   **Logs de archivo** (líneas JSON) escritos por el logger del gateway.

## Logger basado en archivo

-   El archivo de log rotativo por defecto está en `/tmp/openclaw/` (un archivo por día): `openclaw-YYYY-MM-DD.log`
    -   La fecha usa la zona horaria local del host del gateway.
-   La ruta del archivo de log y el nivel se pueden configurar mediante `~/.openclaw/openclaw.json`:
    -   `logging.file`
    -   `logging.level`

El formato del archivo es un objeto JSON por línea. La pestaña Logs de la Control UI sigue este archivo a través del gateway (`logs.tail`). La CLI puede hacer lo mismo:

```bash
openclaw logs --follow
```

**Verboso vs. niveles de log**

-   **Los logs de archivo** se controlan exclusivamente mediante `logging.level`.
-   `--verbose` solo afecta la **verbosidad de la consola** (y el estilo de los logs WS); **no** eleva el nivel de log del archivo.
-   Para capturar detalles solo verbosos en los logs de archivo, establece `logging.level` en `debug` o `trace`.

## Captura de consola

La CLI captura `console.log/info/warn/error/debug/trace` y los escribe en los logs de archivo, mientras aún los imprime en stdout/stderr. Puedes ajustar la verbosidad de la consola de forma independiente mediante:

-   `logging.consoleLevel` (por defecto `info`)
-   `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Redacción de resúmenes de herramientas

Los resúmenes verbosos de herramientas (ej. `🛠️ Exec: ...`) pueden enmascarar tokens sensibles antes de llegar al flujo de la consola. Esto es **solo para herramientas** y no altera los logs de archivo.

-   `logging.redactSensitive`: `off` | `tools` (por defecto: `tools`)
-   `logging.redactPatterns`: array de cadenas regex (anula los valores por defecto)
    -   Usa cadenas regex crudas (auto `gi`), o `/patrón/flags` si necesitas flags personalizados.
    -   Las coincidencias se enmascaran manteniendo los primeros 6 + últimos 4 caracteres (longitud >= 18), de lo contrario `***`.
    -   Los valores por defecto cubren asignaciones de claves comunes, flags CLI, campos JSON, encabezados bearer, bloques PEM y prefijos de token populares.

## Logs de WebSocket del gateway

El gateway imprime logs del protocolo WebSocket en dos modos:

-   **Modo normal (sin `--verbose`)**: solo se imprimen resultados RPC "interesantes":
    -   errores (`ok=false`)
    -   llamadas lentas (umbral por defecto: `>= 50ms`)
    -   errores de análisis
-   **Modo verboso (`--verbose`)**: imprime todo el tráfico de solicitud/respuesta WS.

### Estilo de log WS

`openclaw gateway` soporta un interruptor de estilo por gateway:

-   `--ws-log auto` (por defecto): el modo normal está optimizado; el modo verboso usa salida compacta
-   `--ws-log compact`: salida compacta (solicitud/respuesta emparejada) cuando es verboso
-   `--ws-log full`: salida completa por frame cuando es verboso
-   `--compact`: alias para `--ws-log compact`

Ejemplos:

```bash
# optimizado (solo errores/lentos)
openclaw gateway

# mostrar todo el tráfico WS (emparejado)
openclaw gateway --verbose --ws-log compact

# mostrar todo el tráfico WS (meta completa)
openclaw gateway --verbose --ws-log full
```

## Formateo de consola (registro por subsistema)

El formateador de consola es **consciente de TTY** e imprime líneas consistentes con prefijo. Los loggers de subsistema mantienen la salida agrupada y escaneable. Comportamiento:

-   **Prefijos de subsistema** en cada línea (ej. `[gateway]`, `[canvas]`, `[tailscale]`)
-   **Colores de subsistema** (estables por subsistema) más coloración por nivel
-   **Color cuando la salida es una TTY o el entorno parece una terminal enriquecida** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respeta `NO_COLOR`
-   **Prefijos de subsistema acortados**: elimina el `gateway/` inicial + `channels/`, mantiene los últimos 2 segmentos (ej. `whatsapp/outbound`)
-   **Sub-loggers por subsistema** (prefijo automático + campo estructurado `{ subsystem }`)
-   **`logRaw()`** para salida QR/UX (sin prefijo, sin formato)
-   **Estilos de consola** (ej. `pretty | compact | json`)
-   **Nivel de log de consola** separado del nivel de log de archivo (el archivo mantiene el detalle completo cuando `logging.level` está establecido en `debug`/`trace`)
-   **Los cuerpos de mensajes de WhatsApp** se registran en `debug` (usa `--verbose` para verlos)

Esto mantiene estables los logs de archivo existentes mientras hace que la salida interactiva sea escaneable.

[Doctor](./doctor.md)[Bloqueo del Gateway](./gateway-lock.md)

---