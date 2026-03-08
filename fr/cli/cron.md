title: "Gérer les tâches Cron pour le planificateur OpenClaw Gateway"
description: "Apprenez à ajouter, modifier et gérer les tâches cron pour l'interface CLI OpenClaw. Configurez la livraison, la rétention et le contexte léger pour les tâches automatisées."
keywords: ["openclaw cron", "tâches cron", "planificateur gateway", "automatisation cli", "planification de tâches", "automatisation de tâches", "gestion cron", "tâches isolées"]
---

  Commandes CLI

  
# cron

Gérez les tâches cron pour le planificateur Gateway. Liens utiles :

-   Tâches cron : [Tâches cron](../automation/cron-jobs.md)

Astuce : exécutez `openclaw cron --help` pour voir la surface complète des commandes. Note : les tâches `cron add` isolées utilisent par défaut la livraison `--announce`. Utilisez `--no-deliver` pour garder la sortie interne. `--deliver` reste un alias obsolète pour `--announce`. Note : les tâches ponctuelles (`--at`) sont supprimées après succès par défaut. Utilisez `--keep-after-run` pour les conserver. Note : les tâches récurrentes utilisent désormais un backoff exponentiel après des erreurs consécutives (30s → 1m → 5m → 15m → 60m), puis reviennent au planning normal après la prochaine exécution réussie. Note : la rétention/élagage est contrôlé dans la configuration :

-   `cron.sessionRetention` (par défaut `24h`) élimine les sessions d'exécution isolées terminées.
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` éliminent `~/.openclaw/cron/runs/.jsonl`.

## Modifications courantes

Mettez à jour les paramètres de livraison sans changer le message :

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

Désactivez la livraison pour une tâche isolée :

```bash
openclaw cron edit <job-id> --no-deliver
```

Activez le contexte d'amorçage léger pour une tâche isolée :

```bash
openclaw cron edit <job-id> --light-context
```

Annoncez sur un canal spécifique :

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

Créez une tâche isolée avec un contexte d'amorçage léger :

```bash
openclaw cron add \
  --name "Briefing matinal léger" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "Résumez les mises à jour nocturnes." \
  --light-context \
  --no-deliver
```

`--light-context` s'applique uniquement aux tâches isolées de type agent-turn. Pour les exécutions cron, le mode léger garde le contexte d'amorçage vide au lieu d'injecter l'ensemble complet d'amorçage de l'espace de travail.

[configurer](./configure.md)[démon](./daemon.md)

---