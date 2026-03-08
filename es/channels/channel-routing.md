

  Configuración

  
# Enrutamiento de Canales

OpenClaw enruta las respuestas **de vuelta al canal de donde vino un mensaje**. El modelo no elige un canal; el enrutamiento es determinista y controlado por la configuración del host.

## Términos clave

-   **Canal**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
-   **AccountId**: instancia de cuenta por canal (cuando se admite).
-   Cuenta predeterminada opcional del canal: `channels..defaultAccount` elige qué cuenta se usa cuando una ruta saliente no especifica `accountId`.
    -   En configuraciones multi-cuenta, establece un valor predeterminado explícito (`defaultAccount` o `accounts.default`) cuando se configuran dos o más cuentas. Sin él, el enrutamiento de reserva puede elegir el primer ID de cuenta normalizado.
-   **AgentId**: un espacio de trabajo aislado + almacén de sesiones ("cerebro").
-   **SessionKey**: la clave de depósito utilizada para almacenar contexto y controlar la concurrencia.

## Formas de clave de sesión (ejemplos)

Los mensajes directos se colapsan en la sesión **principal** del agente:

-   `agent::` (predeterminado: `agent:main:main`)

Los grupos y canales permanecen aislados por canal:

-   Grupos: `agent:::group:`
-   Canales/salas: `agent:::channel:`

Hilos:

-   Los hilos de Slack/Discord agregan `:thread:` a la clave base.
-   Los temas de foro de Telegram incrustan `:topic:` en la clave del grupo.

Ejemplos:

-   `agent:main:telegram:group:-1001234567890:topic:42`
-   `agent:main:discord:channel:123456:thread:987654`

## Fijación de ruta principal para MD

Cuando `session.dmScope` es `main`, los mensajes directos pueden compartir una sesión principal. Para evitar que el `lastRoute` de la sesión sea sobrescrito por MD de no propietarios, OpenClaw infiere un propietario fijado a partir de `allowFrom` cuando se cumplen todas estas condiciones:

-   `allowFrom` tiene exactamente una entrada que no es comodín.
-   La entrada se puede normalizar a un ID de remitente concreto para ese canal.
-   El remitente del MD entrante no coincide con ese propietario fijado.

En ese caso de falta de coincidencia, OpenClaw aún registra los metadatos de la sesión entrante, pero omite actualizar el `lastRoute` de la sesión principal.

## Reglas de enrutamiento (cómo se elige un agente)

El enrutamiento elige **un agente** para cada mensaje entrante:

1.  **Coincidencia exacta de par** (`bindings` con `peer.kind` + `peer.id`).
2.  **Coincidencia de par padre** (herencia de hilo).
3.  **Coincidencia de gremio + roles** (Discord) mediante `guildId` + `roles`.
4.  **Coincidencia de gremio** (Discord) mediante `guildId`.
5.  **Coincidencia de equipo** (Slack) mediante `teamId`.
6.  **Coincidencia de cuenta** (`accountId` en el canal).
7.  **Coincidencia de canal** (cualquier cuenta en ese canal, `accountId: "*"`).
8.  **Agente predeterminado** (`agents.list[].default`, si no, la primera entrada de la lista, reserva a `main`).

Cuando una vinculación incluye múltiples campos de coincidencia (`peer`, `guildId`, `teamId`, `roles`), **todos los campos proporcionados deben coincidir** para que se aplique esa vinculación. El agente coincidente determina qué espacio de trabajo y almacén de sesiones se utilizan.

## Grupos de difusión (ejecutar múltiples agentes)

Los grupos de difusión te permiten ejecutar **múltiples agentes** para el mismo par **cuando OpenClaw normalmente respondería** (por ejemplo: en grupos de WhatsApp, después de la activación por mención). Configuración:

```json
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

Ver: [Grupos de Difusión](./broadcast-groups.md).

## Resumen de configuración

-   `agents.list`: definiciones de agentes con nombre (espacio de trabajo, modelo, etc.).
-   `bindings`: mapea canales/cuentas/pares entrantes a agentes.

Ejemplo:

```json
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

## Almacenamiento de sesiones

Los almacenes de sesiones residen bajo el directorio de estado (predeterminado `~/.openclaw`):

-   `~/.openclaw/agents//sessions/sessions.json`
-   Las transcripciones JSONL residen junto al almacén

Puedes anular la ruta del almacén mediante `session.store` y la plantilla `{agentId}`.

## Comportamiento de WebChat

WebChat se adjunta al **agente seleccionado** y usa por defecto la sesión principal del agente. Debido a esto, WebChat te permite ver el contexto entre canales para ese agente en un solo lugar.

## Contexto de respuesta

Las respuestas entrantes incluyen:

-   `ReplyToId`, `ReplyToBody` y `ReplyToSender` cuando están disponibles.
-   El contexto citado se agrega a `Body` como un bloque `[Respondiendo a ...]`.

Esto es consistente en todos los canales.

[Grupos de Difusión](./broadcast-groups.md)[Análisis de Ubicación del Canal](./location.md)

---