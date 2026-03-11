

  Configuración y operaciones

  
# Heartbeat

> **¿Heartbeat vs Cron?** Consulta [Cron vs Heartbeat](../automation/cron-vs-heartbeat.md) para orientación sobre cuándo usar cada uno.

Heartbeat ejecuta **turnos periódicos del agente** en la sesión principal para que el modelo pueda resaltar cualquier cosa que necesite atención sin spamearte. Solución de problemas: [/automation/troubleshooting](../automation/troubleshooting.md)

## Inicio rápido (principiante)

1.  Deja los heartbeats habilitados (el valor predeterminado es `30m`, o `1h` para OAuth/setup-token de Anthropic) o establece tu propia cadencia.
2.  Crea una pequeña lista de verificación `HEARTBEAT.md` en el espacio de trabajo del agente (opcional pero recomendado).
3.  Decide a dónde deben ir los mensajes de heartbeat (`target: "none"` es el valor predeterminado; establece `target: "last"` para enrutar al último contacto).
4.  Opcional: habilita la entrega del razonamiento del heartbeat para transparencia.
5.  Opcional: usa contexto de arranque ligero si los heartbeats solo necesitan `HEARTBEAT.md`.
6.  Opcional: restringe los heartbeats a horas activas (hora local).

Configuración de ejemplo:

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // entrega explícita al último contacto (el valor predeterminado es "none")
        directPolicy: "allow", // predeterminado: permitir destinos directos/DM; establece "block" para suprimir
        lightContext: true, // opcional: solo inyecta HEARTBEAT.md de los archivos de arranque
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // opcional: enviar también un mensaje `Reasoning:` separado
      },
    },
  },
}
```

## Valores predeterminados

-   Intervalo: `30m` (o `1h` cuando OAuth/setup-token de Anthropic es el modo de autenticación detectado). Establece `agents.defaults.heartbeat.every` o por agente `agents.list[].heartbeat.every`; usa `0m` para deshabilitar.
-   Cuerpo del prompt (configurable mediante `agents.defaults.heartbeat.prompt`): `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
-   El prompt del heartbeat se envía **literalmente** como mensaje del usuario. El prompt del sistema incluye una sección "Heartbeat" y la ejecución se marca internamente.
-   Las horas activas (`heartbeat.activeHours`) se verifican en la zona horaria configurada. Fuera de la ventana, los heartbeats se omiten hasta el siguiente tick dentro de la ventana.

## Para qué sirve el prompt del heartbeat

El prompt predeterminado es intencionalmente amplio:

-   **Tareas en segundo plano**: "Consider outstanding tasks" impulsa al agente a revisar seguimientos (bandeja de entrada, calendario, recordatorios, trabajo en cola) y resaltar cualquier cosa urgente.
-   **Registro con el humano**: "Checkup sometimes on your human during day time" impulsa un mensaje ocasional ligero "¿necesitas algo?", pero evita spam nocturno usando tu zona horaria local configurada (ver [/concepts/timezone](../concepts/timezone.md)).

Si quieres que un heartbeat haga algo muy específico (por ejemplo, "verificar estadísticas de Gmail PubSub" o "verificar salud del gateway"), establece `agents.defaults.heartbeat.prompt` (o `agents.list[].heartbeat.prompt`) a un cuerpo personalizado (enviado literalmente).

## Contrato de respuesta

-   Si nada necesita atención, responde con **`HEARTBEAT_OK`**.
-   Durante las ejecuciones de heartbeat, OpenClaw trata `HEARTBEAT_OK` como un acuse de recibo cuando aparece al **inicio o final** de la respuesta. El token se elimina y la respuesta se descarta si el contenido restante es **≤ `ackMaxChars`** (predeterminado: 300).
-   Si `HEARTBEAT_OK` aparece en el **medio** de una respuesta, no se trata de manera especial.
-   Para alertas, **no** incluyas `HEARTBEAT_OK`; devuelve solo el texto de la alerta.

