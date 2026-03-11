

  Sessions et mémoire

  
# Gestion des sessions

OpenClaw traite **une session de discussion directe par agent** comme principale. Les discussions directes sont réduites à `agent::` (par défaut `main`), tandis que les discussions de groupe/canal obtiennent leurs propres clés. `session.mainKey` est respectée. Utilisez `session.dmScope` pour contrôler comment **les messages directs** sont regroupés :

-   `main` (par défaut) : tous les DM partagent la session principale pour la continuité.
-   `per-peer` : isoler par identifiant de l'expéditeur à travers les canaux.
-   `per-channel-peer` : isoler par canal + expéditeur (recommandé pour les boîtes de réception multi-utilisateurs).
-   `per-account-channel-peer` : isoler par compte + canal + expéditeur (recommandé pour les boîtes de réception multi-comptes). Utilisez `session.identityLinks` pour mapper les identifiants de pairs préfixés par le fournisseur à une identité canonique afin que la même personne partage une session DM à travers les canaux lors de l'utilisation de `per-peer`, `per-channel-peer` ou `per-account-channel-peer`.

## Mode DM sécurisé (recommandé pour les configurations multi-utilisateurs)

> **Avertissement de sécurité :** Si votre agent peut recevoir des DM de **plusieurs personnes**, vous devriez sérieusement envisager d'activer le mode DM sécurisé. Sans cela, tous les utilisateurs partagent le même contexte de conversation, ce qui peut divulguer des informations privées entre utilisateurs.

**Exemple du problème avec les paramètres par défaut :**

-   Alice (`<SENDER_A>`) envoie un message à votre agent sur un sujet privé (par exemple, un rendez-vous médical)
-   Bob (`<SENDER_B>`) envoie un message à votre agent demandant "De quoi parlions-nous ?"
-   Parce que les deux DM partagent la même session, le modèle peut répondre à Bob en utilisant le contexte précédent d'Alice.

**La solution :** Définissez `dmScope` pour isoler les sessions par utilisateur :

```
// ~/.openclaw/openclaw.json
{
  session: {
    // Mode DM sécurisé : isoler le contexte DM par canal + expéditeur.
    dmScope: "per-channel-peer",
  },
}
```

**Quand activer ceci :**

-   Vous avez des approbations d'appariement pour plus d'un expéditeur
-   Vous utilisez une liste d'autorisation DM avec plusieurs entrées
-   Vous définissez `dmPolicy: "open"`
-   Plusieurs numéros de téléphone ou comptes peuvent envoyer des messages à votre agent

Notes :

-   La valeur par défaut est `dmScope: "main"` pour la continuité (tous les DM partagent la session principale). C'est acceptable pour les configurations mono-utilisateur.
-   L'intégration en ligne de commande locale écrit `session.dmScope: "per-channel-peer"` par défaut lorsqu'elle n'est pas définie (les valeurs explicites existantes sont préservées).
-   Pour les boîtes de réception multi-comptes sur le même canal, préférez `per-account-channel-peer`.
-   Si la même personne vous contacte sur plusieurs canaux, utilisez `session.identityLinks` pour fusionner ses sessions DM en une seule identité canonique.
-   Vous pouvez vérifier vos paramètres DM avec `openclaw security audit` (voir [sécurité](../cli/security.md)).

## La passerelle est la source de vérité

Tout l'état de la session est **détenu par la passerelle** (le "maître" OpenClaw). Les clients d'interface utilisateur (application macOS, WebChat, etc.) doivent interroger la passerelle pour les listes de sessions et les décomptes de jetons au lieu de lire les fichiers locaux.

