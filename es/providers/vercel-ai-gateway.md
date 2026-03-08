

  Proveedores

  
# Vercel AI Gateway

El [Vercel AI Gateway](https://vercel.com/ai-gateway) proporciona una API unificada para acceder a cientos de modelos a través de un único endpoint.

-   Proveedor: `vercel-ai-gateway`
-   Autenticación: `AI_GATEWAY_API_KEY`
-   API: Compatible con Anthropic Messages

## Inicio rápido

1.  Establece la clave API (recomendado: guárdala para el Gateway):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  Establece un modelo predeterminado:

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Ejemplo no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Nota sobre el entorno

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que `AI_GATEWAY_API_KEY` esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o a través de `env.shellEnv`).

## Abreviatura de ID de modelo

OpenClaw acepta referencias abreviadas de modelos Claude de Vercel y las normaliza en tiempo de ejecución:

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)

---