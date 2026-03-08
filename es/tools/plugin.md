title: "Guía de Plugins de Habilidades de OpenClaw Instalación Configuración y SDK"
description: "Aprende a instalar, configurar y desarrollar plugins para OpenClaw. Extiende la funcionalidad con plugins oficiales, rutas HTTP y ayudantes de tiempo de ejecución."
keywords: ["plugins de openclaw", "plugins de habilidades", "instalación de plugins", "sdk de plugins", "gateway rpc", "herramientas de agente", "manifiesto de plugin", "ayudantes de tiempo de ejecución"]
---

  Habilidades

  
# Plugins

## Inicio rápido (¿nuevo en plugins?)

Un plugin es solo un **pequeño módulo de código** que extiende OpenClaw con características adicionales (comandos, herramientas y Gateway RPC). La mayoría de las veces, usarás plugins cuando quieras una característica que aún no está integrada en el núcleo de OpenClaw (o quieras mantener características opcionales fuera de tu instalación principal). Ruta rápida:

1.  Ver qué ya está cargado:

```bash
openclaw plugins list
```

2.  Instalar un plugin oficial (ejemplo: Llamada de Voz):

```bash
openclaw plugins install @openclaw/voice-call
```

Las especificaciones de Npm son **solo de registro** (nombre del paquete + **versión exacta** opcional o **dist-tag**). Las especificaciones Git/URL/archivo y los rangos semver son rechazados. Las especificaciones simples y `@latest` permanecen en la vía estable. Si npm resuelve cualquiera de esos a una versión preliminar, OpenClaw se detiene y te pide que aceptes explícitamente con una etiqueta preliminar como `@beta`/`@rc` o una versión preliminar exacta.

3.  Reiniciar el Gateway, luego configurar bajo `plugins.entries..config`.

Consulta [Llamada de Voz](../plugins/voice-call.md) para un ejemplo concreto de plugin. ¿Buscas listados de terceros? Consulta [Plugins de la comunidad](../plugins/community.md).

## Plugins disponibles (oficiales)

