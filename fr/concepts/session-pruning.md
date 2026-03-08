

  Sessions et mémoire

  
# Élagage de session

L'élagage de session supprime les **anciens résultats d'outils** du contexte en mémoire juste avant chaque appel LLM. Il ne **réécrit pas** l'historique de session sur disque (`*.jsonl`).

## Quand il s'exécute

-   Lorsque `mode: "cache-ttl"` est activé et que le dernier appel Anthropic pour la session est plus ancien que `ttl`.
-   N'affecte que les messages envoyés au modèle pour cette requête.
-   Seulement actif pour les appels API Anthropic (et les modèles Anthropic via OpenRouter).
-   Pour de meilleurs résultats, faites correspondre `ttl` à votre politique `cacheRetention` du modèle (`short` = 5m, `long` = 1h).
-   Après un élagage, la fenêtre TTL est réinitialisée, donc les requêtes suivantes conservent le cache jusqu'à ce que `ttl` expire à nouveau.

## Valeurs par défaut intelligentes (Anthropic)

-   Profils **OAuth ou setup-token** : activent l'élagage `cache-ttl` et définissent le heartbeat à `1h`.
-   Profils **clé API** : activent l'élagage `cache-ttl`, définissent le heartbeat à `30m`, et par défaut `cacheRetention: "short"` sur les modèles Anthropic.
-   Si vous définissez explicitement l'une de ces valeurs, OpenClaw ne les **écrase pas**.

## Ce que cela améliore (coût + comportement du cache)

-   **Pourquoi élaguer :** La mise en cache des prompts Anthropic ne s'applique que dans la fenêtre TTL. Si une session est inactive au-delà du TTL, la prochaine requête remet en cache l'intégralité du prompt sauf si vous l'élaguez d'abord.
-   **Ce qui devient moins cher :** l'élagage réduit la taille de **cacheWrite** pour cette première requête après l'expiration du TTL.
-   **Pourquoi la réinitialisation du TTL est importante :** une fois l'élagage exécuté, la fenêtre de cache est réinitialisée, donc les requêtes suivantes peuvent réutiliser le prompt fraîchement mis en cache au lieu de remettre en cache tout l'historique.
-   **Ce qu'il ne fait pas :** l'élagage n'ajoute pas de tokens ni ne "double" les coûts ; il modifie seulement ce qui est mis en cache lors de cette première requête post-TTL.

## Ce qui peut être élagué

-   Seulement les messages `toolResult`.
-   Les messages utilisateur + assistant ne sont **jamais** modifiés.
-   Les derniers messages assistant `keepLastAssistants` sont protégés ; les résultats d'outils après cette limite ne sont pas élagués.
-   S'il n'y a pas assez de messages assistant pour établir la limite, l'élagage est ignoré.
-   Les résultats d'outils contenant des **blocs d'image** sont ignorés (jamais nettoyés/supprimés).

## Estimation de la fenêtre de contexte

L'élagage utilise une fenêtre de contexte estimée (caractères ≈ tokens × 4). La fenêtre de base est résolue dans cet ordre :

1.  Surcharge `models.providers.*.models[].contextWindow`.
2.  Définition du modèle `contextWindow` (depuis le registre des modèles).
3.  Valeur par défaut `200000` tokens.

Si `agents.defaults.contextTokens` est défini, il est traité comme une limite (minimum) sur la fenêtre résolue.

## Mode

### cache-ttl

-   L'élagage ne s'exécute que si le dernier appel Anthropic est plus ancien que `ttl` (par défaut `5m`).
-   Quand il s'exécute : même comportement de soft-trim + hard-clear qu'auparavant.

## Élagage doux vs dur

-   **Soft-trim (élagage doux) :** seulement pour les résultats d'outils trop volumineux.
    -   Garde le début + la fin, insère `...`, et ajoute une note avec la taille originale.
    -   Ignore les résultats avec des blocs d'image.
-   **Hard-clear (nettoyage dur) :** remplace l'intégralité du résultat d'outil par `hardClear.placeholder`.

## Sélection des outils

-   `tools.allow` / `tools.deny` supportent les caractères génériques `*`.
-   Deny l'emporte.
-   La correspondance est insensible à la casse.
-   Liste allow vide => tous les outils autorisés.

## Interaction avec les autres limites

-   Les outils intégrés tronquent déjà leur propre sortie ; l'élagage de session est une couche supplémentaire qui empêche les conversations longues d'accumuler trop de sorties d'outils dans le contexte du modèle.
-   La compaction est séparée : la compaction résume et persiste, l'élagage est transitoire par requête. Voir [/concepts/compaction](./compaction.md).

## Valeurs par défaut (lorsqu'activé)

-   `ttl`: `"5m"`
-   `keepLastAssistants`: `3`
-   `softTrimRatio`: `0.3`
-   `hardClearRatio`: `0.5`
-   `minPrunableToolChars`: `50000`
-   `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
-   `hardClear`: `{ enabled: true, placeholder: "[Contenu de l'ancien résultat d'outil effacé]" }`

## Exemples

Par défaut (désactivé) :

```json
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Activer l'élagage basé sur TTL :

```json
{
  agents: { defaults: { contextPruning: { mode: "cache-ttl", ttl: "5m" } } },
}
```

Restreindre l'élagage à des outils spécifiques :

```json
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl",
        tools: { allow: ["exec", "read"], deny: ["*image*"] },
      },
    },
  },
}
```

Voir la référence de configuration : [Configuration de la passerelle](../gateway/configuration.md)

[Gestion de session](./session.md)[Outils de session](./session-tool.md)