-   En **mode distant**, le magasin de sessions qui vous intéresse se trouve sur l'hôte de la passerelle distante, pas sur votre Mac.
-   Les décomptes de jetons affichés dans les interfaces utilisateur proviennent des champs de stockage de la passerelle (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Les clients n'analysent pas les transcriptions JSONL pour "corriger" les totaux.

## Où se trouve l'état

-   Sur **l'hôte de la passerelle** :
    -   Fichier de stockage : `~/.openclaw/agents//sessions/sessions.json` (par agent).
-   Transcripts : `~/.openclaw/agents//sessions/.jsonl` (les sessions de sujets Telegram utilisent `.../-topic-.jsonl`).
-   Le stockage est une carte `sessionKey -> { sessionId, updatedAt, ... }`. Supprimer des entrées est sûr ; elles sont recréées à la demande.
-   Les entrées de groupe peuvent inclure `displayName`, `channel`, `subject`, `room` et `space` pour étiqueter les sessions dans les interfaces utilisateur.
-   Les entrées de session incluent des métadonnées `origin` (étiquette + indices de routage) pour que les interfaces utilisateur puissent expliquer d'où vient une session.
-   OpenClaw **ne lit pas** les dossiers de sessions Pi/Tau hérités.

## Maintenance

OpenClaw applique une maintenance du magasin de sessions pour garder `sessions.json` et les artefacts de transcription limités dans le temps.

### Valeurs par défaut

-   `session.maintenance.mode` : `warn`
-   `session.maintenance.pruneAfter` : `30d`
-   `session.maintenance.maxEntries` : `500`
-   `session.maintenance.rotateBytes` : `10mb`
-   `session.maintenance.resetArchiveRetention` : par défaut `pruneAfter` (`30d`)
-   `session.maintenance.maxDiskBytes` : non défini (désactivé)
-   `session.maintenance.highWaterBytes` : par défaut `80%` de `maxDiskBytes` lorsque la budgétisation est activée

### Comment cela fonctionne

La maintenance s'exécute pendant les écritures dans le magasin de sessions, et vous pouvez la déclencher à la demande avec `openclaw sessions cleanup`.

-   `mode: "warn"` : rapporte ce qui serait évincé mais ne modifie pas les entrées/transcriptions.
-   `mode: "enforce"` : applique le nettoyage dans cet ordre :
    1.  élaguer les entrées périmées plus anciennes que `pruneAfter`
    2.  limiter le nombre d'entrées à `maxEntries` (les plus anciennes d'abord)
    3.  archiver les fichiers de transcription pour les entrées supprimées qui ne sont plus référencées
    4.  purger les anciennes archives `*.deleted.` et `*.reset.` selon la politique de rétention
    5.  faire tourner `sessions.json` lorsqu'il dépasse `rotateBytes`
    6.  si `maxDiskBytes` est défini, appliquer le budget disque vers `highWaterBytes` (les artefacts les plus anciens d'abord, puis les sessions les plus anciennes)

### Mise en garde sur les performances pour les grands magasins

Les grands magasins de sessions sont courants dans les configurations à volume élevé. Le travail de maintenance est un travail sur le chemin d'écriture, donc les très grands magasins peuvent augmenter la latence d'écriture. Ce qui augmente le plus le coût :

-   des valeurs `session.maintenance.maxEntries` très élevées
-   de longues fenêtres `pruneAfter` qui gardent des entrées périmées
-   de nombreux artefacts de transcription/archive dans `~/.openclaw/agents//sessions/`
-   activer les budgets disque (`maxDiskBytes`) sans limites d'élagage/plafond raisonnables

Que faire :

-   utilisez `mode: "enforce"` en production pour que la croissance soit automatiquement limitée
-   définissez à la fois des limites de temps et de nombre (`pruneAfter` + `maxEntries`), pas seulement une
-   définissez `maxDiskBytes` + `highWaterBytes` pour des limites supérieures strictes dans les grands déploiements
-   gardez `highWaterBytes` significativement en dessous de `maxDiskBytes` (la valeur par défaut est 80%)
-   exécutez `openclaw sessions cleanup --dry-run --json` après les changements de configuration pour vérifier l'impact projeté avant de l'appliquer
-   pour les sessions actives fréquentes, passez `--active-key` lors de l'exécution d'un nettoyage manuel

### Exemples de personnalisation

Utilisez une politique d'application conservatrice :

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "45d",
      maxEntries: 800,
      rotateBytes: "20mb",
      resetArchiveRetention: "14d",
    },
  },
}
```

Activez un budget disque strict pour le répertoire des sessions :

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

Ajustez pour les installations plus grandes (exemple) :

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```

