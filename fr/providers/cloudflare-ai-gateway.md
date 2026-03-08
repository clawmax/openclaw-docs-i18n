

  Fournisseurs

  
# Cloudflare AI Gateway

Cloudflare AI Gateway se place devant les API des fournisseurs et vous permet d'ajouter des analyses, de la mise en cache et des contrôles. Pour Anthropic, OpenClaw utilise l'API Anthropic Messages via votre point de terminaison de passerelle.

-   Fournisseur : `cloudflare-ai-gateway`
-   URL de base : `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   Modèle par défaut : `cloudflare-ai-gateway/claude-sonnet-4-5`
-   Clé API : `CLOUDFLARE_AI_GATEWAY_API_KEY` (votre clé API de fournisseur pour les requêtes via la passerelle)

Pour les modèles Anthropic, utilisez votre clé API Anthropic.

## Démarrage rapide

1.  Définissez la clé API du fournisseur et les détails de la passerelle :

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  Définissez un modèle par défaut :

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## Passerelles authentifiées

Si vous avez activé l'authentification de la passerelle dans Cloudflare, ajoutez l'en-tête `cf-aig-authorization` (cela s'ajoute à votre clé API de fournisseur).

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

## Note sur l'environnement

Si la passerelle s'exécute en tant que démon (launchd/systemd), assurez-vous que `CLOUDFLARE_AI_GATEWAY_API_KEY` est disponible pour ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

[Amazon Bedrock](./bedrock.md)[Proxy API Claude Max](./claude-max-api-proxy.md)

---