-   Microsoft Teams es solo-plugin a partir de 2026.1.15; instala `@openclaw/msteams` si usas Teams.
-   Memoria (Núcleo) — plugin de búsqueda de memoria incluido (habilitado por defecto vía `plugins.slots.memory`)
-   Memoria (LanceDB) — plugin de memoria a largo plazo incluido (recuperación automática/captura; establece `plugins.slots.memory = "memory-lancedb"`)
-   [Llamada de Voz](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo Personal](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth (autenticación de proveedor) — incluido como `google-antigravity-auth` (deshabilitado por defecto)
-   Gemini CLI OAuth (autenticación de proveedor) — incluido como `google-gemini-cli-auth` (deshabilitado por defecto)
-   Qwen OAuth (autenticación de proveedor) — incluido como `qwen-portal-auth` (deshabilitado por defecto)
-   Copilot Proxy (autenticación de proveedor) — puente local del Proxy Copilot de VS Code; distinto del inicio de sesión de dispositivo `github-copilot` integrado (incluido, deshabilitado por defecto)

Los plugins de OpenClaw son **módulos TypeScript** cargados en tiempo de ejecución vía jiti. **La validación de configuración no ejecuta código del plugin**; usa el manifiesto del plugin y JSON Schema en su lugar. Consulta [Manifiesto del plugin](../plugins/manifest.md). Los plugins pueden registrar:

-   Métodos Gateway RPC
-   Rutas HTTP del Gateway
-   Herramientas del Agente
-   Comandos CLI
-   Servicios en segundo plano
-   Motores de contexto
-   Validación de configuración opcional
-   **Habilidades** (listando directorios `skills` en el manifiesto del plugin)
-   **Comandos de respuesta automática** (se ejecutan sin invocar al agente de IA)

Los plugins se ejecutan **en proceso** con el Gateway, así que trátalos como código confiable. Guía de creación de herramientas: [Herramientas del agente del plugin](../plugins/agent-tools.md).

## Ayudantes de tiempo de ejecución

Los plugins pueden acceder a ayudantes seleccionados del núcleo vía `api.runtime`. Para TTS de telefonía:

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notas:

-   Usa la configuración `messages.tts` del núcleo (OpenAI o ElevenLabs).
-   Devuelve búfer de audio PCM + frecuencia de muestreo. Los plugins deben remuestrear/codificar para los proveedores.
-   Edge TTS no es compatible para telefonía.

Para STT/transcripción, los plugins pueden llamar:

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // Opcional cuando el MIME no se puede inferir de forma confiable:
  mime: "audio/ogg",
});
```

Notas:

-   Usa la configuración de audio de comprensión de medios del núcleo (`tools.media.audio`) y el orden de respaldo del proveedor.
-   Devuelve `{ text: undefined }` cuando no se produce salida de transcripción (por ejemplo, entrada omitida/no compatible).

## Rutas HTTP del Gateway

Los plugins pueden exponer endpoints HTTP con `api.registerHttpRoute(...)`.

```
api.registerHttpRoute({
  path: "/acme/webhook",
  auth: "plugin",
  match: "exact",
  handler: async (_req, res) => {
    res.statusCode = 200;
    res.end("ok");
    return true;
  },
});
```

Campos de la ruta:

-   `path`: ruta bajo el servidor HTTP del gateway.
-   `auth`: requerido. Usa `"gateway"` para requerir autenticación normal del gateway, o `"plugin"` para autenticación gestionada por plugin/verificación de webhook.
-   `match`: opcional. `"exact"` (por defecto) o `"prefix"`.
-   `replaceExisting`: opcional. Permite que el mismo plugin reemplace su propio registro de ruta existente.
-   `handler`: devuelve `true` cuando la ruta manejó la solicitud.

Notas:

-   `api.registerHttpHandler(...)` está obsoleto. Usa `api.registerHttpRoute(...)`.
-   Las rutas del plugin deben declarar `auth` explícitamente.
-   Los conflictos exactos de `path + match` son rechazados a menos que `replaceExisting: true`, y un plugin no puede reemplazar la ruta de otro plugin.
-   Las rutas superpuestas con diferentes niveles de `auth` son rechazadas. Mantén las cadenas de caída `exact`/`prefix` solo en el mismo nivel de autenticación.

## Rutas de importación del SDK del Plugin

Usa subrutas del SDK en lugar de la importación monolítica `openclaw/plugin-sdk` al crear plugins:

-   `openclaw/plugin-sdk/core` para APIs genéricas de plugins, tipos de autenticación de proveedor y ayudantes compartidos.
-   `openclaw/plugin-sdk/compat` para código de plugin incluido/interno que necesita ayudantes de tiempo de ejecución compartidos más amplios que `core`.
-   `openclaw/plugin-sdk/telegram` para plugins del canal Telegram.
-   `openclaw/plugin-sdk/discord` para plugins del canal Discord.
-   `openclaw/plugin-sdk/slack` para plugins del canal Slack.
-   `openclaw/plugin-sdk/signal` para plugins del canal Signal.
-   `openclaw/plugin-sdk/imessage` para plugins del canal iMessage.
-   `openclaw/plugin-sdk/whatsapp` para plugins del canal WhatsApp.
-   `openclaw/plugin-sdk/line` para plugins del canal LINE.
-   `openclaw/plugin-sdk/msteams` para la superficie del plugin de Microsoft Teams incluido.
-   También están disponibles subrutas específicas de extensiones incluidas: `openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo`, y `openclaw/plugin-sdk/zalouser`.

Nota de compatibilidad:

-   `openclaw/plugin-sdk` sigue siendo compatible para plugins externos existentes.
-   Los plugins incluidos nuevos y migrados deben usar subrutas específicas de canal o extensión; usa `core` para superficies genéricas y `compat` solo cuando se requieran ayudantes compartidos más amplios.

## Inspección de canal de solo lectura

Si tu plugin registra un canal, prefiere implementar `plugin.config.inspectAccount(cfg, accountId)` junto con `resolveAccount(...)`. Por qué:

-   `resolveAccount(...)` es la ruta de tiempo de ejecución. Se le permite asumir que las credenciales están completamente materializadas y puede fallar rápidamente cuando faltan secretos requeridos.
-   Las rutas de comandos de solo lectura como `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, y flujos de reparación de doctor/config no deberían necesitar materializar credenciales de tiempo de ejecución solo para describir la configuración.

