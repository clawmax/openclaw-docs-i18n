

  Configuration et opérations

  
# Outil Exec et Processus en arrière-plan

OpenClaw exécute des commandes shell via l'outil `exec` et conserve les tâches de longue durée en mémoire. L'outil `process` gère ces sessions en arrière-plan.

## Outil exec

Paramètres clés :

-   `command` (obligatoire)
-   `yieldMs` (par défaut 10000) : passe en arrière-plan automatiquement après ce délai
-   `background` (booléen) : passe en arrière-plan immédiatement
-   `timeout` (secondes, par défaut 1800) : tue le processus après ce délai d'attente
-   `elevated` (booléen) : exécute sur l'hôte si le mode élevé est activé/autorisé
-   Besoin d'un vrai TTY ? Définissez `pty: true`.
-   `workdir`, `env`

Comportement :

-   Les exécutions au premier plan renvoient la sortie directement.
-   Lorsqu'elles passent en arrière-plan (explicite ou par délai d'attente), l'outil renvoie `status: "running"` + `sessionId` et une courte fin de sortie.
-   La sortie est conservée en mémoire jusqu'à ce que la session soit interrogée ou effacée.
-   Si l'outil `process` est interdit, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`.
-   Les commandes exec lancées reçoivent `OPENCLAW_SHELL=exec` pour des règles de shell/profil sensibles au contexte.

## Pontage des processus enfants

Lors du lancement de processus enfants de longue durée en dehors des outils exec/process (par exemple, des relances CLI ou des assistants de passerelle), attachez l'assistant de pontage de processus enfant afin que les signaux de terminaison soient transmis et que les écouteurs soient détachés à la sortie/erreur. Cela évite les processus orphelins sur systemd et maintient un comportement d'arrêt cohérent sur toutes les plateformes. Remplacements d'environnement :

-   `PI_BASH_YIELD_MS` : délai de passage en arrière-plan par défaut (ms)
-   `PI_BASH_MAX_OUTPUT_CHARS` : limite de sortie en mémoire (caractères)
-   `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS` : limite de sortie en attente par flux stdout/stderr (caractères)
-   `PI_BASH_JOB_TTL_MS` : durée de vie des sessions terminées (ms, limitée à 1 min – 3 h)

Configuration (préférée) :

-   `tools.exec.backgroundMs` (par défaut 10000)
-   `tools.exec.timeoutSec` (par défaut 1800)
-   `tools.exec.cleanupMs` (par défaut 1800000)
-   `tools.exec.notifyOnExit` (par défaut true) : met en file d'attente un événement système + demande un battement de cœur lorsqu'un exec en arrière-plan se termine.
-   `tools.exec.notifyOnExitEmptySuccess` (par défaut false) : lorsqu'à true, met également en file d'attente des événements d'achèvement pour les exécutions en arrière-plan réussies qui n'ont produit aucune sortie.

## Outil process

Actions :

-   `list` : sessions en cours + terminées
-   `poll` : récupère la nouvelle sortie pour une session (signale également l'état de sortie)
-   `log` : lit la sortie agrégée (prend en charge `offset` + `limit`)
-   `write` : envoie l'entrée standard (`data`, `eof` facultatif)
-   `kill` : termine une session en arrière-plan
-   `clear` : supprime une session terminée de la mémoire
-   `remove` : tue si en cours d'exécution, sinon efface si terminée

Notes :

-   Seules les sessions en arrière-plan sont listées/persistées en mémoire.
-   Les sessions sont perdues lors du redémarrage du processus (pas de persistance sur disque).
-   Les journaux de session ne sont enregistrés dans l'historique du chat que si vous exécutez `process poll/log` et que le résultat de l'outil est enregistré.
-   `process` est limité par agent ; il ne voit que les sessions démarrées par cet agent.
-   `process list` inclut un `name` dérivé (verbe de commande + cible) pour des scans rapides.
-   `process log` utilise `offset`/`limit` basés sur les lignes.
-   Lorsque `offset` et `limit` sont omis, il renvoie les 200 dernières lignes et inclut un indice de pagination.
-   Lorsque `offset` est fourni et `limit` est omis, il renvoie de `offset` jusqu'à la fin (non limité à 200).

## Exemples

Exécuter une tâche longue et interroger plus tard :

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

Démarrer immédiatement en arrière-plan :

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

Envoyer une entrée standard :

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```

[Verrou de la passerelle](./gateway-lock.md)[Passerelles multiples](./multiple-gateways.md)