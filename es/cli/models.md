

  Comandos CLI

  
# models

Descubrimiento, escaneo y configuración de modelos (modelo predeterminado, respaldos, perfiles de autenticación). Relacionado:

-   Proveedores + modelos: [Models](../providers/models.md)
-   Configuración de autenticación del proveedor: [Getting started](../start/getting-started.md)

## Comandos comunes

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` muestra los respaldos/predeterminados resueltos junto con un resumen de autenticación. Cuando hay instantáneas de uso del proveedor disponibles, la sección de estado OAuth/token incluye encabezados de uso del proveedor. Agrega `--probe` para ejecutar sondeos de autenticación en vivo contra cada perfil de proveedor configurado. Los sondeos son solicitudes reales (pueden consumir tokens y activar límites de tasa). Usa `--agent ` para inspeccionar el estado del modelo/autenticación de un agente configurado. Cuando se omite, el comando usa `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` si está configurado, de lo contrario el agente predeterminado configurado. Notas:

-   `models set <model-or-alias>` acepta `provider/model` o un alias.
-   Las referencias de modelo se analizan dividiendo en la **primera** `/`. Si el ID del modelo incluye `/` (estilo OpenRouter), incluye el prefijo del proveedor (ejemplo: `openrouter/moonshotai/kimi-k2`).
-   Si omites el proveedor, OpenClaw trata la entrada como un alias o un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).
-   `models status` puede mostrar `marker()` en la salida de autenticación para marcadores de posición no secretos (por ejemplo `OPENAI_API_KEY`, `secretref-managed`, `minimax-oauth`, `qwen-oauth`, `ollama-local`) en lugar de enmascararlos como secretos.

### models status

Opciones:

-   `--json`
-   `--plain`
-   `--check` (salida 1=caducado/faltante, 2=por caducar)
-   `--probe` (sondeo en vivo de perfiles de autenticación configurados)
-   `--probe-provider ` (sondea un proveedor)
-   `--probe-profile ` (repetir o IDs de perfil separados por comas)
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `
-   `--agent ` (ID de agente configurado; anula `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

## Alias + respaldos

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## Perfiles de autenticación

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` ejecuta el flujo de autenticación de un plugin del proveedor (OAuth/clave API). Usa `openclaw plugins list` para ver qué proveedores están instalados. Notas:

-   `setup-token` solicita un valor de token de configuración (genéralo con `claude setup-token` en cualquier máquina).
-   `paste-token` acepta una cadena de token generada en otro lugar o desde automatización.
-   Nota de política de Anthropic: el soporte de setup-token es por compatibilidad técnica. Anthropic ha bloqueado previamente algunos usos de suscripción fuera de Claude Code, así que verifica los términos actuales antes de usarlo ampliamente.

[message](./message.md)[node](./node.md)