Comportamiento recomendado de `inspectAccount(...)`:

-   Devuelve solo el estado descriptivo de la cuenta.
-   Preserva `enabled` y `configured`.
-   Incluye campos de origen/estado de credenciales cuando sea relevante, como:
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   No necesitas devolver valores de token sin procesar solo para informar disponibilidad de solo lectura. Devolver `tokenStatus: "available"` (y el campo de origen coincidente) es suficiente para comandos de tipo estado.
-   Usa `configured_unavailable` cuando una credencial está configurada vía SecretRef pero no está disponible en la ruta de comando actual.

Esto permite que los comandos de solo lectura informen "configurado pero no disponible en esta ruta de comando" en lugar de fallar o informar incorrectamente la cuenta como no configurada. Nota de rendimiento:

-   El descubrimiento de plugins y los metadatos del manifiesto usan cachés cortas en proceso para reducir el trabajo de inicio/recarga en ráfagas.
-   Establece `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` o `OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1` para deshabilitar estas cachés.
-   Ajusta las ventanas de caché con `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` y `OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`.

## Descubrimiento y precedencia

OpenClaw escanea, en orden:

1.  Rutas de configuración

-   `plugins.load.paths` (archivo o directorio)

2.  Extensiones del espacio de trabajo

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  Extensiones globales

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  Extensiones incluidas (incluidas con OpenClaw, mayormente deshabilitadas por defecto)

-   `/extensions/*`

La mayoría de los plugins incluidos deben habilitarse explícitamente vía `plugins.entries..enabled` o `openclaw plugins enable `. Excepciones de plugins incluidos activados por defecto:

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   plugin de ranura de memoria activa (ranura por defecto: `memory-core`)

Los plugins instalados están habilitados por defecto, pero pueden deshabilitarse de la misma manera. Notas de endurecimiento:

-   Si `plugins.allow` está vacío y hay plugins no incluidos descubribles, OpenClaw registra una advertencia de inicio con ids de plugins y fuentes.
-   Las rutas candidatas se verifican por seguridad antes de la admisión de descubrimiento. OpenClaw bloquea candidatos cuando:
    -   la entrada de extensión se resuelve fuera de la raíz del plugin (incluyendo escapes de enlace simbólico/travesía de ruta),
    -   la ruta raíz/fuente del plugin es escribible por todos,
    -   la propiedad de la ruta es sospechosa para plugins no incluidos (el propietario POSIX no es ni el uid actual ni root).
-   Los plugins no incluidos cargados sin procedencia de instalación/ruta de carga emiten una advertencia para que puedas fijar la confianza (`plugins.allow`) o el seguimiento de instalación (`plugins.installs`).

Cada plugin debe incluir un archivo `openclaw.plugin.json` en su raíz. Si una ruta apunta a un archivo, la raíz del plugin es el directorio del archivo y debe contener el manifiesto. Si múltiples plugins se resuelven al mismo id, la primera coincidencia en el orden anterior gana y las copias de menor precedencia se ignoran.

### Paquetes de packs

