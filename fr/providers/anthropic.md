title: "Configuration du fournisseur Anthropic pour OpenClaw : Clé API et jeton Claude"
description: "Apprenez à authentifier OpenClaw avec Anthropic en utilisant une clé API ou un jeton setup-token. Configurez la mise en cache des prompts, les fenêtres de contexte de 1M et résolvez les problèmes courants."
keywords: ["anthropic", "api claude", "configuration openclaw", "mise en cache des prompts", "setup-token", "fenêtre de contexte 1m", "claude 4.6", "bedrock claude"]
---

  Fournisseurs

  
# Anthropic

Anthropic développe la famille de modèles **Claude** et y donne accès via une API. Dans OpenClaw, vous pouvez vous authentifier avec une clé API ou un **setup-token**.

## Option A : Clé API Anthropic

**Idéal pour :** un accès API standard et une facturation à l'usage. Créez votre clé API dans la Console Anthropic.

### Configuration en CLI

```bash
openclaw onboard
# choisir : Anthropic API key

# ou en mode non interactif
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### Extrait de configuration

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Valeurs par défaut pour la réflexion (Claude 4.6)

-   Les modèles Anthropic Claude 4.6 utilisent par défaut la réflexion `adaptive` dans OpenClaw lorsqu'aucun niveau de réflexion explicite n'est défini.
-   Vous pouvez la remplacer par message (`/think:`) ou dans les paramètres du modèle : `agents.defaults.models["anthropic/"].params.thinking`.
-   Documentation Anthropic associée :
    -   [Réflexion adaptative](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [Réflexion étendue](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## Mise en cache des prompts (API Anthropic)

OpenClaw prend en charge la fonctionnalité de mise en cache des prompts d'Anthropic. C'est **uniquement pour l'API** ; l'authentification par abonnement ne respecte pas les paramètres de cache.

### Configuration

Utilisez le paramètre `cacheRetention` dans votre configuration de modèle :

| Valeur | Durée du cache | Description |
| --- | --- | --- |
| `none` | Pas de cache | Désactive la mise en cache des prompts |
| `short` | 5 minutes | Valeur par défaut pour l'authentification par clé API |
| `long` | 1 heure | Cache étendu (nécessite un drapeau bêta) |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### Valeurs par défaut

Lorsque vous utilisez l'authentification par clé API Anthropic, OpenClaw applique automatiquement `cacheRetention: "short"` (cache de 5 minutes) pour tous les modèles Anthropic. Vous pouvez remplacer ce comportement en définissant explicitement `cacheRetention` dans votre configuration.

### Surcharges de cacheRetention par agent

Utilisez les paramètres au niveau du modèle comme base, puis remplacez-les pour des agents spécifiques via `agents.list[].params`.

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // base pour la plupart des agents
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // remplacement pour cet agent uniquement
    ],
  },
}
```

