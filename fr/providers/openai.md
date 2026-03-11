

  Fournisseurs

  
# OpenAI

OpenAI fournit des API de développement pour les modèles GPT. Codex prend en charge la **connexion ChatGPT** pour un accès par abonnement ou la **connexion par clé API** pour un accès basé sur l'utilisation. Le cloud Codex nécessite une connexion ChatGPT. OpenAI prend explicitement en charge l'utilisation OAuth par abonnement dans des outils/flux de travail externes comme OpenClaw.

## Option A : Clé API OpenAI (OpenAI Platform)

**Idéal pour :** un accès direct à l'API et une facturation basée sur l'utilisation. Obtenez votre clé API depuis le tableau de bord OpenAI.

### Configuration en CLI

```bash
openclaw onboard --auth-choice openai-api-key
# ou en mode non interactif
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Extrait de configuration

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

La documentation actuelle des modèles d'API d'OpenAI liste `gpt-5.4` et `gpt-5.4-pro` pour une utilisation directe de l'API OpenAI. OpenClaw les transmet tous les deux via le chemin Responses `openai/*`.

## Option B : Abonnement OpenAI Code (Codex)

**Idéal pour :** utiliser l'accès par abonnement ChatGPT/Codex au lieu d'une clé API. Le cloud Codex nécessite une connexion ChatGPT, tandis que la CLI Codex prend en charge la connexion ChatGPT ou par clé API.

### Configuration en CLI (OAuth Codex)

```bash
# Exécuter l'OAuth Codex dans l'assistant
openclaw onboard --auth-choice openai-codex

# Ou exécuter l'OAuth directement
openclaw models auth login --provider openai-codex
```

### Extrait de configuration (abonnement Codex)

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

La documentation actuelle de Codex d'OpenAI liste `gpt-5.4` comme modèle Codex actuel. OpenClaw le mappe vers `openai-codex/gpt-5.4` pour une utilisation OAuth ChatGPT/Codex.

### Transport par défaut

OpenClaw utilise `pi-ai` pour le streaming des modèles. Pour `openai/*` et `openai-codex/*`, le transport par défaut est `"auto"` (WebSocket en premier, puis repli sur SSE). Vous pouvez définir `agents.defaults.models.<provider/model>.params.transport` :

-   `"sse"` : forcer SSE
-   `"websocket"` : forcer WebSocket
-   `"auto"` : essayer WebSocket, puis repli sur SSE

Pour `openai/*` (API Responses), OpenClaw active également le préchauffage WebSocket par défaut (`openaiWsWarmup: true`) lorsque le transport WebSocket est utilisé. Documentation OpenAI connexe :

-   [API Realtime avec WebSocket](https://platform.openai.com/docs/guides/realtime-websocket)
-   [Réponses API en streaming (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### Préchauffage WebSocket OpenAI

La documentation OpenAI décrit le préchauffage comme optionnel. OpenClaw l'active par défaut pour `openai/*` afin de réduire la latence du premier tour lors de l'utilisation du transport WebSocket.

### Désactiver le préchauffage

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### Activer le préchauffage explicitement

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### Traitement prioritaire OpenAI

L'API d'OpenAI expose le traitement prioritaire via `service_tier=priority`. Dans OpenClaw, définissez `agents.defaults.models["openai/"].params.serviceTier` pour transmettre ce champ lors des requêtes directes Responses `openai/*`.

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

Les valeurs prises en charge sont `auto`, `default`, `flex` et `priority`.

### Compaction côté serveur OpenAI Responses

Pour les modèles OpenAI Responses directs (`openai/*` utilisant `api: "openai-responses"` avec `baseUrl` sur `api.openai.com`), OpenClaw active désormais automatiquement les indications de payload de compaction côté serveur OpenAI :

-   Force `store: true` (sauf si la compatibilité du modèle définit `supportsStore: false`)
-   Injecte `context_management: [{ type: "compaction", compact_threshold: ... }]`

Par défaut, `compact_threshold` est `70%` du `contextWindow` du modèle (ou `80000` si indisponible).

### Activer la compaction côté serveur explicitement

Utilisez ceci lorsque vous souhaitez forcer l'injection de `context_management` sur les modèles Responses compatibles (par exemple Azure OpenAI Responses) :

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### Activer avec un seuil personnalisé

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### Désactiver la compaction côté serveur

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

`responsesServerCompaction` ne contrôle que l'injection de `context_management`. Les modèles OpenAI Responses directs forcent toujours `store: true` sauf si la compatibilité définit `supportsStore: false`.

## Notes

-   Les références de modèle utilisent toujours `provider/model` (voir [/concepts/models](../concepts/models.md)).
-   Les détails d'authentification et les règles de réutilisation sont dans [/concepts/oauth](../concepts/oauth.md).

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)

---