Prévisualisez ou forcez la maintenance depuis la ligne de commande :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

## Élagage des sessions

Par défaut, OpenClaw supprime **les anciens résultats d'outils** du contexte en mémoire juste avant les appels LLM. Cela **ne réécrit pas** l'historique JSONL. Voir [/concepts/session-pruning](./session-pruning.md).

## Vidage de mémoire pré-compaction

Lorsqu'une session approche de la compaction automatique, OpenClaw peut exécuter un **tour de vidage de mémoire silencieux** qui rappelle au modèle d'écrire des notes durables sur le disque. Cela ne s'exécute que lorsque l'espace de travail est accessible en écriture. Voir [Mémoire](./memory.md) et [Compaction](./compaction.md).

## Cartographie des transports → clés de session

-   Les discussions directes suivent `session.dmScope` (par défaut `main`).
    -   `main` : `agent::` (continuité à travers les appareils/canaux).
        -   Plusieurs numéros de téléphone et canaux peuvent correspondre à la même clé principale d'agent ; ils agissent comme des transports vers une conversation unique.
    -   `per-peer` : `agent::dm:`.
    -   `per-channel-peer` : `agent:::dm:`.
    -   `per-account-channel-peer` : `agent::::dm:` (accountId par défaut `default`).
    -   Si `session.identityLinks` correspond à un identifiant de pair préfixé par le fournisseur (par exemple `telegram:123`), la clé canonique remplace `` pour que la même personne partage une session à travers les canaux.
-   Les discussions de groupe isolent l'état : `agent:::group:` (les salles/canaux utilisent `agent:::channel:`).
    -   Les sujets de forum Telegram ajoutent `:topic:` à l'identifiant de groupe pour l'isolation.
    -   Les clés héritées `group:` sont toujours reconnues pour la migration.