Fuera de los heartbeats, un `HEARTBEAT_OK` extraviado al inicio/final de un mensaje se elimina y se registra; un mensaje que solo es `HEARTBEAT_OK` se descarta.

## Configuración

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // predeterminado: 30m (0m deshabilita)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // predeterminado: false (entregar mensaje Reasoning: separado cuando esté disponible)
        lightContext: false, // predeterminado: false; true mantiene solo HEARTBEAT.md de los archivos de arranque del espacio de trabajo
        target: "last", // predeterminado: none | opciones: last | none | <channel id> (core o plugin, ej. "bluebubbles")
        to: "+15551234567", // anulación opcional específica del canal
        accountId: "ops-bot", // id de canal multi-cuenta opcional
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // caracteres máximos permitidos después de HEARTBEAT_OK
      },
    },
  },
}
```

### Alcance y precedencia

-   `agents.defaults.heartbeat` establece el comportamiento global del heartbeat.
-   `agents.list[].heartbeat` se fusiona encima; si algún agente tiene un bloque `heartbeat`, **solo esos agentes** ejecutan heartbeats.
-   `channels.defaults.heartbeat` establece los valores predeterminados de visibilidad para todos los canales.
-   `channels..heartbeat` anula los valores predeterminados del canal.
-   `channels..accounts..heartbeat` (canales multi-cuenta) anula la configuración por canal.

### Heartbeats por agente

Si alguna entrada `agents.list[]` incluye un bloque `heartbeat`, **solo esos agentes** ejecutan heartbeats. El bloque por agente se fusiona encima de `agents.defaults.heartbeat` (para que puedas establecer valores predeterminados compartidos una vez y anular por agente). Ejemplo: dos agentes, solo el segundo agente ejecuta heartbeats.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // entrega explícita al último contacto (el valor predeterminado es "none")
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### Ejemplo de horas activas

Restringe los heartbeats a horas comerciales en una zona horaria específica:

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // entrega explícita al último contacto (el valor predeterminado es "none")
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // opcional; usa tu userTimezone si está configurada, de lo contrario la zona horaria del host
        },
      },
    },
  },
}
```

Fuera de esta ventana (antes de las 9am o después de las 10pm hora del Este), los heartbeats se omiten. El siguiente tick programado dentro de la ventana se ejecutará normalmente.

### Configuración 24/7

Si quieres que los heartbeats se ejecuten todo el día, usa uno de estos patrones:

-   Omite `activeHours` por completo (sin restricción de ventana de tiempo; este es el comportamiento predeterminado).
-   Establece una ventana de día completo: `activeHours: { start: "00:00", end: "24:00" }`.

No establezcas la misma hora de `start` y `end` (por ejemplo `08:00` a `08:00`). Eso se trata como una ventana de ancho cero, por lo que los heartbeats siempre se omiten.

### Ejemplo multi-cuenta

Usa `accountId` para apuntar a una cuenta específica en canales multi-cuenta como Telegram:

```json
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // opcional: enrutar a un tema/hilo específico
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### Notas de campo

-   `every`: intervalo del heartbeat (cadena de duración; unidad predeterminada = minutos).
-   `model`: anulación opcional del modelo para ejecuciones de heartbeat (`provider/model`).
-   `includeReasoning`: cuando está habilitado, también entrega el mensaje `Reasoning:` separado cuando está disponible (misma forma que `/reasoning on`).
-   `lightContext`: cuando es true, las ejecuciones de heartbeat usan contexto de arranque ligero y mantienen solo `HEARTBEAT.md` de los archivos de arranque del espacio de trabajo.
-   `session`: clave de sesión opcional para ejecuciones de heartbeat.
    -   `main` (predeterminado): sesión principal del agente.
    -   Clave de sesión explícita (copia de `openclaw sessions --json` o la [CLI de sesiones](../cli/sessions.md)).
    -   Formatos de clave de sesión: ver [Sesiones](../concepts/session.md) y [Grupos](../channels/groups.md).
-   `target`:
    -   `last`: entregar al último canal externo utilizado.
    -   canal explícito: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
    -   `none` (predeterminado): ejecutar el heartbeat pero **no entregar** externamente.
-   `directPolicy`: controla el comportamiento de entrega directa/DM:
    -   `allow` (predeterminado): permitir entrega de heartbeat directa/DM.
    -   `block`: suprimir entrega directa/DM (`reason=dm-blocked`).
-   `to`: anulación opcional del destinatario (id específico del canal, ej. E.164 para WhatsApp o un id de chat de Telegram). Para temas/hilos de Telegram, usa `:topic:`.
-   `accountId`: id de cuenta opcional para canales multi-cuenta. Cuando `target: "last"`, el id de cuenta se aplica al último canal resuelto si admite cuentas; de lo contrario, se ignora. Si el id de cuenta no coincide con una cuenta configurada para el canal resuelto, la entrega se omite.
-   `prompt`: anula el cuerpo del prompt predeterminado (no se fusiona).
-   `ackMaxChars`: caracteres máximos permitidos después de `HEARTBEAT_OK` antes de la entrega.
-   `suppressToolErrorWarnings`: cuando es true, suprime las advertencias de error de herramientas durante las ejecuciones de heartbeat.
-   `activeHours`: restringe las ejecuciones de heartbeat a una ventana de tiempo. Objeto con `start` (HH:MM, inclusivo; usa `00:00` para inicio del día), `end` (HH:MM exclusivo; se permite `24:00` para fin del día) y `timezone` opcional.
    -   Omitido o `"user"`: usa tu `agents.defaults.userTimezone` si está configurada, de lo contrario recurre a la zona horaria del sistema host.
    -   `"local"`: siempre usa la zona horaria del sistema host.
    -   Cualquier identificador IANA (ej. `America/New_York`): se usa directamente; si no es válido, recurre al comportamiento `"user"` anterior.
    -   `start` y `end` no deben ser iguales para una ventana activa; valores iguales se tratan como ancho cero (siempre fuera de la ventana).
    -   Fuera de la ventana activa, los heartbeats se omiten hasta el siguiente tick dentro de la ventana.

## Comportamiento de entrega

-   Los heartbeats se ejecutan en la sesión principal del agente por defecto (`agent::`), o `global` cuando `session.scope = "global"`. Establece `session` para anular a una sesión de canal específica (Discord/WhatsApp/etc.).
-   `session` solo afecta el contexto de ejecución; la entrega la controlan `target` y `to`.
-   Para entregar a un canal/destinatario específico, establece `target` + `to`. Con `target: "last"`, la entrega usa el último canal externo para esa sesión.
-   Las entregas de heartbeat permiten destinos directos/DM por defecto. Establece `directPolicy: "block"` para suprimir envíos a destinos directos mientras aún se ejecuta el turno del heartbeat.
-   Si la cola principal está ocupada, el heartbeat se omite y se reintenta más tarde.
-   Si `target` se resuelve a ningún destino externo, la ejecución aún ocurre pero no se envía ningún mensaje saliente.
-   Las respuestas solo de heartbeat **no** mantienen viva la sesión; el último `updatedAt` se restaura para que la expiración por inactividad se comporte normalmente.

## Controles de visibilidad

Por defecto, los acuses de recibo `HEARTBEAT_OK` se suprimen mientras se entrega el contenido de alerta. Puedes ajustar esto por canal o por cuenta:

```
channels:
  defaults:
    heartbeat:
      showOk: false # Ocultar HEARTBEAT_OK (predeterminado)
      showAlerts: true # Mostrar mensajes de alerta (predeterminado)
      useIndicator: true # Emitir eventos indicadores (predeterminado)
  telegram:
    heartbeat:
      showOk: true # Mostrar acuses de recibo OK en Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Suprimir entrega de alertas para esta cuenta
