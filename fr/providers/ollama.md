

  Fournisseurs

  
# Ollama

Ollama est un runtime LLM local qui facilite l'exécution de modèles open source sur votre machine. OpenClaw s'intègre avec l'API native d'Ollama (`/api/chat`), prenant en charge le streaming et l'appel d'outils, et peut **découvrir automatiquement les modèles compatibles avec les outils** lorsque vous optez pour cette fonctionnalité avec `OLLAMA_API_KEY` (ou un profil d'authentification) et ne définissez pas d'entrée explicite `models.providers.ollama`.

> **⚠️** **Utilisateurs d'Ollama distant** : N'utilisez pas l'URL compatible OpenAI `/v1` (`http://host:11434/v1`) avec OpenClaw. Cela casse l'appel d'outils et les modèles peuvent produire du JSON brut d'outils sous forme de texte brut. Utilisez plutôt l'URL de l'API native d'Ollama : `baseUrl: "http://host:11434"` (sans `/v1`).

## Démarrage rapide

1.  Installez Ollama : [https://ollama.ai](https://ollama.ai)
2.  Téléchargez un modèle :

```bash
ollama pull gpt-oss:20b
# ou
ollama pull llama3.3
# ou
ollama pull qwen2.5-coder:32b
# ou
ollama pull deepseek-r1:32b
```

3.  Activez Ollama pour OpenClaw (n'importe quelle valeur fonctionne ; Ollama ne nécessite pas de clé réelle) :

```bash
# Définir la variable d'environnement
export OLLAMA_API_KEY="ollama-local"

# Ou configurez dans votre fichier de configuration
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  Utilisez les modèles Ollama :

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Découverte de modèles (fournisseur implicite)

Lorsque vous définissez `OLLAMA_API_KEY` (ou un profil d'authentification) et **ne définissez pas** `models.providers.ollama`, OpenClaw découvre les modèles depuis l'instance Ollama locale à `http://127.0.0.1:11434` :

-   Interroge `/api/tags` et `/api/show`
-   Ne conserve que les modèles qui signalent la capacité `tools`
-   Marque `reasoning` lorsque le modèle signale `thinking`
-   Lit `contextWindow` depuis `model_info[".context_length"]` quand disponible
-   Définit `maxTokens` à 10 fois la taille du contexte
-   Définit tous les coûts à `0`

Cela évite les entrées manuelles de modèles tout en maintenant le catalogue aligné sur les capacités d'Ollama. Pour voir quels modèles sont disponibles :

```bash
ollama list
openclaw models list
```

Pour ajouter un nouveau modèle, téléchargez-le simplement avec Ollama :

```bash
ollama pull mistral
```

Le nouveau modèle sera automatiquement découvert et disponible à l'utilisation. Si vous définissez explicitement `models.providers.ollama`, la découverte automatique est désactivée et vous devez définir les modèles manuellement (voir ci-dessous).

## Configuration

### Configuration de base (découverte implicite)

La manière la plus simple d'activer Ollama est via une variable d'environnement :

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Configuration explicite (modèles manuels)

Utilisez une configuration explicite lorsque :

-   Ollama s'exécute sur un autre hôte/port.
-   Vous voulez forcer des tailles de contexte ou des listes de modèles spécifiques.
-   Vous voulez inclure des modèles qui ne signalent pas de support des outils.

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Si `OLLAMA_API_KEY` est défini, vous pouvez omettre `apiKey` dans l'entrée du fournisseur et OpenClaw le remplira pour les vérifications de disponibilité.

### URL de base personnalisée (configuration explicite)

Si Ollama s'exécute sur un hôte ou un port différent (la configuration explicite désactive la découverte automatique, donc définissez les modèles manuellement) :

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // Pas de /v1 - utilisez l'URL de l'API native d'Ollama
        api: "ollama", // Défini explicitement pour garantir le comportement natif d'appel d'outils
      },
    },
  },
}
```

> **⚠️** N'ajoutez pas `/v1` à l'URL. Le chemin `/v1` utilise le mode compatible OpenAI, où l'appel d'outils n'est pas fiable. Utilisez l'URL de base d'Ollama sans suffixe de chemin.

### Sélection de modèle

Une fois configuré, tous vos modèles Ollama sont disponibles :

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Avancé

### Modèles de raisonnement

OpenClaw marque les modèles comme capables de raisonnement lorsque Ollama signale `thinking` dans `/api/show` :

```bash
ollama pull deepseek-r1:32b
```

### Coûts des modèles

Ollama est gratuit et s'exécute localement, donc tous les coûts des modèles sont définis à 0 $.

### Configuration du streaming

L'intégration Ollama d'OpenClaw utilise **l'API native d'Ollama** (`/api/chat`) par défaut, qui prend entièrement en charge le streaming et l'appel d'outils simultanément. Aucune configuration spéciale n'est nécessaire.

#### Mode compatible OpenAI hérité

> **⚠️** **L'appel d'outils n'est pas fiable en mode compatible OpenAI.** Utilisez ce mode uniquement si vous avez besoin du format OpenAI pour un proxy et ne dépendez pas du comportement natif d'appel d'outils.

 Si vous devez utiliser le point de terminaison compatible OpenAI à la place (par exemple, derrière un proxy qui ne prend en charge que le format OpenAI), définissez explicitement `api: "openai-completions"` :

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // par défaut : true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Ce mode peut ne pas prendre en charge le streaming et l'appel d'outils simultanément. Vous devrez peut-être désactiver le streaming avec `params: { streaming: false }` dans la configuration du modèle. Lorsque `api: "openai-completions"` est utilisé avec Ollama, OpenClaw injecte `options.num_ctx` par défaut pour qu'Ollama ne revienne pas silencieusement à une fenêtre de contexte de 4096. Si votre proxy/amont rejette les champs `options` inconnus, désactivez ce comportement :

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### Fenêtres de contexte

Pour les modèles découverts automatiquement, OpenClaw utilise la fenêtre de contexte signalée par Ollama quand elle est disponible, sinon elle utilise par défaut `8192`. Vous pouvez remplacer `contextWindow` et `maxTokens` dans la configuration explicite du fournisseur.

## Dépannage

### Ollama non détecté

Assurez-vous qu'Ollama est en cours d'exécution, que vous avez défini `OLLAMA_API_KEY` (ou un profil d'authentification), et que vous **n'avez pas** défini d'entrée explicite `models.providers.ollama` :

```bash
ollama serve
```

Et que l'API est accessible :

```bash
curl http://localhost:11434/api/tags
```

### Aucun modèle disponible

OpenClaw ne découvre automatiquement que les modèles qui signalent un support des outils. Si votre modèle n'est pas listé, soit :

-   Téléchargez un modèle compatible avec les outils, soit
-   Définissez le modèle explicitement dans `models.providers.ollama`.

Pour ajouter des modèles :

```bash
ollama list  # Voir ce qui est installé
ollama pull gpt-oss:20b  # Téléchargez un modèle compatible avec les outils
ollama pull llama3.3     # Ou un autre modèle
```

### Connexion refusée

Vérifiez qu'Ollama s'exécute sur le bon port :

```bash
# Vérifiez si Ollama est en cours d'exécution
ps aux | grep ollama

# Ou redémarrez Ollama
ollama serve
```

## Voir aussi

-   [Fournisseurs de modèles](../concepts/model-providers.md) - Aperçu de tous les fournisseurs
-   [Sélection de modèles](../concepts/models.md) - Comment choisir des modèles
-   [Configuration](../gateway/configuration.md) - Référence complète de configuration

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)

---