Un directorio de plugin puede incluir un `package.json` con `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Cada entrada se convierte en un plugin. Si el pack lista múltiples extensiones, el id del plugin se convierte en `name/`. Si tu plugin importa dependencias de npm, instálalas en ese directorio para que `node_modules` esté disponible (`npm install` / `pnpm install`). Guardarraíl de seguridad: cada entrada de `openclaw.extensions` debe permanecer dentro del directorio del plugin después de la resolución de enlaces simbólicos. Las entradas que escapan del directorio del paquete son rechazadas. Nota de seguridad: `openclaw plugins install` instala dependencias del plugin con `npm install --ignore-scripts` (sin scripts de ciclo de vida). Mantén los árboles de dependencias del plugin "puro JS/TS" y evita paquetes que requieran compilaciones `postinstall`.

### Metadatos del catálogo de canales

Los plugins de canal pueden anunciar metadatos de incorporación vía `openclaw.channel` y sugerencias de instalación vía `openclaw.install`. Esto mantiene los datos del catálogo central libres de datos. Ejemplo:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (autoalojado)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Chat autoalojado vía bots de webhook de Nextcloud Talk.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw también puede fusionar **catálogos de canales externos** (por ejemplo, una exportación del registro MPM). Coloca un archivo JSON en uno de:

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

O apunta `OPENCLAW_PLUGIN_CATALOG_PATHS` (o `OPENCLAW_MPM_CATALOG_PATHS`) a uno o más archivos JSON (separados por comas/punto y coma/`PATH`). Cada archivo debe contener `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

## IDs de Plugin

Ids de plugin por defecto:

-   Paquetes de packs: `package.json` `name`
-   Archivo independiente: nombre base del archivo (`~/.../voice-call.ts` → `voice-call`)

Si un plugin exporta `id`, OpenClaw lo usa pero advierte cuando no coincide con el id configurado.

## Configuración

```json
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Campos:

-   `enabled`: interruptor maestro (por defecto: true)
-   `allow`: lista de permitidos (opcional)
-   `deny`: lista de denegados (opcional; denegar gana)
-   `load.paths`: archivos/directorios de plugin adicionales
-   `slots`: selectores de ranura exclusivos como `memory` y `contextEngine`
-   `entries.`: interruptores por plugin + configuración

Los cambios de configuración **requieren un reinicio del gateway**. Reglas de validación (estrictas):

-   Los ids de plugin desconocidos en `entries`, `allow`, `deny`, o `slots` son **errores**.
-   Las claves `channels.` desconocidas son **errores** a menos que un manifiesto de plugin declare el id del canal.
-   La configuración del plugin se valida usando el JSON Schema incrustado en `openclaw.plugin.json` (`configSchema`).
-   Si un plugin está deshabilitado, su configuración se preserva y se emite una **advertencia**.

## Ranuras de plugin (categorías exclusivas)

Algunas categorías de plugin son **exclusivas** (solo una activa a la vez). Usa `plugins.slots` para seleccionar qué plugin posee la ranura:

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // o "none" para deshabilitar plugins de memoria
      contextEngine: "legacy", // o un id de plugin como "lossless-claw"
    },
  },
}
```

Ranuras exclusivas compatibles:

-   `memory`: plugin de memoria activo (`"none"` deshabilita plugins de memoria)
-   `contextEngine`: plugin de motor de contexto activo (`"legacy"` es el valor por defecto integrado)

Si múltiples plugins declaran `kind: "memory"` o `kind: "context-engine"`, solo el plugin seleccionado se carga para esa ranura. Los otros se deshabilitan con diagnósticos.

### Plugins de motor de contexto

Los plugins de motor de contexto poseen la orquestación de contexto de sesión para ingesta, ensamblaje y compactación. Regístralos desde tu plugin con `api.registerContextEngine(id, factory)`, luego selecciona el motor activo con `plugins.slots.contextEngine`. Usa esto cuando tu plugin necesite reemplazar o extender la canalización de contexto por defecto en lugar de solo agregar búsqueda de memoria o ganchos.

## Interfaz de usuario de control (esquema + etiquetas)

La Interfaz de usuario de control usa `config.schema` (JSON Schema + `uiHints`) para renderizar mejores formularios. OpenClaw aumenta `uiHints` en tiempo de ejecución basado en plugins descubiertos:

-   Agrega etiquetas por plugin para `plugins.entries.` / `.enabled` / `.config`
-   Fusiona sugerencias de campos de configuración opcionales proporcionadas por el plugin bajo: `plugins.entries..config.`

