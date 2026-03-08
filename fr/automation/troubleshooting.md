

  Automatisation

  
# Dépannage de l'automatisation

Utilisez cette page pour les problèmes de planification et de livraison (`cron` + `heartbeat`).

## Échelle de commandes

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis exécutez les vérifications d'automatisation :

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron ne se déclenche pas

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Une bonne sortie ressemble à :

-   `cron status` indique qu'il est activé et une future `nextWakeAtMs`.
-   Le job est activé et a un planning/fuseau horaire valide.
-   `cron runs` montre `ok` ou une raison explicite de saut.

Signatures courantes :

-   `cron: scheduler disabled; jobs will not run automatically` → cron désactivé dans la config/env.
-   `cron: timer tick failed` → le tick du planificateur a planté ; inspectez le contexte de la pile/log environnante.
-   `reason: not-due` dans la sortie d'exécution → exécution manuelle appelée sans `--force` et job pas encore dû.

## Cron déclenché mais pas de livraison

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Une bonne sortie ressemble à :

-   Le statut de l'exécution est `ok`.
-   Le mode/cible de livraison sont définis pour les jobs isolés.
-   La sonde du canal indique que le canal cible est connecté.

Signatures courantes :

-   L'exécution a réussi mais le mode de livraison est `none` → aucun message externe n'est attendu.
-   Cible de livraison manquante/invalide (`channel`/`to`) → l'exécution peut réussir en interne mais sauter l'envoi sortant.
-   Erreurs d'authentification du canal (`unauthorized`, `missing_scope`, `Forbidden`) → la livraison est bloquée par les identifiants/autorisations du canal.

## Heartbeat supprimé ou ignoré

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Une bonne sortie ressemble à :

-   Heartbeat activé avec un intervalle non nul.
-   Le dernier résultat du heartbeat est `ran` (ou la raison du saut est comprise).

Signatures courantes :

-   `heartbeat skipped` avec `reason=quiet-hours` → en dehors des `activeHours`.
-   `requests-in-flight` → voie principale occupée ; heartbeat différé.
-   `empty-heartbeat-file` → le heartbeat d'intervalle est ignoré car `HEARTBEAT.md` n'a pas de contenu actionnable et aucun événement cron étiqueté n'est en file d'attente.
-   `alerts-disabled` → les paramètres de visibilité suppriment les messages de heartbeat sortants.

## Pièges des fuseaux horaires et des heures actives

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

Règles rapides :

-   `Config path not found: agents.defaults.userTimezone` signifie que la clé n'est pas définie ; le heartbeat utilise par défaut le fuseau horaire de l'hôte (ou `activeHours.timezone` si défini).
-   Cron sans `--tz` utilise le fuseau horaire de l'hôte de la passerelle.
-   Les `activeHours` du heartbeat utilisent la résolution de fuseau horaire configurée (`user`, `local`, ou un tz IANA explicite).
-   Les horodatages ISO sans fuseau horaire sont traités comme UTC pour les plannings cron `at`.

Signatures courantes :

-   Les jobs s'exécutent à la mauvaise heure réelle après des changements de fuseau horaire de l'hôte.
-   Le heartbeat est toujours ignoré pendant votre journée car `activeHours.timezone` est incorrect.

Liens connexes :

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron vs Heartbeat](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)