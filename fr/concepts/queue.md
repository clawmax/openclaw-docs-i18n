title: "Gestion de la file d'attente pour les exécutions d'agent de réponse automatique dans OpenClaw"
description: "Apprenez comment OpenClaw sérialise les messages entrants à l'aide d'une file d'attente pour éviter les collisions, gérer la concurrence et configurer les modes et options de file."
keywords: ["file d'attente de messages", "réponse automatique", "concurrence", "gestion de session", "modes de file", "messages entrants", "exécutions d'agent", "sérialisation"]
---

  Messages et livraison

  
# File de commandes

Nous sérialisons les exécutions de réponse automatique entrantes (tous canaux) via une petite file d'attente en processus pour empêcher que plusieurs exécutions d'agent n'entrent en collision, tout en permettant un parallélisme sécurisé entre les sessions.

## Pourquoi

-   Les exécutions de réponse automatique peuvent être coûteuses (appels LLM) et peuvent entrer en collision lorsque plusieurs messages entrants arrivent à peu près en même temps.
-   La sérialisation évite la concurrence pour les ressources partagées (fichiers de session, journaux, stdin CLI) et réduit le risque de limites de débit en amont.

## Fonctionnement

-   Une file FIFO sensible aux voies draine chaque voie avec une limite de concurrence configurable (par défaut 1 pour les voies non configurées ; main par défaut à 4, subagent à 8).
-   `runEmbeddedPiAgent` met en file d'attente par **clé de session** (voie `session:`) pour garantir une seule exécution active par session.
-   Chaque exécution de session est ensuite mise en file d'attente dans une **voie globale** (`main` par défaut) de sorte que le parallélisme global est limité par `agents.defaults.maxConcurrent`.
-   Lorsque la journalisation verbeuse est activée, les exécutions en file d'attente émettent un bref avis si elles ont attendu plus de ~2s avant de démarrer.
-   Les indicateurs de frappe se déclenchent toujours immédiatement lors de la mise en file d'attente (lorsque pris en charge par le canal) afin que l'expérience utilisateur reste inchangée pendant l'attente de notre tour.

## Modes de file (par canal)

Les messages entrants peuvent orienter l'exécution en cours, attendre un tour de suivi, ou faire les deux :

-   `steer` : injecte immédiatement dans l'exécution en cours (annule les appels d'outils en attente après la prochaine limite d'outil). Si pas en streaming, bascule vers followup.
-   `followup` : met en file d'attente pour le prochain tour de l'agent après la fin de l'exécution en cours.
-   `collect` : fusionne tous les messages en file d'attente en un **seul** tour de suivi (par défaut). Si les messages ciblent des canaux/fils de discussion différents, ils sont drainés individuellement pour préserver le routage.
-   `steer-backlog` (alias `steer+backlog`) : oriente maintenant **et** conserve le message pour un tour de suivi.
-   `interrupt` (hérité) : interrompt l'exécution active pour cette session, puis exécute le message le plus récent.
-   `queue` (alias hérité) : identique à `steer`.

Steer-backlog signifie que vous pouvez obtenir une réponse de suivi après l'exécution orientée, donc les surfaces de streaming peuvent sembler être des doublons. Préférez `collect`/`steer` si vous voulez une réponse par message entrant. Envoyez `/queue collect` comme commande autonome (par session) ou définissez `messages.queue.byChannel.discord: "collect"`. Valeurs par défaut (si non définies dans la configuration) :

-   Toutes les surfaces → `collect`

Configurez globalement ou par canal via `messages.queue` :

```json
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

## Options de file

Les options s'appliquent à `followup`, `collect`, et `steer-backlog` (et à `steer` lorsqu'il bascule vers followup) :

-   `debounceMs` : attendre un silence avant de démarrer un tour de suivi (évite les "continue, continue").
-   `cap` : nombre maximum de messages en file d'attente par session.
-   `drop` : politique de débordement (`old`, `new`, `summarize`).

Summarize conserve une courte liste à puces des messages ignorés et l'injecte comme une invite de suivi synthétique. Valeurs par défaut : `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

## Remplacements par session

-   Envoyez `/queue ` comme commande autonome pour stocker le mode pour la session en cours.
-   Les options peuvent être combinées : `/queue collect debounce:2s cap:25 drop:summarize`
-   `/queue default` ou `/queue reset` efface le remplacement de session.

## Portée et garanties

-   S'applique aux exécutions d'agent de réponse automatique sur tous les canaux entrants qui utilisent le pipeline de réponse de la passerelle (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, etc.).
-   La voie par défaut (`main`) est à l'échelle du processus pour les messages entrants + les battements de cœur principaux ; définissez `agents.defaults.maxConcurrent` pour autoriser plusieurs sessions en parallèle.
-   Des voies supplémentaires peuvent exister (par ex. `cron`, `subagent`) afin que les tâches en arrière-plan puissent s'exécuter en parallèle sans bloquer les réponses entrantes.
-   Les voies par session garantissent qu'un seul agent exécute une session donnée à la fois.
-   Aucune dépendance externe ni thread de travail en arrière-plan ; TypeScript pur + promesses.

## Dépannage

-   Si les commandes semblent bloquées, activez les journaux verbeux et recherchez les lignes "queued for …ms" pour confirmer que la file se vide.
-   Si vous avez besoin de la profondeur de la file, activez les journaux verbeux et surveillez les lignes de temporisation de la file.

[Politique de nouvelle tentative](./retry.md)

---