Si quieres que los campos de configuración de tu plugin muestren buenas etiquetas/marcadores de posición (y marcar secretos como sensibles), proporciona `uiHints` junto con tu JSON Schema en el manifiesto del plugin. Ejemplo:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "Clave API", "sensitive": true },
    "region": { "label": "Región", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copia un archivo/directorio local en ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # ruta relativa ok
openclaw plugins install ./plugin.tgz           # instala desde un tarball local
openclaw plugins install ./plugin.zip           # instala desde un zip local
openclaw plugins install -l ./extensions/voice-call # enlace (sin copia) para desarrollo
openclaw plugins install @openclaw/voice-call # instala desde npm
openclaw plugins install @openclaw/voice-call --pin # almacena nombre@versión exacto resuelto
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` solo funciona para instalaciones de npm rastreadas bajo `plugins.installs`. Si los metadatos de integridad almacenados cambian entre actualizaciones, OpenClaw advierte y pide confirmación (usa el global `--yes` para omitir las solicitudes). Los plugins también pueden registrar sus propios comandos de nivel superior (ejemplo: `openclaw voicecall`).

## API del Plugin (resumen)

Los plugins exportan ya sea:

-   Una función: `(api) => { ... }`
-   Un objeto: `{ id, name, configSchema, register(api) { ... } }`

Los plugins de motor de contexto también pueden registrar un gestor de contexto propiedad del tiempo de ejecución:

```bash
export default function (api) {
  api.registerContextEngine("lossless-claw", () => ({
    info: { id: "lossless-claw", name: "Lossless Claw", ownsCompaction: true },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages }) {
      return { messages, estimatedTokens: 0 };
    },
    async compact() {
      return { ok: true, compacted: false };
    },
  }));
}
```

Luego habilítalo en la configuración:

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## Ganchos de Plugin

Los plugins pueden registrar ganchos en tiempo de ejecución. Esto permite que un plugin empaquete automatización impulsada por eventos sin una instalación separada de pack de ganchos.

### Ejemplo

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // Lógica del gancho aquí.
    },
    {
      name: "my-plugin.command-new",
      description: "Se ejecuta cuando se invoca /new",
    },
  );
}
```

Notas:

-   Registra ganchos explícitamente vía `api.registerHook(...)`.
-   Las reglas de elegibilidad de ganchos aún aplican (requisitos de SO/binarios/entorno/configuración).
-   Los ganchos gestionados por plugin aparecen en `openclaw hooks list` con `plugin:`.
-   No puedes habilitar/deshabilitar ganchos gestionados por plugin vía `openclaw hooks`; habilita/deshabilita el plugin en su lugar.

### Ganchos del ciclo de vida del agente (api.on)

