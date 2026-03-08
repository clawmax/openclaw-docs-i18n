

  Commandes CLI

  
# sessions

Lister les sessions de conversation stockées.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

Sélection de la portée :

-   par défaut : le magasin d'agent par défaut configuré
-   `--agent ` : un magasin d'agent configuré
-   `--all-agents` : agréger tous les magasins d'agents configurés
-   `--store ` : chemin explicite du magasin (ne peut pas être combiné avec `--agent` ou `--all-agents`)

Exemples JSON : `openclaw sessions --all-agents --json` :

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## Maintenance de nettoyage

Exécuter la maintenance maintenant (au lieu d'attendre le prochain cycle d'écriture) :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` utilise les paramètres `session.maintenance` de la configuration :

-   Note sur la portée : `openclaw sessions cleanup` maintient uniquement les magasins/transcriptions de session. Il ne nettoie pas les journaux d'exécution cron (`cron/runs/.jsonl`), qui sont gérés par `cron.runLog.maxBytes` et `cron.runLog.keepLines` dans la [Configuration Cron](../automation/cron-jobs.md#configuration) et expliqués dans la [Maintenance Cron](../automation/cron-jobs.md#maintenance).
-   `--dry-run` : prévisualiser combien d'entrées seraient élaguées/limitées sans écrire.
    -   En mode texte, le dry-run affiche un tableau d'actions par session (`Action`, `Clé`, `Âge`, `Modèle`, `Drapeaux`) pour voir ce qui serait conservé vs supprimé.
-   `--enforce` : appliquer la maintenance même lorsque `session.maintenance.mode` est `warn`.
-   `--active-key ` : protéger une clé active spécifique de l'éviction par budget disque.
-   `--agent ` : exécuter le nettoyage pour un magasin d'agent configuré.
-   `--all-agents` : exécuter le nettoyage pour tous les magasins d'agents configurés.
-   `--store ` : exécuter sur un fichier `sessions.json` spécifique.
-   `--json` : imprimer un résumé JSON. Avec `--all-agents`, la sortie inclut un résumé par magasin.

`openclaw sessions cleanup --all-agents --dry-run --json` :

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

Liens connexes :

-   Configuration des sessions : [Référence de configuration](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)