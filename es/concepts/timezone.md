

  Conceptos internos

  
# Zonas Horarias

OpenClaw estandariza las marcas de tiempo para que el modelo vea una **única hora de referencia**.

## Sobres de mensajes (local por defecto)

Los mensajes entrantes se envuelven en un sobre como:

```
[Proveedor ... 2026-01-05 16:26 PST] texto del mensaje
```

La marca de tiempo en el sobre es **local al host por defecto**, con precisión de minutos. Puedes anular esto con:

```json
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | zona horaria IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

-   `envelopeTimezone: "utc"` usa UTC.
-   `envelopeTimezone: "user"` usa `agents.defaults.userTimezone` (recurre a la zona horaria del host).
-   Usa una zona horaria IANA explícita (ej., `"Europe/Vienna"`) para un desplazamiento fijo.
-   `envelopeTimestamp: "off"` elimina las marcas de tiempo absolutas de los encabezados del sobre.
-   `envelopeElapsed: "off"` elimina los sufijos de tiempo transcurrido (el estilo `+2m`).

### Ejemplos

**Local (por defecto):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hola
```

**Zona horaria fija:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hola
```

**Tiempo transcurrido:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] seguimiento
```

## Cargas útiles de herramientas (datos crudos del proveedor + campos normalizados)

Las llamadas a herramientas (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) devuelven **marcas de tiempo crudas del proveedor**. También adjuntamos campos normalizados para consistencia:

-   `timestampMs` (milisegundos de época UTC)
-   `timestampUtc` (cadena ISO 8601 UTC)

Los campos crudos del proveedor se conservan.

## Zona horaria del usuario para el prompt del sistema

Establece `agents.defaults.userTimezone` para indicar al modelo la zona horaria local del usuario. Si no está configurada, OpenClaw resuelve la **zona horaria del host en tiempo de ejecución** (sin escritura de configuración).

```json
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

El prompt del sistema incluye:

-   Sección `Fecha y Hora Actual` con la hora local y la zona horaria
-   `Formato de hora: 12 horas` o `24 horas`

Puedes controlar el formato del prompt con `agents.defaults.timeFormat` (`auto` | `12` | `24`). Consulta [Fecha y Hora](../date-time.md) para el comportamiento completo y ejemplos.

[Seguimiento de Uso](./usage-tracking.md)[Créditos](../reference/credits.md)