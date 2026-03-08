title: "Guía de configuración e integración de OpenClaw con Mattermost"
description: "Aprende a instalar y configurar el plugin de Mattermost para OpenClaw. Configura tokens de bot, comandos de barra, modos de chat, control de acceso y funciones interactivas."
keywords: ["mattermost", "openclaw", "integración de chatbot", "configuración de plugin", "comandos de barra", "mensajería de equipo", "configuración de bot", "chat autoalojado"]
---

  Plataformas de mensajería

  
# Mattermost

Estado: compatible mediante plugin (token de bot + eventos WebSocket). Se admiten canales, grupos y mensajes directos. Mattermost es una plataforma de mensajería de equipo autoalojable; consulta el sitio oficial en [mattermost.com](https://mattermost.com) para detalles del producto y descargas.

## Plugin requerido

Mattermost se distribuye como un plugin y no está incluido en la instalación principal. Instálalo mediante CLI (registro npm):

```bash
openclaw plugins install @openclaw/mattermost
```

Instalación local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/mattermost
```

Si eliges Mattermost durante la configuración/onboarding y se detecta un repositorio git, OpenClaw ofrecerá la ruta de instalación local automáticamente. Detalles: [Plugins](../tools/plugin.md)

## Configuración rápida

1.  Instala el plugin de Mattermost.
2.  Crea una cuenta de bot en Mattermost y copia el **token del bot**.
3.  Copia la **URL base** de Mattermost (ej., `https://chat.example.com`).
4.  Configura OpenClaw e inicia la pasarela.

Configuración mínima:

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Comandos de barra nativos

Los comandos de barra nativos son opcionales. Cuando están habilitados, OpenClaw registra comandos `oc_*` mediante la API de Mattermost y recibe llamadas POST en el servidor HTTP de la pasarela.

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // Usar cuando Mattermost no puede alcanzar la pasarela directamente (proxy inverso/URL pública).
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

Notas:

-   `native: "auto"` se deshabilita por defecto para Mattermost. Establece `native: true` para habilitar.
-   Si se omite `callbackUrl`, OpenClaw deriva una a partir del host/puerto de la pasarela + `callbackPath`.
-   Para configuraciones multi-cuenta, `commands` se puede establecer a nivel superior o bajo `channels.mattermost.accounts..commands` (los valores de cuenta anulan los campos de nivel superior).
-   Las llamadas de comandos se validan con tokens por comando y fallan cerradas cuando fallan las comprobaciones de token.
-   Requisito de accesibilidad: el endpoint de callback debe ser accesible desde el servidor de Mattermost.
    -   No establezcas `callbackUrl` a `localhost` a menos que Mattermost se ejecute en el mismo host/espacio de nombres de red que OpenClaw.
    -   No establezcas `callbackUrl` a tu URL base de Mattermost a menos que esa URL redirija mediante proxy inverso `/api/channels/mattermost/command` a OpenClaw.
    -   Una comprobación rápida es `curl https://<gateway-host>/api/channels/mattermost/command`; un GET debería devolver `405 Method Not Allowed` de OpenClaw, no `404`.
-   Requisito de lista blanca de salida de Mattermost:
    -   Si tu callback apunta a direcciones privadas/tailnet/internas, establece `ServiceSettings.AllowedUntrustedInternalConnections` de Mattermost para incluir el host/dominio del callback.
    -   Usa entradas de host/dominio, no URLs completas.
        -   Correcto: `gateway.tailnet-name.ts.net`
        -   Incorrecto: `https://gateway.tailnet-name.ts.net`

## Variables de entorno (cuenta por defecto)

Establece estas en el host de la pasarela si prefieres variables de entorno:

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

Las variables de entorno se aplican solo a la cuenta **por defecto** (`default`). Otras cuentas deben usar valores de configuración.

## Modos de chat

Mattermost responde a mensajes directos automáticamente. El comportamiento en canales se controla mediante `chatmode`:

-   `oncall` (por defecto): responde solo cuando es @mencionado en canales.
-   `onmessage`: responde a cada mensaje del canal.
-   `onchar`: responde cuando un mensaje comienza con un prefijo desencadenante.

Ejemplo de configuración:

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Notas:

-   `onchar` aún responde a @menciones explícitas.
-   `channels.mattermost.requireMention` se respeta para configuraciones heredadas, pero se prefiere `chatmode`.

## Control de acceso (mensajes directos)

-   Por defecto: `channels.mattermost.dmPolicy = "pairing"` (los remitentes desconocidos reciben un código de emparejamiento).
-   Aprobar mediante:
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   Mensajes directos públicos: `channels.mattermost.dmPolicy="open"` más `channels.mattermost.allowFrom=["*"]`.

## Canales (grupos)

-   Por defecto: `channels.mattermost.groupPolicy = "allowlist"` (acceso condicionado a mención).
-   Lista blanca de remitentes con `channels.mattermost.groupAllowFrom` (se recomiendan IDs de usuario).
-   La coincidencia por `@nombredeusuario` es mutable y solo se habilita cuando `channels.mattermost.dangerouslyAllowNameMatching: true`.
-   Canales abiertos: `channels.mattermost.groupPolicy="open"` (acceso condicionado a mención).
-   Nota de tiempo de ejecución: si `channels.mattermost` falta completamente, el tiempo de ejecución recurre a `groupPolicy="allowlist"` para comprobaciones de grupo (incluso si `channels.defaults.groupPolicy` está establecido).

## Destinos para entrega saliente

Usa estos formatos de destino con `openclaw message send` o cron/webhooks:

-   `channel:` para un canal
-   `user:` para un mensaje directo
-   `@nombredeusuario` para un mensaje directo (resuelto mediante la API de Mattermost)

Los IDs simples se tratan como canales.

## Reacciones (herramienta de mensaje)

-   Usa `message action=react` con `channel=mattermost`.
-   `messageId` es el id de publicación de Mattermost.
-   `emoji` acepta nombres como `thumbsup` o `:+1:` (los dos puntos son opcionales).
-   Establece `remove=true` (booleano) para eliminar una reacción.
-   Los eventos de agregar/eliminar reacción se reenvían como eventos del sistema a la sesión del agente enrutado.

Ejemplos:

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Configuración:

-   `channels.mattermost.actions.reactions`: habilita/deshabilita acciones de reacción (verdadero por defecto).
-   Anulación por cuenta: `channels.mattermost.accounts..actions.reactions`.

## Botones interactivos (herramienta de mensaje)

Envía mensajes con botones en los que se puede hacer clic. Cuando un usuario hace clic en un botón, el agente recibe la selección y puede responder. Habilita los botones agregando `inlineButtons` a las capacidades del canal:

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

Usa `message action=send` con un parámetro `buttons`. Los botones son un array 2D (filas de botones):

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Sí","callback_data":"yes"},{"text":"No","callback_data":"no"}]]
```

Campos del botón:

-   `text` (requerido): etiqueta de visualización.
-   `callback_data` (requerido): valor enviado de vuelta al hacer clic (usado como ID de acción).
-   `style` (opcional): `"default"`, `"primary"`, o `"danger"`.

Cuando un usuario hace clic en un botón:

1.  Todos los botones se reemplazan por una línea de confirmación (ej., ”✓ **Sí** seleccionado por @usuario”).
2.  El agente recibe la selección como un mensaje entrante y responde.

Notas:

-   Las llamadas de botones usan verificación HMAC-SHA256 (automática, no requiere configuración).
-   Mattermost elimina los datos de callback de sus respuestas API (característica de seguridad), por lo que todos los botones se eliminan al hacer clic — la eliminación parcial no es posible.
-   Los IDs de acción que contienen guiones o guiones bajos se sanitizan automáticamente (limitación de enrutamiento de Mattermost).

Configuración:

-   `channels.mattermost.capabilities`: array de cadenas de capacidad. Agrega `"inlineButtons"` para habilitar la descripción de la herramienta de botones en el mensaje del sistema del agente.
-   `channels.mattermost.interactions.callbackBaseUrl`: URL base externa opcional para llamadas de botones (por ejemplo `https://gateway.example.com`). Úsala cuando Mattermost no pueda alcanzar la pasarela en su host de enlace directamente.
-   En configuraciones multi-cuenta, también puedes establecer el mismo campo bajo `channels.mattermost.accounts..interactions.callbackBaseUrl`.
-   Si se omite `interactions.callbackBaseUrl`, OpenClaw deriva la URL de callback de `gateway.customBindHost` + `gateway.port`, luego recurre a `http://localhost:`.
-   Regla de accesibilidad: la URL de callback del botón debe ser accesible desde el servidor de Mattermost. `localhost` solo funciona cuando Mattermost y OpenClaw se ejecutan en el mismo host/espacio de nombres de red.
-   Si tu destino de callback es privado/tailnet/internal, agrega su host/dominio a `ServiceSettings.AllowedUntrustedInternalConnections` de Mattermost.

### Integración directa de API (scripts externos)

Los scripts externos y webhooks pueden publicar botones directamente mediante la API REST de Mattermost en lugar de pasar por la herramienta `message` del agente. Usa `buildButtonAttachments()` de la extensión cuando sea posible; si publicas JSON crudo, sigue estas reglas: **Estructura del payload:**

```json
{
  channel_id: "<channelId>",
  message: "Elige una opción:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // solo alfanumérico — ver abajo
            type: "button", // requerido, o los clics se ignoran silenciosamente
            name: "Aprobar", // etiqueta de visualización
            style: "primary", // opcional: "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // debe coincidir con el id del botón (para búsqueda de nombre)
                action: "approve",
                // ... cualquier campo personalizado ...
                _token: "<hmac>", // ver sección HMAC abajo
              },
            },
          },
        ],
      },
    ],
  },
}
```

**Reglas críticas:**

1.  Los attachments van en `props.attachments`, no en `attachments` de nivel superior (se ignoran silenciosamente).
2.  Cada acción necesita `type: "button"` — sin ello, los clics se tragan silenciosamente.
3.  Cada acción necesita un campo `id` — Mattermost ignora acciones sin IDs.
4.  El `id` de la acción debe ser **solo alfanumérico** (`[a-zA-Z0-9]`). Los guiones y guiones bajos rompen el enrutamiento de acciones del lado del servidor de Mattermost (devuelve 404). Elimínalos antes de usar.
5.  `context.action_id` debe coincidir con el `id` del botón para que el mensaje de confirmación muestre el nombre del botón (ej., “Aprobar”) en lugar de un ID crudo.
6.  `context.action_id` es requerido — el manejador de interacciones devuelve 400 sin él.

**Generación de token HMAC:** La pasarela verifica los clics de botones con HMAC-SHA256. Los scripts externos deben generar tokens que coincidan con la lógica de verificación de la pasarela:

1.  Deriva el secreto del token del bot: `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  Construye el objeto de contexto con todos los campos **excepto** `_token`.
3.  Serializa con **claves ordenadas** y **sin espacios** (la pasarela usa `JSON.stringify` con claves ordenadas, lo que produce salida compacta).
4.  Firma: `HMAC-SHA256(key=secret, data=serializedContext)`
5.  Agrega el digest hexadecimal resultante como `_token` en el contexto.

Ejemplo en Python:

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

Problemas comunes con HMAC:

-   `json.dumps` de Python agrega espacios por defecto (`{"key": "val"}`). Usa `separators=(",", ":")` para coincidir con la salida compacta de JavaScript (`{"key":"val"}`).
-   Siempre firma **todos** los campos del contexto (menos `_token`). La pasarela elimina `_token` y luego firma todo lo que queda. Firmar un subconjunto causa fallo de verificación silencioso.
-   Usa `sort_keys=True` — la pasarela ordena las claves antes de firmar, y Mattermost puede reordenar los campos del contexto al almacenar el payload.
-   Deriva el secreto del token del bot (determinístico), no de bytes aleatorios. El secreto debe ser el mismo en el proceso que crea los botones y en la pasarela que verifica.

## Adaptador de directorio

El plugin de Mattermost incluye un adaptador de directorio que resuelve nombres de canales y usuarios mediante la API de Mattermost. Esto habilita destinos `#nombre-canal` y `@nombredeusuario` en `openclaw message send` y entregas cron/webhook. No se necesita configuración — el adaptador usa el token del bot de la configuración de la cuenta.

## Multi-cuenta

Mattermost admite múltiples cuentas bajo `channels.mattermost.accounts`:

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Principal", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alertas", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Solución de problemas

-   No hay respuestas en canales: asegúrate de que el bot esté en el canal y menciónalo (oncall), usa un prefijo desencadenante (onchar), o establece `chatmode: "onmessage"`.
-   Errores de autenticación: verifica el token del bot, la URL base y si la cuenta está habilitada.
-   Problemas multi-cuenta: las variables de entorno solo se aplican a la cuenta `default`.
-   Los botones aparecen como cajas blancas: el agente podría estar enviando datos de botón malformados. Verifica que cada botón tenga los campos `text` y `callback_data`.
-   Los botones se renderizan pero los clics no hacen nada: verifica que `AllowedUntrustedInternalConnections` en la configuración del servidor de Mattermost incluya `127.0.0.1 localhost`, y que `EnablePostActionIntegration` sea `true` en ServiceSettings.
-   Los botones devuelven 404 al hacer clic: es probable que el `id` del botón contenga guiones o guiones bajos. El enrutador de acciones de Mattermost falla con IDs no alfanuméricos. Usa solo `[a-zA-Z0-9]`.
-   La pasarela registra `invalid _token`: discrepancia HMAC. Verifica que firmes todos los campos del contexto (no un subconjunto), uses claves ordenadas y uses JSON compacto (sin espacios). Consulta la sección HMAC anterior.
-   La pasarela registra `missing _token in context`: el campo `_token` no está en el contexto del botón. Asegúrate de incluirlo al construir el payload de integración.
-   La confirmación muestra ID crudo en lugar del nombre del botón: `context.action_id` no coincide con el `id` del botón. Establece ambos al mismo valor sanitizado.
-   El agente no sabe sobre botones: agrega `capabilities: ["inlineButtons"]` a la configuración del canal de Mattermost.

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)