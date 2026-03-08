

  Automatisation

  
# Cron vs Heartbeat

Les heartbeats et les tâches cron vous permettent tous deux d'exécuter des tâches selon un planning. Ce guide vous aide à choisir le bon mécanisme pour votre cas d'utilisation.

## Guide de décision rapide

| Cas d'utilisation | Recommandé | Pourquoi |
| --- | --- | --- |
| Vérifier la boîte de réception toutes les 30 min | Heartbeat | Regroupe avec d'autres vérifications, conscient du contexte |
| Envoyer un rapport quotidien à 9h pile | Cron (isolé) | Timing exact nécessaire |
| Surveiller le calendrier pour les événements à venir | Heartbeat | Adapté naturellement pour une prise de conscience périodique |
| Exécuter une analyse approfondie hebdomadaire | Cron (isolé) | Tâche autonome, peut utiliser un modèle différent |
| Me rappeler dans 20 minutes | Cron (principal, `--at`) | Tâche unique avec un timing précis |
| Vérification de l'état de santé du projet en arrière-plan | Heartbeat | Profite du cycle existant |

## Heartbeat : Prise de conscience périodique

Les heartbeats s'exécutent dans la **session principale** à un intervalle régulier (par défaut : 30 min). Ils sont conçus pour que l'agent vérifie les choses et remonte à la surface tout ce qui est important.

### Quand utiliser un heartbeat

-   **Vérifications périodiques multiples** : Au lieu de 5 tâches cron distinctes vérifiant la boîte de réception, le calendrier, la météo, les notifications et l'état du projet, un seul heartbeat peut regrouper toutes ces vérifications.
-   **Décisions conscientes du contexte** : L'agent a le contexte complet de la session principale, il peut donc prendre des décisions intelligentes sur ce qui est urgent et ce qui peut attendre.
-   **Continuité conversationnelle** : Les exécutions de heartbeat partagent la même session, donc l'agent se souvient des conversations récentes et peut assurer un suivi naturel.
-   **Surveillance à faible surcharge** : Un heartbeat remplace de nombreuses petites tâches de sondage.

### Avantages du heartbeat

-   **Regroupe plusieurs vérifications** : Un tour d'agent peut examiner la boîte de réception, le calendrier et les notifications ensemble.
-   **Réduit les appels API** : Un seul heartbeat est moins cher que 5 tâches cron isolées.
-   **Conscient du contexte** : L'agent sait sur quoi vous avez travaillé et peut prioriser en conséquence.
-   **Suppression intelligente** : Si rien ne nécessite d'attention, l'agent répond `HEARTBEAT_OK` et aucun message n'est délivré.
-   **Timing naturel** : Dérive légèrement en fonction de la charge de la file d'attente, ce qui convient à la plupart des surveillances.

### Exemple de heartbeat : liste de contrôle HEARTBEAT.md

```bash
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

L'agent lit ceci à chaque heartbeat et traite tous les éléments en un seul tour.

### Configuration du heartbeat

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // interval
        target: "last", // explicit alert delivery target (default is "none")
        activeHours: { start: "08:00", end: "22:00" }, // optional
      },
    },
  },
}
```

Voir [Heartbeat](../gateway/heartbeat.md) pour la configuration complète.

## Cron : Planification précise

Les tâches cron s'exécutent à des heures précises et peuvent s'exécuter dans des sessions isolées sans affecter le contexte principal. Les planifications récurrentes en début d'heure sont automatiquement réparties par un décalage déterministe par tâche dans une fenêtre de 0 à 5 minutes.

### Quand utiliser cron

-   **Timing exact requis** : "Envoyer ceci à 9h00 tous les lundis" (pas "vers 9h").
-   **Tâches autonomes** : Tâches qui n'ont pas besoin de contexte conversationnel.
-   **Modèle/réflexion différent** : Analyse lourde qui justifie un modèle plus puissant.
-   **Rappels uniques** : "Me rappeler dans 20 minutes" avec `--at`.
-   **Tâches bruyantes/fréquentes** : Tâches qui encombreraient l'historique de la session principale.
-   **Déclencheurs externes** : Tâches qui doivent s'exécuter indépendamment du fait que l'agent soit actif ou non.

### Avantages de cron

-   **Timing précis** : Expressions cron à 5 ou 6 champs (secondes) avec prise en charge des fuseaux horaires.
-   **Répartition de charge intégrée** : les planifications récurrentes en début d'heure sont décalées jusqu'à 5 minutes par défaut.
-   **Contrôle par tâche** : outrepasser le décalage avec `--stagger ` ou forcer un timing exact avec `--exact`.
-   **Isolation de session** : S'exécute dans `cron:` sans polluer l'historique principal.
-   **Surcharges de modèle** : Utiliser un modèle moins cher ou plus puissant par tâche.
-   **Contrôle de la livraison** : Les tâches isolées sont par défaut en mode `announce` (résumé) ; choisissez `none` si nécessaire.
-   **Livraison immédiate** : Le mode announce publie directement sans attendre le heartbeat.
-   **Aucun contexte d'agent nécessaire** : S'exécute même si la session principale est inactive ou compactée.
-   **Prise en charge des tâches uniques** : `--at` pour des horodatages futurs précis.

### Exemple cron : Briefing matinal quotidien

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Ceci s'exécute exactement à 7h00 heure de New York, utilise Opus pour la qualité et annonce un résumé directement sur WhatsApp.

### Exemple cron : Rappel unique

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Voir [Tâches cron](./cron-jobs.md) pour la référence CLI complète.

## Organigramme de décision

