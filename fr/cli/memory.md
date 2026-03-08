

  Commandes CLI

  
# memory

Gérez l'indexation et la recherche de la mémoire sémantique. Fourni par le plugin de mémoire actif (par défaut : `memory-core` ; définissez `plugins.slots.memory = "none"` pour le désactiver). Liens utiles :

-   Concept Mémoire : [Mémoire](../concepts/memory.md)
-   Plugins : [Plugins](../tools/plugin.md)

## Exemples

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Options

`memory status` et `memory index` :

-   `--agent ` : limiter la portée à un seul agent. Sans cela, ces commandes s'exécutent pour chaque agent configuré ; si aucune liste d'agents n'est configurée, elles reviennent à l'agent par défaut.
-   `--verbose` : émettre des journaux détaillés pendant les sondages et l'indexation.

`memory status` :

-   `--deep` : sonder la disponibilité du vecteur et des embeddings.
-   `--index` : exécuter une réindexation si le stock est sale (implique `--deep`).
-   `--json` : afficher la sortie en JSON.

`memory index` :

-   `--force` : forcer une réindexation complète.

`memory search` :

-   Entrée de requête : passez soit l'argument positionnel `[query]` soit `--query `.
-   Si les deux sont fournis, `--query` l'emporte.
-   Si aucun n'est fourni, la commande se termine avec une erreur.
-   `--agent ` : limiter la portée à un seul agent (par défaut : l'agent par défaut).
-   `--max-results ` : limiter le nombre de résultats retournés.
-   `--min-score ` : filtrer les correspondances avec un score faible.
-   `--json` : afficher les résultats en JSON.

Notes :

-   `memory index --verbose` affiche les détails par phase (fournisseur, modèle, sources, activité par lot).
-   `memory status` inclut tous les chemins supplémentaires configurés via `memorySearch.extraPaths`.
-   Si les champs de clé API distante de mémoire effectivement actifs sont configurés comme SecretRefs, la commande résout ces valeurs à partir de l'instantané de la passerelle active. Si la passerelle est indisponible, la commande échoue rapidement.
-   Note sur la divergence de version de la passerelle : ce chemin de commande nécessite une passerelle qui prend en charge `secrets.resolve` ; les passerelles plus anciennes retournent une erreur de méthode inconnue.

[logs](./logs.md)[message](./message.md)