-   Les contextes entrants peuvent encore utiliser `group:` ; le canal est déduit de `Provider` et normalisé vers la forme canonique `agent:::group:`.
-   Autres sources :
    -   Tâches cron : `cron:<job.id>`
    -   Webhooks : `hook:` (sauf s'il est explicitement défini par le webhook)
    -   Exécutions de nœuds : `node-`

## Cycle de vie

-   Politique de réinitialisation : les sessions sont réutilisées jusqu'à leur expiration, et l'expiration est évaluée au prochain message entrant.
-   Réinitialisation quotidienne : par défaut à **4h00 heure locale sur l'hôte de la passerelle**. Une session est périmée une fois que sa dernière mise à jour est antérieure à l'heure de réinitialisation quotidienne la plus récente.
-   Réinitialisation d'inactivité (optionnelle) : `idleMinutes` ajoute une fenêtre d'inactivité glissante. Lorsque les réinitialisations quotidiennes et d'inactivité sont configurées, **celle qui expire en premier** force une nouvelle session.
-   Inactivité seule héritée : si vous définissez `session.idleMinutes` sans aucune configuration `session.reset`/`resetByType`, OpenClaw reste en mode inactivité seule pour la rétrocompatibilité.
-   Remplacements par type (optionnels) : `resetByType` vous permet de remplacer la politique pour les sessions `direct`, `group` et `thread` (thread = fils Slack/Discord, sujets Telegram, fils Matrix lorsqu'ils sont fournis par le connecteur).
-   Remplacements par canal (optionnels) : `resetByChannel` remplace la politique de réinitialisation pour un canal (s'applique à tous les types de sessions pour ce canal et prend la priorité sur `reset`/`resetByType`).
-   Déclencheurs de réinitialisation : les commandes exactes `/new` ou `/reset` (plus tout extra dans `resetTriggers`) démarrent un nouvel identifiant de session et transmettent le reste du message. `/new ` accepte un alias de modèle, `provider/model` ou un nom de fournisseur (correspondance approximative) pour définir le modèle de la nouvelle session. Si `/new` ou `/reset` est envoyé seul, OpenClaw exécute un court tour de salutation "bonjour" pour confirmer la réinitialisation.
-   Réinitialisation manuelle : supprimez des clés spécifiques du magasin ou supprimez la transcription JSONL ; le prochain message les recrée.
-   Les tâches cron isolées créent toujours un nouveau `sessionId` par exécution (pas de réutilisation en cas d'inactivité).

## Politique d'envoi (optionnelle)

Bloquez la livraison pour des types de sessions spécifiques sans lister les identifiants individuels.

```json
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Correspond à la clé de session brute (incluant le préfixe `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Remplacement à l'exécution (propriétaire uniquement) :

-   `/send on` → autoriser pour cette session
-   `/send off` → refuser pour cette session
-   `/send inherit` → effacer le remplacement et utiliser les règles de configuration Envoyez-les comme messages autonomes pour qu'ils soient enregistrés.

## Configuration (exemple de renommage optionnel)

```
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // garder les clés de groupe séparées
    dmScope: "main", // continuité DM (définir per-channel-peer/per-account-channel-peer pour les boîtes de réception partagées)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Valeurs par défaut : mode=daily, atHour=4 (heure locale de l'hôte de la passerelle).
      // Si vous définissez également idleMinutes, celui qui expire en premier l'emporte.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Inspection

-   `openclaw status` — montre le chemin du magasin et les sessions récentes.
-   `openclaw sessions --json` — exporte chaque entrée (filtrer avec `--active `).
-   `openclaw gateway call sessions.list --params '{}'` — récupère les sessions depuis la passerelle en cours d'exécution (utilisez `--url`/`--token` pour l'accès à une passerelle distante).
-   Envoyez `/status` comme message autonome dans le chat pour voir si l'agent est joignable, combien du contexte de session est utilisé, les bascules de réflexion/verbose actuelles, et quand vos identifiants WhatsApp web ont été actualisés pour la dernière fois (aide à détecter les besoins de re-liaison).
-   Envoyez `/context list` ou `/context detail` pour voir ce qui se trouve dans l'invite système et les fichiers d'espace de travail injectés (et les plus grands contributeurs de contexte).
-   Envoyez `/stop` (ou des phrases d'abandon autonomes comme `stop`, `stop action`, `stop run`, `stop openclaw`) pour abandonner l'exécution en cours, effacer les suivis en file d'attente pour cette session et arrêter toute exécution de sous-agent lancée depuis celle-ci (la réponse inclut le nombre d'arrêts).
-   Envoyez `/compact` (instructions optionnelles) comme message autonome pour résumer le contexte plus ancien et libérer de l'espace dans la fenêtre. Voir [/concepts/compaction](./compaction.md).
-   Les transcriptions JSONL peuvent être ouvertes directement pour revoir les tours complets.

## Conseils

-   Gardez la clé principale dédiée au trafic 1:1 ; laissez les groupes garder leurs propres clés.
-   Lors de l'automatisation du nettoyage, supprimez des clés individuelles au lieu de tout le magasin pour préserver le contexte ailleurs.

## Métadonnées d'origine de session

Chaque entrée de session enregistre d'où elle vient (au mieux) dans `origin` :

-   `label` : étiquette humaine (résolue à partir de l'étiquette de conversation + sujet/groupe/canal)
-   `provider` : identifiant de canal normalisé (incluant les extensions)
-   `from`/`to` : identifiants de routage bruts de l'enveloppe entrante
-   `accountId` : identifiant de compte du fournisseur (en multi-compte)
-   `threadId` : identifiant de fil/sujet lorsque le canal le supporte Les champs d'origine sont remplis pour les messages directs, les canaux et les groupes. Si un connecteur ne met à jour que le routage de livraison (par exemple, pour garder une session DM principale fraîche), il devrait quand même fournir un contexte entrant pour que la session conserve ses métadonnées explicatives. Les extensions peuvent faire cela en envoyant `ConversationLabel`, `GroupSubject`, `GroupChannel`, `GroupSpace` et `SenderName` dans le contexte entrant et en appelant `recordSessionMetaFromInbound` (ou en passant le même contexte à `updateLastRoute`).

[Amorçage](../start/bootstrapping.md)[Élagage des sessions](./session-pruning.md)