```
La tâche doit-elle s'exécuter à un moment EXACT ?
  OUI -> Utiliser cron
  NON -> Continuer...

La tâche a-t-elle besoin d'être isolée de la session principale ?
  OUI -> Utiliser cron (isolé)
  NON -> Continuer...

Cette tâche peut-elle être regroupée avec d'autres vérifications périodiques ?
  OUI -> Utiliser heartbeat (ajouter à HEARTBEAT.md)
  NON -> Utiliser cron

S'agit-il d'un rappel unique ?
  OUI -> Utiliser cron avec --at
  NON -> Continuer...

A-t-elle besoin d'un modèle ou d'un niveau de réflexion différent ?
  OUI -> Utiliser cron (isolé) avec --model/--thinking
  NON -> Utiliser heartbeat
```

## Combiner les deux

La configuration la plus efficace utilise **les deux** :

1.  **Heartbeat** gère la surveillance de routine (boîte de réception, calendrier, notifications) en un seul tour groupé toutes les 30 minutes.
2.  **Cron** gère les planifications précises (rapports quotidiens, revues hebdomadaires) et les rappels uniques.

### Exemple : Configuration d'automatisation efficace

**HEARTBEAT.md** (vérifié toutes les 30 min) :

```bash
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Tâches cron** (timing précis) :

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# Weekly project review on Mondays at 9am
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

## Lobster : Workflows déterministes avec approbations

Lobster est le moteur d'exécution de workflow pour **les pipelines d'outils multi-étapes** qui nécessitent une exécution déterministe et des approbations explicites. Utilisez-le lorsque la tâche est plus qu'un simple tour d'agent unique, et que vous voulez un workflow reprise avec des points de contrôle humains.

### Quand Lobster convient

-   **Automatisation multi-étapes** : Vous avez besoin d'un pipeline fixe d'appels d'outils, pas d'une simple invite ponctuelle.
-   **Portes d'approbation** : Les effets secondaires doivent être mis en pause jusqu'à votre approbation, puis reprendre.
-   **Exécutions reprises** : Continuer un workflow en pause sans ré-exécuter les étapes précédentes.

### Comment il s'associe avec heartbeat et cron

-   **Heartbeat/cron** décident *quand* une exécution a lieu.
-   **Lobster** définit *quelles étapes* se produisent une fois l'exécution démarrée.

Pour les workflows planifiés, utilisez cron ou heartbeat pour déclencher un tour d'agent qui appelle Lobster. Pour les workflows ad hoc, appelez Lobster directement.

### Notes opérationnelles (du code)

-   Lobster s'exécute en tant que **sous-processus local** (CLI `lobster`) en mode outil et renvoie une **enveloppe JSON**.
-   Si l'outil renvoie `needs_approval`, vous reprenez avec un `resumeToken` et un drapeau `approve`.
-   L'outil est un **plugin optionnel** ; activez-le de manière additive via `tools.alsoAllow: ["lobster"]` (recommandé).
-   Lobster s'attend à ce que la CLI `lobster` soit disponible dans le `PATH`.

Voir [Lobster](../tools/lobster.md) pour l'utilisation complète et des exemples.

## Session principale vs Session isolée

Le heartbeat et cron peuvent tous deux interagir avec la session principale, mais différemment :

|  | Heartbeat | Cron (principal) | Cron (isolé) |
| --- | --- | --- | --- |
| Session | Principale | Principale (via événement système) | `cron:` |
| Historique | Partagé | Partagé | Nouveau à chaque exécution |
| Contexte | Complet | Complet | Aucun (démarre vierge) |
| Modèle | Modèle de la session principale | Modèle de la session principale | Peut être outrepassé |
| Sortie | Livré si pas `HEARTBEAT_OK` | Invite heartbeat + événement | Annonce un résumé (par défaut) |

### Quand utiliser cron en session principale

Utilisez `--session main` avec `--system-event` lorsque vous voulez :

-   Que le rappel/l'événement apparaisse dans le contexte de la session principale
-   Que l'agent le gère lors du prochain heartbeat avec le contexte complet
-   Pas d'exécution isolée séparée

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### Quand utiliser cron isolé

Utilisez `--session isolated` lorsque vous voulez :

-   Une ardoise propre sans contexte antérieur
-   Des paramètres de modèle ou de réflexion différents
-   Annoncer des résumés directement sur un canal
-   Un historique qui n'encombre pas la session principale

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --announce
```

## Considérations de coût

| Mécanisme | Profil de coût |
| --- | --- |
| Heartbeat | Un tour toutes les N minutes ; évolue avec la taille de HEARTBEAT.md |
| Cron (principal) | Ajoute un événement au prochain heartbeat (pas de tour isolé) |
| Cron (isolé) | Tour d'agent complet par tâche ; peut utiliser un modèle moins cher |

**Conseils** :

-   Gardez `HEARTBEAT.md` petit pour minimiser la surcharge en tokens.
-   Regroupez les vérifications similaires dans un heartbeat au lieu de plusieurs tâches cron.
-   Utilisez `target: "none"` sur le heartbeat si vous ne voulez qu'un traitement interne.
-   Utilisez cron isolé avec un modèle moins cher pour les tâches de routine.

## Liens connexes

-   [Heartbeat](../gateway/heartbeat.md) - configuration complète du heartbeat
-   [Tâches cron](./cron-jobs.md) - référence complète de la CLI et de l'API cron
-   [Système](../cli/system.md) - événements système + contrôles du heartbeat

[Tâches Cron](./cron-jobs.md)[Dépannage de l'automatisation](./troubleshooting.md)