

  Fundamentos

  
# OAuth

OpenClaw admite la "autenticación por suscripción" a través de OAuth para los proveedores que la ofrecen (notablemente **OpenAI Codex (ChatGPT OAuth)**). Para suscripciones de Anthropic, usa el flujo **setup-token**. El uso de suscripciones de Anthropic fuera de Claude Code ha sido restringido para algunos usuarios en el pasado, así que trátalo como un riesgo de elección del usuario y verifica tú mismo la política actual de Anthropic. El OAuth de OpenAI Codex está explícitamente soportado para su uso en herramientas externas como OpenClaw. Esta página explica: Para Anthropic en producción, la autenticación por clave API es el camino más seguro recomendado sobre la autenticación por setup-token de suscripción.

-   cómo funciona el **intercambio de tokens** OAuth (PKCE)
-   dónde se **almacenan** los tokens (y por qué)
-   cómo manejar **múltiples cuentas** (perfiles + anulaciones por sesión)

OpenClaw también admite **plugins de proveedor** que incluyen sus propios flujos OAuth o de clave API. Ejecútalos mediante:

```bash
openclaw models auth login --provider <id>
```

## El sumidero de tokens (por qué existe)

Los proveedores OAuth comúnmente generan un **nuevo token de actualización** durante los flujos de inicio de sesión/actualización. Algunos proveedores (o clientes OAuth) pueden invalidar tokens de actualización más antiguos cuando se emite uno nuevo para el mismo usuario/aplicación. Síntoma práctico:

-   inicias sesión a través de OpenClaw *y* a través de Claude Code / Codex CLI → uno de ellos se "desconecta" aleatoriamente más tarde

Para reducir eso, OpenClaw trata `auth-profiles.json` como un **sumidero de tokens**:

-   el tiempo de ejecución lee las credenciales desde **un solo lugar**
-   podemos mantener múltiples perfiles y enrutarlos de manera determinista

## Almacenamiento (dónde viven los tokens)

Los secretos se almacenan **por agente**:

-   Perfiles de autenticación (OAuth + claves API + referencias opcionales a nivel de valor): `~/.openclaw/agents//agent/auth-profiles.json`
-   Archivo de compatibilidad heredado: `~/.openclaw/agents//agent/auth.json` (las entradas estáticas `api_key` se eliminan cuando se descubren)

Archivo de importación heredado (todavía soportado, pero no es el almacén principal):

-   `~/.openclaw/credentials/oauth.json` (importado a `auth-profiles.json` en el primer uso)

Todos los anteriores también respetan `$OPENCLAW_STATE_DIR` (anulación del directorio de estado). Referencia completa: [/gateway/configuration](../gateway/configuration.md#auth-storage-oauth--api-keys) Para referencias estáticas de secretos y el comportamiento de activación de instantáneas en tiempo de ejecución, consulta [Gestión de Secretos](../gateway/secrets.md).

## Setup-token de Anthropic (autenticación por suscripción)

> **⚠️** El soporte de setup-token de Anthropic es una compatibilidad técnica, no una garantía de política. Anthropic ha bloqueado algunos usos de suscripción fuera de Claude Code en el pasado. Decide por ti mismo si usar la autenticación por suscripción y verifica los términos actuales de Anthropic.

 Ejecuta `claude setup-token` en cualquier máquina, luego pégala en OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Si generaste el token en otro lugar, pégala manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Verifica:

```bash
openclaw models status
```

## Intercambio OAuth (cómo funciona el inicio de sesión)

Los flujos de inicio de sesión interactivos de OpenClaw se implementan en `@mariozechner/pi-ai` y se conectan a los asistentes/comandos.

### Setup-token de Anthropic

Forma del flujo:

1.  ejecuta `claude setup-token`
2.  pega el token en OpenClaw
3.  almacénalo como un perfil de autenticación de token (sin actualización)

La ruta del asistente es `openclaw onboard` → opción de autenticación `setup-token` (Anthropic).

### OpenAI Codex (ChatGPT OAuth)

El OAuth de OpenAI Codex está explícitamente soportado para su uso fuera de la CLI de Codex, incluyendo flujos de trabajo de OpenClaw. Forma del flujo (PKCE):

1.  genera verificador/desafío PKCE + `state` aleatorio
2.  abre `https://auth.openai.com/oauth/authorize?...`
3.  intenta capturar la devolución de llamada en `http://127.0.0.1:1455/auth/callback`
4.  si la devolución de llamada no puede vincularse (o estás remoto/sin interfaz gráfica), pega la URL/código de redirección
5.  intercambia en `https://auth.openai.com/oauth/token`
6.  extrae `accountId` del token de acceso y almacena `{ access, refresh, expires, accountId }`

La ruta del asistente es `openclaw onboard` → opción de autenticación `openai-codex`.

## Actualización + caducidad

Los perfiles almacenan una marca de tiempo `expires`. En tiempo de ejecución:

-   si `expires` está en el futuro → usa el token de acceso almacenado
-   si ha caducado → actualiza (bajo un bloqueo de archivo) y sobrescribe las credenciales almacenadas

El flujo de actualización es automático; generalmente no necesitas gestionar los tokens manualmente.

## Múltiples cuentas (perfiles) + enrutamiento

Dos patrones:

### 1) Preferido: agentes separados

Si quieres que "personal" y "trabajo" nunca interactúen, usa agentes aislados (sesiones + credenciales + espacio de trabajo separados):

```bash
openclaw agents add work
openclaw agents add personal
```

Luego configura la autenticación por agente (asistente) y enruta los chats al agente correcto.

### 2) Avanzado: múltiples perfiles en un agente

`auth-profiles.json` admite múltiples IDs de perfil para el mismo proveedor. Elige qué perfil se usa:

-   globalmente a través del orden de configuración (`auth.order`)
-   por sesión a través de `/model ...@`

Ejemplo (anulación por sesión):

-   `/model Opus@anthropic:work`

Cómo ver qué IDs de perfil existen:

-   `openclaw channels list --json` (muestra `auth[]`)

Documentación relacionada:

-   [/concepts/model-failover](./model-failover.md) (reglas de rotación + enfriamiento)
-   [/tools/slash-commands](../tools/slash-commands.md) (superficie de comandos)

[Espacio de Trabajo del Agente](./agent-workspace.md)[Arranque](../start/bootstrapping.md)