Para ganchos de ciclo de vida de tiempo de ejecución tipados, usa `api.on(...)`:

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "Sigue la guía de estilo de la empresa.",
      };
    },
    { priority: 10 },
  );
}
```

Ganchos importantes para la construcción de prompts:

-   `before_model_resolve`: se ejecuta antes de la carga de la sesión (`messages` no están disponibles). Usa esto para anular determinísticamente `modelOverride` o `providerOverride`.
-   `before_prompt_build`: se ejecuta después de la carga de la sesión (`messages` están disponibles). Usa esto para dar forma a la entrada del prompt.
-   `before_agent_start`: gancho de compatibilidad heredado. Prefiere los dos ganchos explícitos anteriores.

Política de ganchos aplicada por el núcleo:

-   Los operadores pueden deshabilitar ganchos de mutación de prompt por plugin vía `plugins.entries..hooks.allowPromptInjection: false`.
-   Cuando está deshabilitado, OpenClaw bloquea `before_prompt_build` e ignora los campos de mutación de prompt devueltos desde el heredado `before_agent_start` mientras preserva los heredados `modelOverride` y `providerOverride`.

Campos de resultado de `before_prompt_build`:

-   `prependContext`: antepone texto al prompt del usuario para esta ejecución. Mejor para contenido específico por turno o dinámico.
-   `systemPrompt`: anulación completa del prompt del sistema.
-   `prependSystemContext`: antepone texto al prompt del sistema actual.
-   `appendSystemContext`: agrega texto al prompt del sistema actual.

Orden de construcción de prompt en tiempo de ejecución integrado:

1.  Aplica `prependContext` al prompt del usuario.
2.  Aplica la anulación `systemPrompt` cuando se proporciona.
3.  Aplica `prependSystemContext + prompt del sistema actual + appendSystemContext`.

Notas de fusión y precedencia:

-   Los manejadores de ganchos se ejecutan por prioridad (mayor primero).
-   Para campos de contexto fusionados, los valores se concatenan en orden de ejecución.
-   Los valores de `before_prompt_build` se aplican antes de los valores de respaldo heredados de `before_agent_start`.

Guía de migración:

-   Mueve la guía estática de `prependContext` a `prependSystemContext` (o `appendSystemContext`) para que los proveedores puedan almacenar en caché contenido estable de prefijo del sistema.
-   Mantén `prependContext` para contexto dinámico por turno que deba permanecer vinculado al mensaje del usuario.

## Plugins de proveedor (autenticación de modelo)

Los plugins pueden registrar flujos de **autenticación de proveedor de modelo** para que los usuarios puedan ejecutar OAuth o configuración de clave API dentro de OpenClaw (sin scripts externos necesarios). Registra un proveedor vía `api.registerProvider(...)`. Cada proveedor expone uno o más métodos de autenticación (OAuth, clave API, código de dispositivo, etc.). Estos métodos impulsan:

-   `openclaw models auth login --provider  [--method ]`

Ejemplo:

```
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Ejecuta flujo OAuth y devuelve perfiles de autenticación.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notas:

-   `run` recibe un `ProviderAuthContext` con ayudantes `prompter`, `runtime`, `openUrl`, y `oauth.createVpsAwareHandlers`.
-   Devuelve `configPatch` cuando necesites agregar modelos por defecto o configuración de proveedor.
-   Devuelve `defaultModel` para que `--set-default` pueda actualizar los valores por defecto del agente.

### Registrar un canal de mensajería

Los plugins pueden registrar **plugins de canal** que se comportan como canales integrados (WhatsApp, Telegram, etc.). La configuración del canal vive bajo `channels.` y es validada por tu código de plugin de canal.

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "plugin de canal de demostración.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notas:

-   Coloca la configuración bajo `channels.` (no `plugins.entries`).
-   `meta.label` se usa para etiquetas en listas CLI/UI.
-   `meta.aliases` agrega ids alternativos para normalización y entradas CLI.
-   `meta.preferOver` lista ids de canal para omitir la habilitación automática cuando ambos están configurados.
-   `meta.detailLabel` y `meta.systemImage` permiten que las UIs muestren etiquetas/iconos de canal más ricos.

### Ganchos de incorporación de canal

Los plugins de canal pueden definir ganchos de incorporación opcionales en `plugin.onboarding`:

-   `configure(ctx)` es el flujo de configuración básico.
-   `configureInteractive(ctx)` puede poseer completamente la configuración interactiva para estados configurados y no configurados.
-   `configureWhenConfigured(ctx)` puede anular el comportamiento solo para canales ya configurados.

Precedencia de ganchos en el asistente:

1.  `configureInteractive` (si está presente)
2.  `configureWhenConfigured` (solo cuando el estado del canal ya está configurado)
3.  respaldo a `configure`

Detalles del contexto:

-   `configureInteractive` y `configureWhenConfigured` reciben:
    -   `configured` (`true` o `false`)
    -   `label` (nombre del canal orientado al usuario usado por los prompts)
    -   más los campos compartidos config/runtime/prompter/options
-   Devolver `"skip"` deja la selección y el seguimiento de cuenta sin cambios.
-   Devolver `{ cfg, accountId? }` aplica actualizaciones de configuración y registra la selección de cuenta.

### Escribir un nuevo canal de mensajería (paso a paso)

Usa esto cuando quieras una **nueva superficie de chat** (un "canal de mensajería"), no un proveedor de modelo. La documentación del proveedor de modelo vive bajo `/prov