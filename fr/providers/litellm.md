title: "Guide d'installation et de configuration de l'intégration OpenClaw avec LiteLLM"
description: "Apprenez à router OpenClaw via LiteLLM pour un suivi centralisé des coûts, un routage des modèles et une journalisation. Configurez les fournisseurs et gérez les clés virtuelles."
keywords: ["litellm", "openclaw", "routage de modèles", "suivi des coûts", "passerelle llm", "clés virtuelles", "openai-compatible", "intégration api"]
---

  Fournisseurs

  
# Litellm

[LiteLLM](https://litellm.ai) est une passerelle LLM open-source qui fournit une API unifiée vers plus de 100 fournisseurs de modèles. Routez OpenClaw via LiteLLM pour obtenir un suivi centralisé des coûts, une journalisation et la flexibilité de changer de backends sans modifier votre configuration OpenClaw.

## Pourquoi utiliser LiteLLM avec OpenClaw ?

-   **Suivi des coûts** — Voyez exactement ce que dépense OpenClaw sur tous les modèles
-   **Routage de modèles** — Passez entre Claude, GPT-4, Gemini, Bedrock sans changer la configuration
-   **Clés virtuelles** — Créez des clés avec des limites de dépenses pour OpenClaw
-   **Journalisation** — Logs complets des requêtes/réponses pour le débogage
-   **Basculement** — Reprise automatique si votre fournisseur principal est indisponible

## Démarrage rapide

### Via l'intégration guidée

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Configuration manuelle

1.  Démarrez le proxy LiteLLM :

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2.  Dirigez OpenClaw vers LiteLLM :

```bash
export LITELLM_API_KEY="votre-clé-litellm"

openclaw
```

C'est tout. OpenClaw route désormais via LiteLLM.

## Configuration

### Variables d'environnement

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Fichier de configuration

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Clés virtuelles

Créez une clé dédiée pour OpenClaw avec des limites de dépenses :

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Utilisez la clé générée comme `LITELLM_API_KEY`.

## Routage de modèles

LiteLLM peut router les requêtes de modèles vers différents backends. Configurez cela dans votre `config.yaml` LiteLLM :

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw continue de demander `claude-opus-4-6` — LiteLLM gère le routage.

## Consultation de l'utilisation

Consultez le tableau de bord ou l'API de LiteLLM :

```bash
# Informations sur la clé
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Logs des dépenses
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Notes

-   LiteLLM s'exécute sur `http://localhost:4000` par défaut
-   OpenClaw se connecte via le point de terminaison compatible OpenAI `/v1/chat/completions`
-   Toutes les fonctionnalités d'OpenClaw fonctionnent via LiteLLM — aucune limitation

## Voir aussi

-   [Documentation LiteLLM](https://docs.litellm.ai)
-   [Fournisseurs de modèles](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[Modèles GLM](./glm.md)