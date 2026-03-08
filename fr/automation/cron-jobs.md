title: "Guide des tâches cron OpenClaw : Planification et automatisation"
description: "Apprenez à planifier des tâches automatisées avec les tâches cron OpenClaw. Configurez des rappels ponctuels, des tâches récurrentes et choisissez entre les modes d'exécution principal ou isolé."
keywords: ["cron openclaw", "planification automatisation", "tâches cron", "planificateur gateway", "tour d'agent isolé", "événements système", "tâches récurrentes", "livraison de job"]
---

  Automatisation

  
# Tâches Cron

> **Cron vs Heartbeat ?** Voir [Cron vs Heartbeat](./cron-vs-heartbeat.md) pour savoir quand utiliser chacun.

Cron est le planificateur intégré de la Gateway. Il persiste les jobs, réveille l'agent au bon moment et peut éventuellement renvoyer la sortie vers un chat. Si vous voulez *"exécuter ceci chaque matin"* ou *"relancer l'agent dans 20 minutes"*, cron est le mécanisme. Dépannage : [/automation/troubleshooting](./troubleshooting.md)

## TL;DR

-   Cron s'exécute **à l'intérieur de la Gateway** (pas à l'intérieur du modèle).
-   Les jobs persistent sous `~/.openclaw/cron/` donc les redémarrages ne perdent pas les planifications.
-   Deux styles d'exécution :
    -   **Session principale** : mettre en file d'attente un événement système, puis exécuter au prochain heartbeat.
    -   **Isolée** : exécuter un tour d'agent dédié dans `cron:`, avec livraison (annonce par défaut ou aucune).
-   Les réveils sont de première classe : un job peut demander "réveiller maintenant" vs "prochain heartbeat".
-   La publication webhook est par job via `delivery.mode = "webhook"` + `delivery.to = ""`.
-   La rétrocompatibilité reste pour les jobs stockés avec `notify: true` quand `cron.webhook` est défini, migrez ces jobs vers le mode de livraison webhook.

## Démarrage rapide (actionnable)

Créez un rappel ponctuel, vérifiez son existence et exécutez-le immédiatement :

```bash
openclaw cron add \
  --name "Rappel" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Rappel : vérifier le brouillon de la doc cron" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

Planifiez un job isolé récurrent avec livraison :

```bash
openclaw cron add \
  --name "Briefing matinal" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Résumez les mises à jour de la nuit." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## Équivalents d'appel d'outil (outil cron Gateway)

Pour les formes JSON canoniques et exemples, voir [Schéma JSON pour les appels d'outil](./cron-jobs.md#json-schema-for-tool-calls).

## Où sont stockées les tâches cron

Les tâches cron sont persistées sur l'hôte de la Gateway à `~/.openclaw/cron/jobs.json` par défaut. La Gateway charge le fichier en mémoire et le réécrit lors des modifications, donc les éditions manuelles ne sont sûres que lorsque la Gateway est arrêtée. Préférez `openclaw cron add/edit` ou l'API d'appel d'outil cron pour les modifications.

## Aperçu pour débutants

Considérez une tâche cron comme : **quand** exécuter + **quoi** faire.

1.  **Choisissez une planification**
    -   Rappel ponctuel → `schedule.kind = "at"` (CLI : `--at`)
    -   Job répétitif → `schedule.kind = "every"` ou `schedule.kind = "cron"`
    -   Si votre horodatage ISO omet un fuseau horaire, il est traité comme **UTC**.
2.  **Choisissez où il s'exécute**
    -   `sessionTarget: "main"` → s'exécute lors du prochain heartbeat avec le contexte principal.
    -   `sessionTarget: "isolated"` → exécute un tour d'agent dédié dans `cron:`.
3.  **Choisissez la charge utile**
    -   Session principale → `payload.kind = "systemEvent"`
    -   Session isolée → `payload.kind = "agentTurn"`

Optionnel : les jobs ponctuels (`schedule.kind = "at"`) se suppriment après succès par défaut. Définissez `deleteAfterRun: false` pour les conserver (ils se désactiveront après succès).

## Concepts

### Jobs

Un job cron est un enregistrement stocké avec :

-   une **planification** (quand il doit s'exécuter),
-   une **charge utile** (ce qu'il doit faire),
-   un **mode de livraison** optionnel (`announce`, `webhook`, ou `none`).
-   une **liaison d'agent** optionnelle (`agentId`) : exécuter le job sous un agent spécifique ; si absent ou inconnu, la gateway revient à l'agent par défaut.

Les jobs sont identifiés par un `jobId` stable (utilisé par les API CLI/Gateway). Dans les appels d'outil d'agent, `jobId` est canonique ; l'ancien `id` est accepté pour la compatibilité. Les jobs ponctuels se suppriment automatiquement après succès par défaut ; définissez `deleteAfterRun: false` pour les conserver.

### Planifications

Cron prend en charge trois types de planification :

-   `at` : horodatage ponctuel via `schedule.at` (ISO 8601).
-   `every` : intervalle fixe (ms).
-   `cron` : expression cron à 5 champs (ou 6 avec secondes) avec fuseau horaire IANA optionnel.

Les expressions cron utilisent `croner`. Si un fuseau horaire est omis, le fuseau horaire local de l'hôte de la Gateway est utilisé. Pour réduire les pics de charge en début d'heure sur de nombreuses gateways, OpenClaw applique une fenêtre d'étalement déterministe par job allant jusqu'à 5 minutes pour les expressions récurrentes en début d'heure (par exemple `0 * * * *`, `0 */2 * * *`). Les expressions à heure fixe comme `0 7 * * *` restent exactes. Pour toute planification cron, vous pouvez définir une fenêtre d'étalement explicite avec `schedule.staggerMs` (`0` garde un timing exact). Raccourcis CLI :

-   `--stagger 30s` (ou `1m`, `5m`) pour définir une fenêtre d'étalement explicite.
-   `--exact` pour forcer `staggerMs = 0`.

### Exécution principale vs isolée

#### Jobs de session principale (événements système)

Les jobs principaux mettent en file d'attente un événement système et réveillent éventuellement le runner de heartbeat. Ils doivent utiliser `payload.kind = "systemEvent"`.

-   `wakeMode: "now"` (par défaut) : l'événement déclenche une exécution immédiate du heartbeat.
-   `wakeMode: "next-heartbeat"` : l'événement attend le prochain heartbeat planifié.

C'est le meilleur choix quand vous voulez l'invite de heartbeat normale + le contexte de session principale. Voir [Heartbeat](../gateway/heartbeat.md).

#### Jobs isolés (sessions cron dédiées)

Les jobs isolés exécutent un tour d'agent dédié dans la session `cron:`. Comportements clés :

-   L'invite est préfixée par `[cron: ]` pour la traçabilité.
-   Chaque exécution démarre un **nouvel identifiant de session** (pas de report de conversation précédente).
-   Comportement par défaut : si `delivery` est omis, les jobs isolés annoncent un résumé (`delivery.mode = "announce"`).
-   `delivery.mode` choisit ce qui se passe :
    -   `announce` : livrer un résumé vers le canal cible et poster un bref résumé dans la session principale.
    -   `webhook` : POSTER la charge utile de l'événement terminé vers `delivery.to` quand l'événement terminé inclut un résumé.
    -   `none` : interne uniquement (pas de livraison, pas de résumé dans la session principale).
-   `wakeMode` contrôle quand le résumé de la session principale est posté :
    -   `now` : heartbeat immédiat.
    -   `next-heartbeat` : attend le prochain heartbeat planifié.

Utilisez les jobs isolés pour les tâches bruyantes, fréquentes ou "corvées en arrière-plan" qui ne devraient pas polluer l'historique de votre chat principal.

### Formes de charge utile (ce qui s'exécute)

Deux types de charge utile sont supportés :

-   `systemEvent` : session principale uniquement, routé via l'invite de heartbeat.
-   `agentTurn` : session isolée uniquement, exécute un tour d'agent dédié.

Champs communs `agentTurn` :

-   `message` : invite texte requise.
-   `model` / `thinking` : remplacements optionnels (voir ci-dessous).
-   `timeoutSeconds` : remplacement de délai d'attente optionnel.
-   `lightContext` : mode d'amorçage léger optionnel pour les jobs qui n'ont pas besoin de l'injection de fichiers d'amorçage de l'espace de travail.

Configuration de livraison :

-   `delivery.mode` : `none` | `announce` | `webhook`.
-   `delivery.channel` : `last` ou un canal spécifique.
-   `delivery.to` : cible spécifique au canal (annonce) ou URL webhook (mode webhook).
-   `delivery.bestEffort` : éviter d'échouer le job si la livraison d'annonce échoue.

La livraison par annonce supprime les envois d'outil de messagerie pour l'exécution ; utilisez `delivery.channel`/`delivery.to` pour cibler le chat à la place. Quand `delivery.mode = "none"`, aucun résumé n'est posté dans la session principale. Si `delivery` est omis pour les jobs isolés, OpenClaw utilise par défaut `announce`.

#### Flux de livraison par annonce

Quand `delivery.mode = "announce"`, cron livre directement via les adaptateurs de canal sortants. L'agent principal n'est pas lancé pour créer ou transférer le message. Détails du comportement :

-   Contenu : la livraison utilise les charges utiles sortantes de l'exécution isolée (texte/média) avec un découpage et un formatage de canal normaux.
-   Les réponses heartbeat uniquement (`HEARTBEAT_OK` sans contenu réel) ne sont pas livrées.
-   Si l'exécution isolée a déjà envoyé un message à la même cible via l'outil de message, la livraison est ignorée pour éviter les doublons.
-   Les cibles de livraison manquantes ou invalides font échouer le job sauf si `delivery.bestEffort = true`.
-   Un bref résumé est posté dans la session principale uniquement quand `delivery.mode = "announce"`.
-   Le résumé de la session principale respecte `wakeMode` : `now` déclenche un heartbeat immédiat et `next-heartbeat` attend le prochain heartbeat planifié.

#### Flux de livraison webhook

Quand `delivery.mode = "webhook"`, cron POSTE la charge utile de l'événement terminé vers `delivery.to` quand l'événement terminé inclut un résumé. Détails du comportement :

-   Le point de terminaison doit être une URL HTTP(S) valide.
-   Aucune livraison de canal n'est tentée en mode webhook.
-   Aucun résumé de session principale n'est posté en mode webhook.
-   Si `cron.webhookToken` est défini, l'en-tête d'authentification est `Authorization: Bearer <cron.webhookToken>`.
-   Rétrocompatibilité dépréciée : les jobs hérités stockés avec `notify: true` postent toujours vers `cron.webhook` (si configuré), avec un avertissement pour que vous puissiez migrer vers `delivery.mode = "webhook"`.

### Remplacements de modèle et de réflexion

Les jobs isolés (`agentTurn`) peuvent remplacer le modèle et le niveau de réflexion :

-   `model` : Chaîne fournisseur/modèle (par ex., `anthropic/claude-sonnet-4-20250514`) ou alias (par ex., `opus`)
-   `thinking` : Niveau de réflexion (`off`, `minimal`, `low`, `medium`, `high`, `xhigh` ; modèles GPT-5.2 + Codex uniquement)

Note : Vous pouvez définir `model` sur les jobs de session principale aussi, mais cela change le modèle de session principale partagé. Nous recommandons les remplacements de modèle uniquement pour les jobs isolés pour éviter des changements de contexte inattendus. Priorité de résolution :

1.  Remplacement de la charge utile du job (le plus élevé)
2.  Valeurs par défaut spécifiques au hook (par ex., `hooks.gmail.model`)
3.  Valeur par défaut de la configuration de l'agent

### Contexte d'amorçage léger

Les jobs isolés (`agentTurn`) peuvent définir `lightContext: true` pour s'exécuter avec un contexte d'amorçage léger.

-   Utilisez ceci pour les tâches planifiées qui n'ont pas besoin de l'injection de fichiers d'amorçage de l'espace de travail.
-   En pratique, le runtime embarqué s'exécute avec `bootstrapContextMode: "lightweight"`, qui garde intentionnellement le contexte d'amorçage cron vide.
-   Équivalents CLI : `openclaw cron add --light-context ...` et `openclaw cron edit --light-context`.

### Livraison (canal + cible)

Les jobs isolés peuvent livrer la sortie vers un canal via la configuration de haut niveau `delivery` :

-   `delivery.mode` : `announce` (livraison canal), `webhook` (POST HTTP), ou `none`.
-   `delivery.channel` : `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`.
-   `delivery.to` : cible de destinataire spécifique au canal.

La livraison `announce` n'est valide que pour les jobs isolés (`sessionTarget: "isolated"`). La livraison `webhook` est valide pour les jobs principaux et isolés. Si `delivery.channel` ou `delivery.to` est omis, cron peut revenir à la "dernière route" de la session principale (le dernier endroit où l'agent a répondu). Rappels de format de cible :

-   Les cibles Slack/Discord/Mattermost (plugin) doivent utiliser des préfixes explicites (par ex. `channel:`, `user:`) pour éviter l'ambiguïté.
-   Les sujets Telegram doivent utiliser la forme `:topic:` (voir ci-dessous).

#### Cibles de livraison Telegram (sujets / fils de forum)

Telegram prend en charge les sujets de forum via `message_thread_id`. Pour la livraison cron, vous pouvez encoder le sujet/fil dans le champ `to` :

-   `-1001234567890` (id de chat uniquement)
-   `-1001234567890:topic:123` (préféré : marqueur de sujet explicite)
-   `-1001234567890:123` (raccourci : suffixe numérique)

Les cibles préfixées comme `telegram:...` / `telegram:group:...` sont aussi acceptées :

-   `telegram:group:-1001234567890:topic:123`

## Schéma JSON pour les appels d'outil

Utilisez ces formes lors de l'appel direct des outils `cron.*` de la Gateway (appels d'outil d'agent ou RPC). Les drapeaux CLI acceptent des durées humaines comme `20m`, mais les appels d'outil doivent utiliser une chaîne ISO 8601 pour `schedule.at` et des millisecondes pour `schedule.everyMs`.

### Paramètres cron.add

Job ponctuel, session principale (événement système) :

```json
{
  "name": "Rappel",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Texte du rappel" },
  "deleteAfterRun": true
}
```

Job récurrent, isolé avec livraison :

```json
{
  "name": "Briefing matinal",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Résumez les mises à jour de la nuit.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

Notes :

-   `schedule.kind` : `at` (`at`), `every` (`everyMs`), ou `cron` (`expr`, `tz` optionnel).
-   `schedule.at` accepte ISO 8601 (fuseau horaire optionnel ; traité comme UTC quand omis).
-   `everyMs` est en millisecondes.
-   `sessionTarget` doit être `"main"` ou `"isolated"` et doit correspondre à `payload.kind`.
-   Champs optionnels : `agentId`, `description`, `enabled`, `deleteAfterRun` (par défaut true pour `at`), `delivery`.
-   `wakeMode` par défaut `"now"` quand omis.

### Paramètres cron.update

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

Notes :

-   `jobId` est canonique ; `id` est accepté pour la compatibilité.
-   Utilisez `agentId: null` dans le patch pour effacer une liaison d'agent.

### Paramètres cron.run et cron.remove

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## Stockage & historique

-   Stockage des jobs : `~/.openclaw/cron/jobs.json` (JSON géré par la Gateway).
-   Historique des exécutions : `~/.openclaw/cron/runs/.jsonl` (JSONL, auto-élagué par taille et nombre de lignes).
-   Les sessions d'exécution cron isolées dans `sessions.json` sont élaguées par `cron.sessionRetention` (par défaut `24h` ; définir `false` pour désactiver).
-   Chemin de stockage de remplacement : `cron.store` dans la configuration.

## Politique de réessai

Quand un job échoue, OpenClaw classe les erreurs comme **transitoires** (réessayables) ou **permanentes** (désactivation immédiate).

### Erreurs transitoires (réessayées)

-   Limite de débit (429, trop de requêtes, ressources épuisées)
-   Surcharge du fournisseur (par exemple Anthropic `529 overloaded_error`, résumés de secours en surcharge)
-   Erreurs réseau (timeout, ECONNRESET, échec de récupération, socket)
-   Erreurs serveur (5xx)
-   Erreurs liées à Cloudflare

### Erreurs permanentes (pas de réessai)

-   Échecs d'authentification (clé API invalide, non autorisé)
-   Erreurs de configuration ou de validation
-   Autres erreurs non transitoires

### Comportement par défaut (sans configuration)

**Jobs ponctuels (`schedule.kind: "at"`) :**

-   En cas d'erreur transitoire : réessayer jusqu'à 3 fois avec backoff exponentiel (30s → 1m → 5m).
-   En cas d'erreur permanente : désactiver immédiatement.
-   En cas de succès ou de saut : désactiver (ou supprimer si `deleteAfterRun: true`).

**Jobs récurrents (`cron` / `every`) :**

-   En cas d'erreur : appliquer un backoff exponentiel (30s → 1m → 5m → 15m → 60m) avant la prochaine exécution planifiée.
-   Le job reste activé ; le backoff est réinitialisé après la prochaine exécution réussie.

Configurez `cron.retry` pour remplacer ces valeurs par défaut (voir [Configuration](./cron-jobs.md#configuration)).

## Configuration

```json
{
  cron: {
    enabled: true, // par défaut true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // par défaut 1
    // Optionnel : remplacer la politique de réessai pour les jobs ponctuels
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },
    webhook: "https://example.invalid/legacy", // rétrocompatibilité dépréciée pour les jobs stockés notify:true
    webhookToken: "replace-with-dedicated-webhook-token", // jeton porteur optionnel pour le mode webhook
    sessionRetention: "24h", // chaîne de durée ou false
    runLog: {
      maxBytes: "2mb", // par défaut 2_000_000 octets
      keepLines: 2000, // par défaut 2000
    },
  },
}
```

Comportement d'élagage des journaux d'exécution :

-   `cron.runLog.maxBytes` : taille maximale du fichier journal d'exécution avant élagage.
-   `cron.runLog.keepLines` : lors de l'élagage, ne conserver que les N lignes les plus récentes.
-   Les deux s'appliquent aux fichiers `cron/runs/.jsonl`.

Comportement webhook :

-   Préféré : définir `delivery.mode: "webhook"` avec `delivery.to: "https://..."` par job.
-   Les URL webhook doivent être des URL `http://` ou `https://` valides.
-   Lors du POST, la charge utile est le JSON de l'événement cron terminé.
-   Si `cron.webhookToken` est défini, l'en-tête d'authentification est `Authorization: Bearer <cron.webhookToken>`.
-   Si `cron.webhookToken` n'est pas défini, aucun en-tête `Authorization` n'est envoyé.
-   Rétrocompatibilité dépréciée : les jobs hérités stockés avec `notify: true` utilisent toujours `cron.webhook` quand présent.

Désactiver cron entièrement :

-   `cron.enabled: false` (configuration)
-   `OPENCLAW_SKIP_CRON=1` (env)

## Maintenance

Cron a deux chemins de maintenance intégrés : la rétention des sessions d'exécution isolées et l'élagage des journaux d'exécution.

### Valeurs par défaut

-   `cron.sessionRetention` : `24h` (définir `false` pour désactiver l'élagage des sessions d'exécution)
-   `cron.runLog.maxBytes` : `2_000_000` octets
-   `cron.runLog.keepLines` : `2000`

### Comment ça fonctionne

-   Les exécutions isolées créent des entrées de session (`...:cron::run:`) et des fichiers de transcription.
-   Le récupérateur supprime les entrées de session d'exécution expirées plus anciennes que `cron.sessionRetention`.
-   Pour les sessions d'exécution supprimées qui ne sont plus référencées par le stockage de session, OpenClaw archive les fichiers de transcription et purge les anciennes archives supprimées sur la même fenêtre de rétention.
-   Après chaque ajout d'exécution, `cron/runs/.jsonl` est vérifié en taille :
    -   si la taille du fichier dépasse `runLog.maxBytes`, il est réduit aux `runLog.keepLines` lignes les plus récentes.

### Mise en garde sur les performances pour les planificateurs à volume élevé

Les configurations cron à haute fréquence peuvent générer de grandes empreintes de sessions d'exécution et de journaux d'exécution. La maintenance est intégrée, mais des limites larges peuvent quand même créer un travail d'IO et de nettoyage évitable. Ce qu'il faut surveiller :

-   de longues fenêtres `cron.sessionRetention` avec de nombreuses exécutions isolées
-   un `cron.runLog.keepLines` élevé combiné à un `runLog.maxBytes` important
-   de nombreux jobs récurrents bruyants écrivant dans le même `cron/runs/.jsonl`

Ce qu'il faut faire :

-   gardez `cron.sessionRetention` aussi court que vos besoins de débogage/audit le permettent
-   gardez les journaux d'exécution limités avec des `runLog.maxBytes` et `runLog.keepLines` modérés
-   déplacez les tâches d'arrière-plan bruyantes en mode isolé avec des règles de livraison qui évitent le bavardage inutile
-   examinez périodiquement la croissance avec `openclaw cron runs` et ajustez la rétention avant que les journaux ne deviennent volumineux

### Exemples de personnalisation

Gardez les sessions d'exécution pendant une semaine et autorisez des journaux d'exécution plus grands :

```json
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

Désactivez l'élagage des sessions d'exécution isolées mais gardez l'élagage des journaux d'exécution :

```json
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

Ajustez pour une utilisation cron à volume élevé (exemple) :

```json
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## Démarrage rapide CLI

Rappel ponctuel (UTC ISO, suppression automatique après succès) :

```bash
openclaw cron add \
  --name "Envoyer un rappel" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Rappel : soumettre le rapport de dépenses." \
  --wake now \
  --delete-after-run
```

Rappel ponctuel (session principale, réveil immédiat) :

```bash
openclaw cron add \
  --name "Vérification calendrier" \
  --at "20m" \
  --session main \
  --system-event "Prochain heartbeat : vérifier le calendrier." \
  --wake now
```

Job isolé récurrent (annonce vers WhatsApp) :

```bash
openclaw cron add \
  --name "Statut matinal" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Résumez la boîte de réception + le calendrier pour aujourd'hui." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Job cron récurrent avec étalement explicite de 30 secondes :

```bash
openclaw cron add \
  --name "Surveillant minute" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Exécutez les vérifications du surveillant minute." \
  --announce
```

Job isolé récurrent (livraison vers un sujet Telegram) :

```bash
openclaw cron add \
  --name "Résumé nocturne (sujet)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Résumez la journée ; envoyez au sujet nocturne." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Job isolé avec remplacement de modèle et de réflexion :

```bash
openclaw cron add \
  --name "Analyse approfondie" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Analyse approfondie hebdomadaire de l'avancement du projet." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Sélection d'agent (configurations multi-agents) :

```bash
# Épingler un job à l'agent "ops" (revient à l'agent par défaut si cet agent est manquant)
openclaw cron add --name "Balayage Ops" --cron "0 6 * * *" --session isolated --message "Vérifier la file d'attente ops" --agent ops

# Changer ou effacer l'agent sur un job existant
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

Exécution manuelle (force est la valeur par défaut, utilisez `--due` pour exécuter seulement quand dû) :

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

Éditer un job existant (patcher des champs) :

```bash
openclaw cron edit <jobId> \
  --message "Invite mise à jour" \
  --model "opus" \
  --thinking low
```

Forcer un job cron existant à s'exécuter exactement selon la planification (pas d'étalement) :

```bash
openclaw cron edit <jobId> --exact
```

Historique des exécutions :

```bash
openclaw cron runs --id <jobId> --limit 50
```

Événement système immédiat sans créer de job :

```bash
openclaw system event --mode now --text "Prochain heartbeat : vérifier la batterie."
```

## Surface d'API Gateway

-   `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
-   `cron.run` (force ou dû), `cron.runs` Pour les événements système immédiats sans job, utilisez [`openclaw system event`](../cli/system.md).

## Dépannage

### "Rien ne s'exécute"

-   Vérifiez que cron est activé : `cron.enabled` et `OPENCLAW_SKIP_CRON`.
-   Vérifiez que la Gateway fonctionne en continu (cron s'exécute dans le processus de la Gateway).
-   Pour les planifications `cron` : confirmez le fuseau horaire (`--tz`) vs le fuseau horaire de l'hôte.

### Un job récurrent continue de retarder après des échecs

-   OpenClaw applique un backoff de réessai exponentiel pour les jobs récurrents après des erreurs consécutives : 30s, 1m, 5m, 15m, puis 60m entre les réessais.
-   Le backoff est réinitialisé automatiquement après la prochaine exécution réussie.
-   Les jobs ponctuels (`at`) réessaient les erreurs transitoires (limite de débit, surchargé, réseau, server\_error) jusqu'à 3 fois avec backoff ; les erreurs permanentes désactivent immédiatement. Voir [Politique de réessai](./cron-jobs.md#retry-policy).

### Telegram livre au mauvais endroit

-   Pour les sujets