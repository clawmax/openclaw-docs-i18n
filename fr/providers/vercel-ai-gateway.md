title: "Configuration et paramétrage du fournisseur Vercel AI Gateway pour OpenClaw"
description: "Apprenez à configurer OpenClaw avec la passerelle IA Vercel. Configurez votre clé API, définissez des modèles par défaut comme Claude Opus et utilisez les raccourcis de modèles pour un accès unifié à l'IA."
keywords: ["passerelle ia vercel", "configuration openclaw", "claude opus", "clé api passerelle ia", "raccourci modèle", "api messages anthropic", "fournisseurs openclaw", "intégration modèle ia"]
---

  Fournisseurs

  
# Vercel AI Gateway

La [Vercel AI Gateway](https://vercel.com/ai-gateway) fournit une API unifiée pour accéder à des centaines de modèles via un seul point de terminaison.

-   Fournisseur : `vercel-ai-gateway`
-   Authentification : `AI_GATEWAY_API_KEY`
-   API : Compatible Anthropic Messages

## Démarrage rapide

1.  Définissez la clé API (recommandé : la stocker pour la passerelle) :

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  Définissez un modèle par défaut :

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Note sur l'environnement

Si la passerelle fonctionne en tant que démon (launchd/systemd), assurez-vous que `AI_GATEWAY_API_KEY` est disponible pour ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

## Raccourci d'identifiant de modèle

OpenClaw accepte les références de modèles Claude abrégées de Vercel et les normalise à l'exécution :

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)