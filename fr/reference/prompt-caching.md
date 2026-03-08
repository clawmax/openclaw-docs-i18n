

  Référence technique

  
# Mise en cache des prompts

La mise en cache des prompts signifie que le fournisseur de modèle peut réutiliser les préfixes de prompts inchangés (généralement les instructions système/développeur et d'autres contextes stables) d'un tour à l'autre au lieu de les retraiter à chaque fois. La première requête correspondante écrit les tokens de cache (`cacheWrite`), et les requêtes correspondantes ultérieures peuvent les relire (`cacheRead`). Pourquoi c'est important : coût en tokens réduit, réponses plus rapides et performances plus prévisibles pour les sessions de longue durée. Sans mise en cache, les prompts répétés paient le coût complet du prompt à chaque tour même lorsque la majeure partie de l'entrée n'a pas changé. Cette page couvre tous les paramètres liés au cache qui affectent la réutilisation des prompts et le coût en tokens. Pour les détails de tarification Anthropic, voir : [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## Paramètres principaux

### cacheRetention (modèle et par agent)

Définissez la rétention du cache dans les paramètres du modèle :

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

Surcharge par agent :

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

Ordre de fusion de la configuration :

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (correspondant à l'id de l'agent ; écrase par clé)

### Ancien paramètre cacheControlTtl

Les valeurs héritées sont toujours acceptées et mappées :

-   `5m` -> `short`
-   `1h` -> `long`

Préférez `cacheRetention` pour les nouvelles configurations.

### contextPruning.mode: "cache-ttl"

Élague l'ancien contexte des résultats d'outils après les fenêtres de durée de vie (TTL) du cache afin que les requêtes après une période d'inactivité ne remettent pas en cache un historique trop volumineux.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

Voir [Élagage de session](../concepts/session-pruning.md) pour le comportement complet.

### Heartbeat pour maintenir au chaud

Le heartbeat peut maintenir les fenêtres de cache au chaud et réduire les écritures répétées dans le cache après des périodes d'inactivité.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

Le heartbeat par agent est pris en charge dans `agents.list[].heartbeat`.

## Comportement des fournisseurs

### Anthropic (API directe)

-   `cacheRetention` est pris en charge.
-   Avec les profils d'authentification par clé API Anthropic, OpenClaw initialise `cacheRetention: "short"` pour les références de modèles Anthropic lorsqu'il n'est pas défini.

### Amazon Bedrock

-   Les références de modèles Anthropic Claude (`amazon-bedrock/*anthropic.claude*`) prennent en charge le passage explicite de `cacheRetention`.
-   Les modèles Bedrock non-Anthropic sont forcés à `cacheRetention: "none"` à l'exécution.

### Modèles Anthropic via OpenRouter

Pour les références de modèles `openrouter/anthropic/*`, OpenClaw injecte `cache_control` Anthropic sur les blocs de prompts système/développeur pour améliorer la réutilisation du cache de prompts.

### Autres fournisseurs

Si le fournisseur ne prend pas en charge ce mode de cache, `cacheRetention` n'a aucun effet.

## Modèles d'optimisation

### Trafic mixte (par défaut recommandé)

Maintenez une base de référence de longue durée sur votre agent principal, désactivez la mise en cache sur les agents de notification à forte activité :

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### Base de référence axée sur les coûts

-   Définissez la base de référence `cacheRetention: "short"`.
-   Activez `contextPruning.mode: "cache-ttl"`.
-   Gardez le heartbeat en dessous de votre TTL uniquement pour les agents qui bénéficient de caches chauds.

## Diagnostics du cache

OpenClaw expose des diagnostics dédiés de trace de cache pour les exécutions d'agents embarqués.

### Configuration diagnostics.cacheTrace

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # optionnel
    includeMessages: false # par défaut true
    includePrompt: false # par défaut true
    includeSystem: false # par défaut true
```

Valeurs par défaut :

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### Interrupteurs d'environnement (débogage ponctuel)

-   `OPENCLAW_CACHE_TRACE=1` active la trace du cache.
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` écrase le chemin de sortie.
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` active/désactive la capture complète des charges utiles des messages.
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` active/désactive la capture du texte du prompt.
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` active/désactive la capture du prompt système.

### Ce qu'il faut inspecter

-   Les événements de trace de cache sont au format JSONL et incluent des instantanés mis en scène comme `session:loaded`, `prompt:before`, `stream:context`, et `session:after`.
-   L'impact des tokens de cache par tour est visible dans les surfaces d'utilisation normales via `cacheRead` et `cacheWrite` (par exemple `/usage full` et les résumés d'utilisation de session).

## Dépannage rapide

-   `cacheWrite` élevé sur la plupart des tours : vérifiez les entrées de prompts système volatiles et assurez-vous que le modèle/le fournisseur prend en charge vos paramètres de cache.
-   Aucun effet de `cacheRetention` : confirmez que la clé du modèle correspond à `agents.defaults.models["provider/model"]`.
-   Requêtes Bedrock Nova/Mistral avec paramètres de cache : forçage attendu à `none` à l'exécution.

Documentation associée :

-   [Anthropic](../providers/anthropic.md)
-   [Utilisation et coûts des tokens](./token-use.md)
-   [Élagage de session](../concepts/session-pruning.md)
-   [Référence de configuration de la passerelle](../gateway/configuration-reference.md)

[Surface d'identification SecretRef](./secretref-credential-surface.md)[Utilisation de l'API et coûts](./api-usage-costs.md)