```

Precedencia: por-cuenta → por-canal → valores predeterminados del canal → valores predeterminados integrados.

### Qué hace cada bandera

-   `showOk`: envía un acuse de recibo `HEARTBEAT_OK` cuando el modelo devuelve una respuesta solo OK.
-   `showAlerts`: envía el contenido de alerta cuando el modelo devuelve una respuesta no OK.
-   `useIndicator`: emite eventos indicadores para superficies de estado de la UI.

Si **las tres** son falsas, OpenClaw omite la ejecución del heartbeat por completo (sin llamada al modelo).

### Ejemplos por canal vs por cuenta

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # todas las cuentas de Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # suprimir alertas solo para la cuenta ops
  telegram:
    heartbeat:
      showOk: true
```

### Patrones comunes

| Objetivo | Configuración |
| --- | --- |
| Comportamiento predeterminado (OKs silenciosos, alertas activadas) | *(no se necesita configuración)* |
| Totalmente silencioso (sin mensajes, sin indicador) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Solo indicador (sin mensajes) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OKs solo en un canal | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md (opcional)

Si existe un archivo `HEARTBEAT.md` en el espacio de trabajo, el prompt predeterminado le dice al agente que lo lea. Piensa en él como tu "lista de verificación del heartbeat": pequeña, estable y segura para incluir cada 30 minutos. Si `HEARTBEAT.md` existe pero está efectivamente vacío (solo líneas en blanco y encabezados markdown como `# Heading`), OpenClaw omite la ejecución del heartbeat para ahorrar llamadas a la API. Si el archivo falta, el heartbeat aún se ejecuta y el modelo decide qué hacer. Mantenlo pequeño (lista de verificación corta o recordatorios) para evitar la inflación del prompt. Ejemplo `HEARTBEAT.md`:

```bash
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it’s daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### ¿Puede el agente actualizar HEARTBEAT.md?

Sí, si se lo pides. `HEARTBEAT.md` es solo un archivo normal en el espacio de trabajo del agente, por lo que puedes decirle al agente (en un chat normal) algo como:

-   "Actualiza `HEARTBEAT.md` para agregar una verificación diaria del calendario."
-   "Reescribe `HEARTBEAT.md` para que sea más corto y se centre en seguimientos de la bandeja de entrada."

Si quieres que esto suceda de manera proactiva, también puedes incluir una línea explícita en tu prompt de heartbeat como: "Si la lista de verificación se vuelve obsoleta, actualiza HEARTBEAT.md con una mejor." Nota de seguridad: no pongas secretos (claves API, números de teléfono, tokens privados) en `HEARTBEAT.md` — se convierte en parte del contexto del prompt.

## Despertar manual (bajo demanda)

Puedes poner en cola un evento del sistema y activar un heartbeat inmediato con:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Si múltiples agentes tienen `heartbeat` configurado, un despertar manual ejecuta cada uno de esos heartbeats de agente inmediatamente. Usa `--mode next-heartbeat` para esperar al siguiente tick programado.

## Entrega de razonamiento (opcional)

Por defecto, los heartbeats entregan solo la carga útil "respuesta" final. Si quieres transparencia, habilita:

-   `agents.defaults.heartbeat.includeReasoning: true`

Cuando está habilitado, los heartbeats también entregarán un mensaje separado con prefijo `Reasoning:` (misma forma que `/reasoning on`). Esto puede ser útil cuando el agente está gestionando múltiples sesiones/codexes y quieres ver por qué decidió contactarte, pero también puede filtrar más detalles internos de los que deseas. Prefiere mantenerlo desactivado en chats grupales.

## Conciencia de costos

Los heartbeats ejecutan turnos completos del agente. Intervalos más cortos consumen más tokens. Mantén `HEARTBEAT.md` pequeño y considera un `model` más barato o `target: "none"` si solo quieres actualizaciones de estado internas.

[Health Checks](./health.md)[Doctor](./doctor.md)