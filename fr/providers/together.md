title: "Guide de configuration du fournisseur Together AI et des modèles dans OpenClaw"
description: "Apprenez à configurer le fournisseur Together AI dans OpenClaw, définir votre clé API et accéder aux modèles comme Llama, DeepSeek et Kimi."
keywords: ["together ai", "fournisseur openclaw", "modèle llama", "deepseek", "kimi ai", "api compatible openai", "configuration modèle", "configuration clé api"]
---

  Fournisseurs

  
# Together

La plateforme [Together AI](https://together.ai) donne accès à des modèles open-source de premier plan, dont Llama, DeepSeek, Kimi, et bien d'autres, via une API unifiée.

-   Fournisseur : `together`
-   Authentification : `TOGETHER_API_KEY`
-   API : Compatible OpenAI

## Démarrage rapide

1.  Définissez la clé API (recommandé : la stocker pour la Gateway) :

```bash
openclaw onboard --auth-choice together-api-key
```

2.  Définissez un modèle par défaut :

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Cela définira `together/moonshotai/Kimi-K2.5` comme modèle par défaut.

## Note sur l'environnement

Si la Gateway s'exécute en tant que démon (launchd/systemd), assurez-vous que `TOGETHER_API_KEY` est disponible pour ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

## Modèles disponibles

Together AI donne accès à de nombreux modèles open-source populaires :

-   **GLM 4.7 Fp8** - Modèle par défaut avec une fenêtre de contexte de 200K
-   **Llama 3.3 70B Instruct Turbo** - Suivi d'instructions rapide et efficace
-   **Llama 4 Scout** - Modèle de vision avec compréhension d'images
-   **Llama 4 Maverick** - Vision et raisonnement avancés
-   **DeepSeek V3.1** - Modèle puissant pour le codage et le raisonnement
-   **DeepSeek R1** - Modèle de raisonnement avancé
-   **Kimi K2 Instruct** - Modèle haute performance avec une fenêtre de contexte de 262K

Tous les modèles prennent en charge les complétions de chat standard et sont compatibles avec l'API OpenAI.

[Synthetic](./synthetic.md)[Vercel AI Gateway](./vercel-ai-gateway.md)

---