

  Fournisseurs

  
# vLLM

vLLM peut servir des modèles open-source (et certains modèles personnalisés) via une API HTTP **compatible OpenAI**. OpenClaw peut se connecter à vLLM en utilisant l'API `openai-completions`. OpenClaw peut également **découvrir automatiquement** les modèles disponibles depuis vLLM lorsque vous optez pour cette fonctionnalité avec `VLLM_API_KEY` (n'importe quelle valeur fonctionne si votre serveur n'applique pas d'authentification) et que vous ne définissez pas d'entrée explicite `models.providers.vllm`.

## Démarrage rapide

1.  Démarrez vLLM avec un serveur compatible OpenAI.

Votre URL de base doit exposer les points de terminaison `/v1` (par ex. `/v1/models`, `/v1/chat/completions`). vLLM s'exécute généralement sur :

-   `http://127.0.0.1:8000/v1`

2.  Optez pour la découverte automatique (n'importe quelle valeur fonctionne si aucune authentification n'est configurée) :

```bash
export VLLM_API_KEY="vllm-local"
```

3.  Sélectionnez un modèle (remplacez par l'un de vos identifiants de modèle vLLM) :

```json
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Découverte de modèles (fournisseur implicite)

Lorsque `VLLM_API_KEY` est défini (ou qu'un profil d'authentification existe) et que vous **ne définissez pas** `models.providers.vllm`, OpenClaw interrogera :

-   `GET http://127.0.0.1:8000/v1/models`

…et convertira les identifiants renvoyés en entrées de modèle. Si vous définissez `models.providers.vllm` explicitement, la découverte automatique est ignorée et vous devez définir les modèles manuellement.

## Configuration explicite (modèles manuels)

Utilisez une configuration explicite lorsque :

-   vLLM s'exécute sur un hôte/port différent.
-   Vous souhaitez fixer les valeurs `contextWindow`/`maxTokens`.
-   Votre serveur nécessite une véritable clé API (ou vous souhaitez contrôler les en-têtes).

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Modèle vLLM Local",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Dépannage

-   Vérifiez que le serveur est accessible :

```bash
curl http://127.0.0.1:8000/v1/models
```

-   Si les requêtes échouent avec des erreurs d'authentification, définissez une véritable `VLLM_API_KEY` qui correspond à la configuration de votre serveur, ou configurez le fournisseur explicitement sous `models.providers.vllm`.

[Venice AI](./venice.md)[Xiaomi MiMo](./xiaomi.md)

---