Ordre de fusion de la configuration pour les paramètres liés au cache :

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (correspondant à l'`id`, remplace par clé)

Cela permet à un agent de conserver un cache de longue durée tandis qu'un autre agent utilisant le même modèle désactive la mise en cache pour éviter les coûts d'écriture sur un trafic irrégulier/à faible réutilisation.

### Notes sur Bedrock Claude

-   Les modèles Anthropic Claude sur Bedrock (`amazon-bedrock/*anthropic.claude*`) acceptent le passage du paramètre `cacheRetention` lorsqu'il est configuré.
-   Les modèles Bedrock non-Anthropic sont forcés à `cacheRetention: "none"` à l'exécution.
-   Les valeurs par défaut intelligentes de la clé API Anthropic définissent également `cacheRetention: "short"` pour les références de modèles Claude-on-Bedrock lorsqu'aucune valeur explicite n'est définie.

### Paramètre hérité

L'ancien paramètre `cacheControlTtl` est toujours pris en charge pour la rétrocompatibilité :

-   `"5m"` correspond à `short`
-   `"1h"` correspond à `long`

Nous recommandons de migrer vers le nouveau paramètre `cacheRetention`. OpenClaw inclut le drapeau bêta `extended-cache-ttl-2025-04-11` pour les requêtes API Anthropic ; conservez-le si vous remplacez les en-têtes du fournisseur (voir [/gateway/configuration](../gateway/configuration.md)).

## Fenêtre de contexte de 1M (bêta Anthropic)

La fenêtre de contexte de 1M d'Anthropic est soumise à un accès bêta. Dans OpenClaw, activez-la par modèle avec `params.context1m: true` pour les modèles Opus/Sonnet pris en charge.

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw mappe ceci à `anthropic-beta: context-1m-2025-08-07` sur les requêtes Anthropic. Cela ne s'active que lorsque `params.context1m` est explicitement défini sur `true` pour ce modèle. Prérequis : Anthropic doit autoriser l'utilisation du long contexte avec ces identifiants (généralement la facturation par clé API, ou un compte d'abonnement avec l'option Extra Usage activée). Sinon, Anthropic renvoie : `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. Note : Anthropic rejette actuellement les requêtes bêta `context-1m-*` lors de l'utilisation de jetons OAuth/abonnement (`sk-ant-oat-*`). OpenClaw ignore automatiquement l'en-tête bêta context1m pour l'authentification OAuth et conserve les bêtas OAuth requis.

## Option B : Setup-token Claude

**Idéal pour :** utiliser votre abonnement Claude.

### Où obtenir un setup-token

Les setup-tokens sont créés par la **CLI Claude Code**, et non par la Console Anthropic. Vous pouvez l'exécuter sur **n'importe quelle machine** :

```bash
claude setup-token
```

Collez le jeton dans OpenClaw (assistant : **Anthropic token (paste setup-token)**), ou exécutez-le sur l'hôte de la passerelle :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le jeton sur une autre machine, collez-le :

```bash
openclaw models auth paste-token --provider anthropic
```

### Configuration CLI (setup-token)

```bash
# Collez un setup-token pendant l'intégration
openclaw onboard --auth-choice setup-token
```

### Extrait de configuration (setup-token)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Notes

-   Générez le setup-token avec `claude setup-token` et collez-le, ou exécutez `openclaw models auth setup-token` sur l'hôte de la passerelle.
-   Si vous voyez "OAuth token refresh failed …" avec un abonnement Claude, ré-authentifiez-vous avec un setup-token. Voir [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription).
-   Les détails d'authentification et les règles de réutilisation sont dans [/concepts/oauth](../concepts/oauth.md).

## Dépannage

**Erreurs 401 / jeton soudainement invalide**

-   L'authentification par abonnement Claude peut expirer ou être révoquée. Ré-exécutez `claude setup-token` et collez-le sur **l'hôte de la passerelle**.
-   Si la connexion CLI Claude se trouve sur une autre machine, utilisez `openclaw models auth paste-token --provider anthropic` sur l'hôte de la passerelle.

**Aucune clé API trouvée pour le fournisseur “anthropic”**

-   L'authentification est **par agent**. Les nouveaux agents n'héritent pas des clés de l'agent principal.
-   Relancez l'intégration pour cet agent, ou collez un setup-token / une clé API sur l'hôte de la passerelle, puis vérifiez avec `openclaw models status`.

**Aucun identifiant trouvé pour le profil `anthropic:default`**

-   Exécutez `openclaw models status` pour voir quel profil d'authentification est actif.
-   Relancez l'intégration, ou collez un setup-token / une clé API pour ce profil.

**Aucun profil d'authentification disponible (tous en refroidissement/indisponibles)**

-   Vérifiez `openclaw models status --json` pour `auth.unusableProfiles`.
-   Ajoutez un autre profil Anthropic ou attendez la fin de la période de refroidissement.

Plus d'informations : [/gateway/troubleshooting](../gateway/troubleshooting.md) et [/help/faq](../help/faq.md).

[Basculement de modèle](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)