

  Proveedores

  
# Cloudflare AI Gateway

Cloudflare AI Gateway se sitúa delante de las APIs de los proveedores y te permite agregar análisis, caché y controles. Para Anthropic, OpenClaw utiliza la API de Mensajes de Anthropic a través de tu endpoint del Gateway.

-   Proveedor: `cloudflare-ai-gateway`
-   URL base: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   Modelo predeterminado: `cloudflare-ai-gateway/claude-sonnet-4-5`
-   Clave API: `CLOUDFLARE_AI_GATEWAY_API_KEY` (tu clave API del proveedor para solicitudes a través del Gateway)

Para los modelos de Anthropic, usa tu clave API de Anthropic.

## Inicio rápido

1.  Configura la clave API del proveedor y los detalles del Gateway:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  Establece un modelo predeterminado:

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## Ejemplo no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## Gateways autenticados

Si habilitaste la autenticación del Gateway en Cloudflare, agrega el encabezado `cf-aig-authorization` (esto es adicional a tu clave API del proveedor).

```json
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## Nota sobre el entorno

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que `CLOUDFLARE_AI_GATEWAY_API_KEY` esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o a través de `env.shellEnv`).

[Amazon Bedrock](./bedrock.md)[Claude Max API Proxy](./claude-max-api-proxy.md)