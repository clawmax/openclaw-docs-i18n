

  Referencia técnica

  
# Fecha y Hora

OpenClaw utiliza por defecto **la hora local del host para las marcas de tiempo de transporte** y **solo la zona horaria del usuario en el prompt del sistema**. Las marcas de tiempo del proveedor se conservan para que las herramientas mantengan su semántica nativa (la hora actual está disponible a través de `session_status`).

## Sobres de mensajes (local por defecto)

Los mensajes entrantes se envuelven con una marca de tiempo (precisión de minuto):

```
[Provider ... 2026-01-05 16:26 PST] texto del mensaje
```

Esta marca de tiempo del sobre es **local del host por defecto**, independientemente de la zona horaria del proveedor. Puedes anular este comportamiento:

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
-   `envelopeTimezone: "local"` usa la zona horaria del host.
-   `envelopeTimezone: "user"` usa `agents.defaults.userTimezone` (recurre a la zona horaria del host).
-   Usa una zona horaria IANA explícita (ej., `"America/Chicago"`) para una zona fija.
-   `envelopeTimestamp: "off"` elimina las marcas de tiempo absolutas de los encabezados del sobre.
-   `envelopeElapsed: "off"` elimina los sufijos de tiempo transcurrido (el estilo `+2m`).

### Ejemplos

**Local (por defecto):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hola
```

**Zona horaria del usuario:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hola
```

**Tiempo transcurrido habilitado:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] seguimiento
```

## Prompt del sistema: Fecha y Hora Actual

Si se conoce la zona horaria del usuario, el prompt del sistema incluye una sección dedicada **Fecha y Hora Actual** con **solo la zona horaria** (sin formato de reloj/hora) para mantener estable el almacenamiento en caché del prompt:

```bash
Zona horaria: America/Chicago
```

Cuando el agente necesita la hora actual, usa la herramienta `session_status`; la tarjeta de estado incluye una línea con la marca de tiempo.

## Líneas de eventos del sistema (local por defecto)

Los eventos del sistema en cola insertados en el contexto del agente tienen prefijo de marca de tiempo usando la misma selección de zona horaria que los sobres de mensajes (por defecto: local del host).

```yaml
System: [2026-01-12 12:19:17 PST] Modelo cambiado.
```

### Configurar zona horaria del usuario + formato

```json
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

-   `userTimezone` establece la **zona horaria local del usuario** para el contexto del prompt.
-   `timeFormat` controla la **visualización 12h/24h** en el prompt. `auto` sigue las preferencias del SO.

## Detección de formato de hora (auto)

Cuando `timeFormat: "auto"`, OpenClaw inspecciona la preferencia del SO (macOS/Windows) y recurre al formato de localización. El valor detectado se **almacena en caché por proceso** para evitar llamadas repetidas al sistema.

## Cargas útiles de herramientas + conectores (hora del proveedor en crudo + campos normalizados)

Las herramientas de canal devuelven **marcas de tiempo nativas del proveedor** y añaden campos normalizados para consistencia:

-   `timestampMs`: milisegundos de época (UTC)
-   `timestampUtc`: cadena ISO 8601 UTC

Los campos crudos del proveedor se conservan para que no se pierda nada.

-   Slack: cadenas tipo época de la API
-   Discord: marcas de tiempo ISO UTC
-   Telegram/WhatsApp: marcas de tiempo numéricas/ISO específicas del proveedor

Si necesitas la hora local, conviértela aguas abajo usando la zona horaria conocida.

## Documentación relacionada

-   [Prompt del Sistema](./concepts/system-prompt.md)
-   [Zonas Horarias](./concepts/timezone.md)
-   [Mensajes](./concepts/messages.md)

[Higiene de Transcripciones](./reference/transcript-hygiene.md)[TypeBox](./concepts/typebox.md)