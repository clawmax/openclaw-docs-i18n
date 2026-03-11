

  Fournisseurs

  
# Hugging Face (Inférence)

Les [Fournisseurs d'Inférence Hugging Face](https://huggingface.co/docs/inference-providers) offrent des complétions de chat compatibles OpenAI via une unique API routeur. Vous obtenez l'accès à de nombreux modèles (DeepSeek, Llama, et plus) avec un seul jeton. OpenClaw utilise le **point de terminaison compatible OpenAI** (chat completions uniquement) ; pour la génération d'images, les embeddings ou la parole, utilisez directement les [clients d'inférence HF](https://huggingface.co/docs/api-inference/quicktour).

-   Fournisseur : `huggingface`
-   Authentification : `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN` (jeton fin avec la permission **Make calls to Inference Providers**)
-   API : Compatible OpenAI (`https://router.huggingface.co/v1`)
-   Facturation : Jeton HF unique ; la [tarification](https://huggingface.co/docs/inference-providers/pricing) suit les taux des fournisseurs avec un niveau gratuit.

## Démarrage rapide

1.  Créez un jeton fin dans [Hugging Face → Paramètres → Jetons](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) avec la permission **Make calls to Inference Providers**.
2.  Lancez l'intégration et choisissez **Hugging Face** dans la liste déroulante des fournisseurs, puis entrez votre clé API lorsque demandé :

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  Dans la liste déroulante **Modèle Hugging Face par défaut**, choisissez le modèle souhaité (la liste est chargée depuis l'API d'inférence lorsque vous avez un jeton valide ; sinon une liste intégrée est affichée). Votre choix est enregistré comme modèle par défaut.
4.  Vous pouvez également définir ou changer le modèle par défaut plus tard dans la configuration :

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Cela définira `huggingface/deepseek-ai/DeepSeek-R1` comme modèle par défaut.

## Note sur l'environnement

Si la Gateway s'exécute en tant que démon (launchd/systemd), assurez-vous que `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN` est disponible pour ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

## Découverte des modèles et liste déroulante d'intégration

OpenClaw découvre les modèles en appelant directement le **point de terminaison d'inférence** :

```bash
GET https://router.huggingface.co/v1/models
```

(Optionnel : envoyez `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` ou `$HF_TOKEN` pour la liste complète ; certains endpoints retournent un sous-ensemble sans authentification.) La réponse est au format OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`. Lorsque vous configurez une clé API Hugging Face (via l'intégration, `HUGGINGFACE_HUB_TOKEN`, ou `HF_TOKEN`), OpenClaw utilise cette requête GET pour découvrir les modèles de chat-complétion disponibles. Pendant **l'intégration interactive**, après avoir entré votre jeton, vous voyez une liste déroulante **Modèle Hugging Face par défaut** peuplée à partir de cette liste (ou du catalogue intégré si la requête échoue). Au moment de l'exécution (par exemple au démarrage de la Gateway), lorsqu'une clé est présente, OpenClaw appelle à nouveau **GET** `https://router.huggingface.co/v1/models` pour rafraîchir le catalogue. La liste est fusionnée avec un catalogue intégré (pour les métadonnées comme la fenêtre de contexte et le coût). Si la requête échoue ou si aucune clé n'est définie, seul le catalogue intégré est utilisé.

## Noms de modèles et options modifiables

-   **Nom depuis l'API :** Le nom d'affichage du modèle est **hydraté depuis GET /v1/models** lorsque l'API retourne `name`, `title`, ou `display_name` ; sinon il est dérivé de l'id du modèle (par ex. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
-   **Surcharger le nom d'affichage :** Vous pouvez définir un libellé personnalisé par modèle dans la configuration pour qu'il apparaisse comme vous le souhaitez dans la CLI et l'interface :

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (rapide)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (économique)" },
      },
    },
  },
}
```

-   **Sélection du fournisseur / politique :** Ajoutez un suffixe à l'**id du modèle** pour choisir comment le routeur sélectionne le backend :
    
    -   **`:fastest`** — débit le plus élevé (le routeur choisit ; le choix du fournisseur est **verrouillé** — pas de sélecteur de backend interactif).
    -   **`:cheapest`** — coût le plus bas par jeton de sortie (le routeur choisit ; le choix du fournisseur est **verrouillé**).
    -   **`:provider`** — forcer un backend spécifique (par ex. `:sambanova`, `:together`).
    
    Lorsque vous sélectionnez **:cheapest** ou **:fastest** (par ex. dans la liste déroulante des modèles d'intégration), le fournisseur est verrouillé : le routeur décide par coût ou vitesse et aucune étape optionnelle "préférer un backend spécifique" n'est affichée. Vous pouvez ajouter ces options comme entrées séparées dans `models.providers.huggingface.models` ou définir `model.primary` avec le suffixe. Vous pouvez également définir votre ordre par défaut dans les [paramètres des Fournisseurs d'Inférence](https://hf.co/settings/inference-providers) (pas de suffixe = utiliser cet ordre).
-   **Fusion de configuration :** Les entrées existantes dans `models.providers.huggingface.models` (par ex. dans `models.json`) sont conservées lors de la fusion de configuration. Ainsi, tout `name`, `alias` personnalisé ou options de modèle que vous y définissez sont préservés.

## IDs de modèles et exemples de configuration

Les références de modèle utilisent la forme `huggingface//` (IDs de style Hub). La liste ci-dessous provient de **GET** `https://router.huggingface.co/v1/models` ; votre catalogue peut en inclure davantage. **Exemples d'IDs (depuis le point de terminaison d'inférence) :**

| Modèle | Référence (préfixer par `huggingface/`) |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

Vous pouvez ajouter `:fastest`, `:cheapest`, ou `:provider` (par ex. `:together`, `:sambanova`) à l'id du modèle. Définissez votre ordre par défaut dans les [paramètres des Fournisseurs d'Inférence](https://hf.co/settings/inference-providers) ; consultez [Fournisseurs d'Inférence](https://huggingface.co/docs/inference-providers) et **GET** `https://router.huggingface.co/v1/models` pour la liste complète.

### Exemples de configuration complets

**DeepSeek R1 principal avec Qwen en secours :**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen par défaut, avec variantes :cheapest et :fastest :**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (le plus économique)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (le plus rapide)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS avec alias :**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Forcer un backend spécifique avec :provider :**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Multiples modèles Qwen et DeepSeek avec suffixes de politique :**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (économique)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (rapide)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)