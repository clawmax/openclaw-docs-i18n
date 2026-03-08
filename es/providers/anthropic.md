

  Proveedores

  
# Anthropic

Anthropic desarrolla la familia de modelos **Claude** y proporciona acceso a través de una API. En OpenClaw puedes autenticarte con una clave API o un **setup-token**.

## Opción A: Clave API de Anthropic

**Ideal para:** acceso estándar a la API y facturación por uso. Crea tu clave API en la Consola de Anthropic.

### Configuración por CLI

```bash
openclaw onboard
# elige: Anthropic API key

# o no interactivo
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### Fragmento de configuración

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Valores predeterminados de pensamiento (Claude 4.6)

-   Los modelos Anthropic Claude 4.6 usan por defecto el pensamiento `adaptive` en OpenClaw cuando no se establece un nivel de pensamiento explícito.
-   Puedes anularlo por mensaje (`/think:`) o en los parámetros del modelo: `agents.defaults.models["anthropic/"].params.thinking`.
-   Documentación relacionada de Anthropic:
    -   [Pensamiento adaptativo](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [Pensamiento extendido](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## Almacenamiento en caché de prompts (API de Anthropic)

OpenClaw admite la función de almacenamiento en caché de prompts de Anthropic. Esto es **solo para la API**; la autenticación por suscripción no respeta la configuración de caché.

### Configuración

Usa el parámetro `cacheRetention` en la configuración de tu modelo:

| Valor | Duración del Caché | Descripción |
| --- | --- | --- |
| `none` | Sin caché | Deshabilita el almacenamiento en caché de prompts |
| `short` | 5 minutos | Predeterminado para autenticación con Clave API |
| `long` | 1 hora | Caché extendido (requiere bandera beta) |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### Valores predeterminados

Cuando se usa autenticación con Clave API de Anthropic, OpenClaw aplica automáticamente `cacheRetention: "short"` (caché de 5 minutos) para todos los modelos de Anthropic. Puedes anular esto estableciendo explícitamente `cacheRetention` en tu configuración.

### Anulaciones de cacheRetention por agente

Usa los parámetros a nivel de modelo como base, luego anula agentes específicos a través de `agents.list[].params`.

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // base para la mayoría de agentes
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // anulación solo para este agente
    ],
  },
}
```

Orden de fusión de configuración para parámetros relacionados con caché:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (coincidencia por `id`, anula por clave)

Esto permite que un agente mantenga un caché de larga duración mientras otro agente en el mismo modelo deshabilita el caché para evitar costos de escritura en tráfico con ráfagas o bajo reutilización.

### Notas sobre Bedrock Claude

-   Los modelos Anthropic Claude en Bedrock (`amazon-bedrock/*anthropic.claude*`) aceptan el pase de `cacheRetention` cuando están configurados.
-   Los modelos Bedrock no de Anthropic son forzados a `cacheRetention: "none"` en tiempo de ejecución.
-   Los valores predeterminados inteligentes de la clave API de Anthropic también establecen `cacheRetention: "short"` para las referencias de modelos Claude-on-Bedrock cuando no se establece un valor explícito.

### Parámetro heredado

El parámetro anterior `cacheControlTtl` todavía es compatible por retrocompatibilidad:

-   `"5m"` se asigna a `short`
-   `"1h"` se asigna a `long`

Recomendamos migrar al nuevo parámetro `cacheRetention`. OpenClaw incluye la bandera beta `extended-cache-ttl-2025-04-11` para las solicitudes a la API de Anthropic; mantenla si anulas las cabeceras del proveedor (ver [/gateway/configuration](../gateway/configuration.md)).

## Ventana de contexto de 1M (beta de Anthropic)

La ventana de contexto de 1M de Anthropic está en fase beta. En OpenClaw, habilítala por modelo con `params.context1m: true` para los modelos Opus/Sonnet compatibles.

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw asigna esto a `anthropic-beta: context-1m-2025-08-07` en las solicitudes a Anthropic. Esto solo se activa cuando `params.context1m` se establece explícitamente en `true` para ese modelo. Requisito: Anthropic debe permitir el uso de contexto largo en esa credencial (típicamente facturación por clave API, o una cuenta de suscripción con Uso Extra habilitado). De lo contrario, Anthropic devuelve: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. Nota: Anthropic actualmente rechaza las solicitudes beta `context-1m-*` cuando se usan tokens OAuth/de suscripción (`sk-ant-oat-*`). OpenClaw omite automáticamente la cabecera beta context1m para autenticación OAuth y mantiene las betas OAuth requeridas.

## Opción B: Setup-token de Claude

**Ideal para:** usar tu suscripción a Claude.

### Dónde obtener un setup-token

Los setup-tokens son creados por la **CLI de Claude Code**, no por la Consola de Anthropic. Puedes ejecutar esto en **cualquier máquina**:

```bash
claude setup-token
```

Pega el token en OpenClaw (asistente: **Anthropic token (paste setup-token)**), o ejecútalo en el host de la puerta de enlace:

```bash
openclaw models auth setup-token --provider anthropic
```

Si generaste el token en una máquina diferente, pégalo:

```bash
openclaw models auth paste-token --provider anthropic
```

### Configuración por CLI (setup-token)

```bash
# Pega un setup-token durante la incorporación
openclaw onboard --auth-choice setup-token
```

### Fragmento de configuración (setup-token)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Notas

-   Genera el setup-token con `claude setup-token` y pégalo, o ejecuta `openclaw models auth setup-token` en el host de la puerta de enlace.
-   Si ves "OAuth token refresh failed …" en una suscripción a Claude, reautentícate con un setup-token. Ver [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription).
-   Los detalles de autenticación + reglas de reutilización están en [/concepts/oauth](../concepts/oauth.md).

## Solución de problemas

**Errores 401 / token repentinamente inválido**

-   La autenticación por suscripción a Claude puede expirar o ser revocada. Vuelve a ejecutar `claude setup-token` y pégalo en el **host de la puerta de enlace**.
-   Si el inicio de sesión de la CLI de Claude está en una máquina diferente, usa `openclaw models auth paste-token --provider anthropic` en el host de la puerta de enlace.

**No se encontró clave API para el proveedor “anthropic”**

-   La autenticación es **por agente**. Los nuevos agentes no heredan las claves del agente principal.
-   Vuelve a ejecutar la incorporación para ese agente, o pega un setup-token / clave API en el host de la puerta de enlace, luego verifica con `openclaw models status`.

**No se encontraron credenciales para el perfil `anthropic:default`**

-   Ejecuta `openclaw models status` para ver qué perfil de autenticación está activo.
-   Vuelve a ejecutar la incorporación, o pega un setup-token / clave API para ese perfil.

**No hay perfil de autenticación disponible (todos en enfriamiento/no disponibles)**

-   Revisa `openclaw models status --json` para ver `auth.unusableProfiles`.
-   Añade otro perfil de Anthropic o espera a que pase el tiempo de enfriamiento.

Más: [/gateway/troubleshooting](../gateway/troubleshooting.md) y [/help/faq](../help/faq.md).

[Conmutación por error de modelo](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)