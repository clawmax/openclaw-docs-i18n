

  Mensajes y entrega

  
# Mensajes

Esta página explica cómo OpenClaw maneja los mensajes entrantes, sesiones, colas, streaming y visibilidad del razonamiento.

## Flujo de mensajes (nivel alto)

```
Mensaje entrante
  -> enrutamiento/vinculaciones -> clave de sesión
  -> cola (si hay una ejecución activa)
  -> ejecución del agente (streaming + herramientas)
  -> respuestas salientes (límites del canal + fragmentación)
```

Los controles clave están en la configuración:

-   `messages.*` para prefijos, colas y comportamiento de grupo.
-   `agents.defaults.*` para streaming por bloques y valores predeterminados de fragmentación.
-   Anulaciones por canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) para límites y alternancias de streaming.

Consulta [Configuración](../gateway/configuration.md) para el esquema completo.

## Deduplicación entrante

Los canales pueden reenviar el mismo mensaje tras reconexiones. OpenClaw mantiene una caché de corta duración indexada por canal/cuenta/par/sesión/id del mensaje para que las entregas duplicadas no desencadenen otra ejecución del agente.

## Debouncing entrante

Los mensajes consecutivos rápidos del **mismo remitente** pueden agruparse en un solo turno del agente mediante `messages.inbound`. El debouncing se aplica por canal + conversación y utiliza el mensaje más reciente para el hilo/IDs de respuesta. Configuración (predeterminado global + anulaciones por canal):

```json
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Notas:

-   El debouncing se aplica a mensajes **solo de texto**; los medios/archivos adjuntos se envían inmediatamente.
-   Los comandos de control omiten el debouncing para que permanezcan independientes.

## Sesiones y dispositivos

Las sesiones son propiedad de la pasarela, no de los clientes.

-   Los chats directos se agrupan en la clave de sesión principal del agente.
-   Los grupos/canales obtienen sus propias claves de sesión.
-   El almacén de sesiones y las transcripciones residen en el host de la pasarela.

Múltiples dispositivos/canales pueden mapearse a la misma sesión, pero el historial no se sincroniza completamente con cada cliente. Recomendación: usa un dispositivo principal para conversaciones largas para evitar contexto divergente. La Interfaz de Control y la TUI siempre muestran la transcripción de sesión respaldada por la pasarela, por lo que son la fuente de verdad. Detalles: [Gestión de sesiones](./session.md).

## Cuerpos entrantes y contexto del historial

OpenClaw separa el **cuerpo del prompt** del **cuerpo del comando**:

-   `Body`: texto del prompt enviado al agente. Puede incluir envoltorios del canal y envoltorios opcionales del historial.
-   `CommandBody`: texto crudo del usuario para el análisis de directivas/comandos.
-   `RawBody`: alias heredado para `CommandBody` (mantenido por compatibilidad).

Cuando un canal proporciona historial, utiliza un envoltorio compartido:

-   `[Mensajes del chat desde tu última respuesta - para contexto]`
-   `[Mensaje actual - responde a esto]`

Para **chats no directos** (grupos/canales/salas), el **cuerpo del mensaje actual** tiene como prefijo la etiqueta del remitente (mismo estilo usado para las entradas del historial). Esto mantiene los mensajes en tiempo real y los mensajes en cola/del historial consistentes en el prompt del agente. Los búferes de historial son **solo pendientes**: incluyen mensajes grupales que *no* desencadenaron una ejecución (por ejemplo, mensajes con mención condicionada) y **excluyen** mensajes ya en la transcripción de la sesión. La eliminación de directivas solo se aplica a la sección del **mensaje actual** para que el historial permanezca intacto. Los canales que envuelven el historial deben establecer `CommandBody` (o `RawBody`) al texto original del mensaje y mantener `Body` como el prompt combinado. Los búferes de historial son configurables mediante `messages.groupChat.historyLimit` (predeterminado global) y anulaciones por canal como `channels.slack.historyLimit` o `channels.telegram.accounts..historyLimit` (establece `0` para deshabilitar).

## Colas y seguimientos

Si ya hay una ejecución activa, los mensajes entrantes pueden ponerse en cola, dirigirse a la ejecución actual o recopilarse para un turno de seguimiento.

-   Configura mediante `messages.queue` (y `messages.queue.byChannel`).
-   Modos: `interrupt`, `steer`, `followup`, `collect`, más variantes de acumulación.

Detalles: [Colas](./queue.md).

## Streaming, fragmentación y agrupación

El streaming por bloques envía respuestas parciales a medida que el modelo produce bloques de texto. La fragmentación respeta los límites de texto del canal y evita dividir código delimitado. Configuraciones clave:

-   `agents.defaults.blockStreamingDefault` (`on|off`, predeterminado off)
-   `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
-   `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
-   `agents.defaults.blockStreamingCoalesce` (agrupación basada en inactividad)
-   `agents.defaults.humanDelay` (pausa de estilo humano entre respuestas de bloques)
-   Anulaciones por canal: `*.blockStreaming` y `*.blockStreamingCoalesce` (los canales no-Telegram requieren `*.blockStreaming: true` explícito)

Detalles: [Streaming + fragmentación](./streaming.md).

## Visibilidad del razonamiento y tokens

OpenClaw puede exponer u ocultar el razonamiento del modelo:

-   `/reasoning on|off|stream` controla la visibilidad.
-   El contenido del razonamiento aún cuenta para el uso de tokens cuando lo produce el modelo.
-   Telegram admite el flujo de razonamiento en la burbuja de borrador.

Detalles: [Directivas de pensamiento + razonamiento](../tools/thinking.md) y [Uso de tokens](../reference/token-use.md).

## Prefijos, hilos y respuestas

El formato de los mensajes salientes está centralizado en `messages`:

-   `messages.responsePrefix`, `channels..responsePrefix`, y `channels..accounts..responsePrefix` (cascada de prefijos salientes), más `channels.whatsapp.messagePrefix` (prefijo entrante de WhatsApp)
-   Hilos de respuesta mediante `replyToMode` y valores predeterminados por canal

Detalles: [Configuración](../gateway/configuration.md#messages) y documentación del canal.

[Presencia](./presence.md)[Streaming y Fragmentación](./streaming.md)