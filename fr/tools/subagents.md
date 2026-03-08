

  Coordination d'agents

  
# Sous-Agents

Les sous-agents sont des exécutions d'agents en arrière-plan lancées à partir d'une exécution d'agent existante. Ils s'exécutent dans leur propre session (`agent::subagent:`) et, une fois terminés, **annoncent** leur résultat au canal de discussion du demandeur.

## Commande slash

Utilisez `/subagents` pour inspecter ou contrôler les exécutions de sous-agents pour la **session actuelle** :

-   `/subagents list`
-   `/subagents kill <id|#|all>`
-   `/subagents log <id|#> [limit] [tools]`
-   `/subagents info <id|#>`
-   `/subagents send <id|#> `
-   `/subagents steer <id|#> `
-   `/subagents spawn   [--model ] [--thinking ]`

Contrôles de liaison de thread : Ces commandes fonctionnent sur les canaux qui prennent en charge les liaisons de threads persistantes. Voir **Canaux supportant les threads** ci-dessous.

-   `/focus <subagent-label|session-key|session-id|session-label>`
-   `/unfocus`
-   `/agents`
-   `/session idle <duration|off>`
-   `/session max-age <duration|off>`

`/subagents info` affiche les métadonnées d'exécution (statut, horodatages, id de session, chemin du transcript, nettoyage).

### Comportement de lancement

`/subagents spawn` démarre un sous-agent en arrière-plan en tant que commande utilisateur, et non comme un relais interne, et envoie une seule mise à jour de finalisation au canal de discussion du demandeur lorsque l'exécution se termine.

-   La commande spawn est non bloquante ; elle retourne un identifiant d'exécution immédiatement.
-   À la fin, le sous-agent annonce un message de résumé/résultat au canal de discussion du demandeur.
-   Pour les lancements manuels, la livraison est résiliente :
    -   OpenClaw tente d'abord une livraison directe `agent` avec une clé d'idempotence stable.
    -   Si la livraison directe échoue, il revient à un routage par file d'attente.
    -   Si le routage par file d'attente n'est toujours pas disponible, l'annonce est réessayée avec un backoff exponentiel court avant abandon définitif.
-   Le transfert de finalisation vers la session du demandeur est un contexte interne généré à l'exécution (pas un texte rédigé par l'utilisateur) et inclut :
    -   `Result` (texte de réponse `assistant`, ou dernier `toolResult` si la réponse de l'assistant est vide)
    -   `Status` (`completed successfully` / `failed` / `timed out` / `unknown`)
    -   des statistiques compactes d'exécution/jetons
    -   une instruction de livraison indiquant à l'agent demandeur de reformuler avec une voix d'assistant normale (ne pas transmettre les métadonnées internes brutes)
-   `--model` et `--thinking` remplacent les valeurs par défaut pour cette exécution spécifique.
-   Utilisez `info`/`log` pour inspecter les détails et la sortie après finalisation.
-   `/subagents spawn` est en mode one-shot (`mode: "run"`). Pour les sessions persistantes liées à un thread, utilisez `sessions_spawn` avec `thread: true` et `mode: "session"`.
-   Pour les sessions de harnais ACP (Codex, Claude Code, Gemini CLI), utilisez `sessions_spawn` avec `runtime: "acp"` et consultez [ACP Agents](./acp-agents.md).

Objectifs principaux :

-   Paralléliser le travail de "recherche / tâche longue / outil lent" sans bloquer l'exécution principale.
-   Garder les sous-agents isolés par défaut (séparation de session + sandboxing optionnel).
-   Garder la surface d'outil difficile à mal utiliser : les sous-agents n'obtiennent **pas** les outils de session par défaut.
-   Prendre en charge une profondeur d'imbrication configurable pour les modèles d'orchestrateur.

Note sur les coûts : chaque sous-agent a son **propre** contexte et utilisation de jetons. Pour les tâches lourdes ou répétitives, définissez un modèle moins cher pour les sous-agents et gardez votre agent principal sur un modèle de meilleure qualité. Vous pouvez configurer cela via `agents.defaults.subagents.model` ou des remplacements par agent.

## Outil

Utilisez `sessions_spawn` :

-   Démarre une exécution de sous-agent (`deliver: false`, voie globale : `subagent`)
-   Exécute ensuite une étape d'annonce et publie la réponse d'annonce dans le canal de discussion du demandeur
-   Modèle par défaut : hérite de l'appelant sauf si vous définissez `agents.defaults.subagents.model` (ou par agent `agents.list[].subagents.model`) ; un `sessions_spawn.model` explicite l'emporte toujours.
-   Niveau de réflexion par défaut : hérite de l'appelant sauf si vous définissez `agents.defaults.subagents.thinking` (ou par agent `agents.list[].subagents.thinking`) ; un `sessions_spawn.thinking` explicite l'emporte toujours.
-   Délai d'exécution par défaut : si `sessions_spawn.runTimeoutSeconds` est omis, OpenClaw utilise `agents.defaults.subagents.runTimeoutSeconds` lorsqu'il est défini ; sinon, il revient à `0` (pas de délai).

Paramètres de l'outil :

-   `task` (obligatoire)
-   `label?` (optionnel)
-   `agentId?` (optionnel ; lance sous un autre identifiant d'agent si autorisé)
-   `model?` (optionnel ; remplace le modèle du sous-agent ; les valeurs invalides sont ignorées et le sous-agent s'exécute sur le modèle par défaut avec un avertissement dans le résultat de l'outil)
-   `thinking?` (optionnel ; remplace le niveau de réflexion pour l'exécution du sous-agent)
-   `runTimeoutSeconds?` (par défaut `agents.defaults.subagents.runTimeoutSeconds` lorsqu'il est défini, sinon `0` ; lorsqu'il est défini, l'exécution du sous-agent est interrompue après N secondes)
-   `thread?` (par défaut `false` ; lorsque `true`, demande une liaison de thread de canal pour cette session de sous-agent)
-   `mode?` (`run|session`)
    -   la valeur par défaut est `run`
    -   si `thread: true` et `mode` omis, la valeur par défaut devient `session`
    -   `mode: "session"` nécessite `thread: true`
-   `cleanup?` (`delete|keep`, par défaut `keep`)
-   `sandbox?` (`inherit|require`, par défaut `inherit` ; `require` rejette le lancement sauf si le runtime enfant cible est sandboxé)
-   `sessions_spawn` n'accepte **pas** les paramètres de livraison de canal (`target`, `channel`, `to`, `threadId`, `replyTo`, `transport`). Pour la livraison, utilisez `message`/`sessions_send` depuis l'exécution lancée.

## Sessions liées à un thread

Lorsque les liaisons de threads sont activées pour un canal, un sous-agent peut rester lié à un thread afin que les messages utilisateurs suivants dans ce thread continuent d'être acheminés vers la même session de sous-agent.

### Canaux supportant les threads

-   Discord (actuellement le seul canal supporté) : prend en charge les sessions de sous-agents persistantes liées à un thread (`sessions_spawn` avec `thread: true`), les contrôles manuels de thread (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`), et les clés d'adaptateur `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.idleHours`, `channels.discord.threadBindings.maxAgeHours`, et `channels.discord.threadBindings.spawnSubagentSessions`.

Déroulement rapide :

1.  Lancez avec `sessions_spawn` en utilisant `thread: true` (et optionnellement `mode: "session"`).
2.  OpenClaw crée ou lie un thread à cette session cible dans le canal actif.
3.  Les réponses et messages de suivi dans ce thread sont acheminés vers la session liée.
4.  Utilisez `/session idle` pour inspecter/mettre à jour le détachement automatique en cas d'inactivité et `/session max-age` pour contrôler la limite dure.
5.  Utilisez `/unfocus` pour détacher manuellement.

Contrôles manuels :

-   `/focus ` lie le thread actuel (ou en crée un) à une cible de sous-agent/session.
-   `/unfocus` supprime la liaison pour le thread lié actuel.
-   `/agents` liste les exécutions actives et l'état de liaison (`thread:` ou `unbound`).
-   `/session idle` et `/session max-age` ne fonctionnent que pour les threads liés et focalisés.

Interrupteurs de configuration :

-   Valeur par défaut globale : `session.threadBindings.enabled`, `session.threadBindings.idleHours`, `session.threadBindings.maxAgeHours`
-   Les clés de remplacement par canal et de liaison automatique au lancement sont spécifiques à l'adaptateur. Voir **Canaux supportant les threads** ci-dessus.

Voir [Référence de configuration](../gateway/configuration-reference.md) et [Commandes slash](./slash-commands.md) pour les détails actuels de l'adaptateur. Liste d'autorisation :

-   `agents.list[].subagents.allowAgents` : liste des identifiants d'agents pouvant être ciblés via `agentId` (`["*"]` pour autoriser tous). Par défaut : uniquement l'agent demandeur.
-   Garde d'héritage du sandbox : si la session du demandeur est sandboxée, `sessions_spawn` rejette les cibles qui s'exécuteraient sans sandbox.

Découverte :

-   Utilisez `agents_list` pour voir quels identifiants d'agents sont actuellement autorisés pour `sessions_spawn`.

Archivage automatique :

-   Les sessions de sous-agents sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (par défaut : 60).
-   L'archivage utilise `sessions.delete` et renomme le transcript en `*.deleted.` (même dossier).
-   `cleanup: "delete"` archive immédiatement après l'annonce (conserve toujours le transcript via le renommage).
-   L'archivage automatique est best-effort ; les minuteries en attente sont perdues si la passerelle redémarre.
-   `runTimeoutSeconds` n'archive **pas** automatiquement ; il arrête seulement l'exécution. La session reste jusqu'à l'archivage automatique.
-   L'archivage automatique s'applique également aux sessions de profondeur 1 et 2.

## Sous-Agents Imbriqués

Par défaut, les sous-agents ne peuvent pas lancer leurs propres sous-agents (`maxSpawnDepth: 1`). Vous pouvez activer un niveau d'imbrication en définissant `maxSpawnDepth: 2`, ce qui permet le **modèle d'orchestrateur** : principal → sous-agent orchestrateur → sous-sous-agents travailleurs.

### Comment activer

```json
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // autoriser les sous-agents à lancer des enfants (par défaut : 1)
        maxChildrenPerAgent: 5, // nombre maximum d'enfants actifs par session d'agent (par défaut : 5)
        maxConcurrent: 8, // limite globale de concurrence de voie (par défaut : 8)
        runTimeoutSeconds: 900, // délai par défaut pour sessions_spawn lorsqu'omis (0 = pas de délai)
      },
    },
  },
}
```

### Niveaux de profondeur

| Profondeur | Forme de la clé de session | Rôle | Peut lancer ? |
| --- | --- | --- | --- |
| 0 | `agent::main` | Agent principal | Toujours |
| 1 | `agent::subagent:` | Sous-agent (orchestrateur lorsque la profondeur 2 est autorisée) | Seulement si `maxSpawnDepth >= 2` |
| 2 | `agent::subagent::subagent:` | Sous-sous-agent (travailleur feuille) | Jamais |

### Chaîne d'annonce

Les résultats remontent la chaîne :

1.  Le travailleur de profondeur 2 se termine → annonce à son parent (orchestrateur de profondeur 1)
2.  L'orchestrateur de profondeur 1 reçoit l'annonce, synthétise les résultats, se termine → annonce au principal
3.  L'agent principal reçoit l'annonce et la livre à l'utilisateur

Chaque niveau ne voit que les annonces de ses enfants directs.

### Politique d'outils par profondeur

-   **Profondeur 1 (orchestrateur, lorsque `maxSpawnDepth >= 2`)** : Obtient `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` pour pouvoir gérer ses enfants. Les autres outils de session/système restent refusés.
-   **Profondeur 1 (feuille, lorsque `maxSpawnDepth == 1`)** : Aucun outil de session (comportement par défaut actuel).
-   **Profondeur 2 (travailleur feuille)** : Aucun outil de session — `sessions_spawn` est toujours refusé à la profondeur 2. Ne peut pas lancer d'autres enfants.

### Limite de lancement par agent

Chaque session d'agent (à n'importe quelle profondeur) peut avoir au maximum `maxChildrenPerAgent` (par défaut : 5) enfants actifs en même temps. Cela empêche une prolifération incontrôlée à partir d'un seul orchestrateur.

### Arrêt en cascade

Arrêter un orchestrateur de profondeur 1 arrête automatiquement tous ses enfants de profondeur 2 :

-   `/stop` dans le chat principal arrête tous les agents de profondeur 1 et se propage à leurs enfants de profondeur 2.
-   `/subagents kill ` arrête un sous-agent spécifique et se propage à ses enfants.
-   `/subagents kill all` arrête tous les sous-agents du demandeur et se propage.

## Authentification

L'authentification des sous-agents est résolue par **identifiant d'agent**, et non par type de session :

-   La clé de session du sous-agent est `agent::subagent:`.
-   Le magasin d'authentification est chargé depuis le `agentDir` de cet agent.
-   Les profils d'authentification de l'agent principal sont fusionnés en tant que **secours** ; les profils de l'agent écrasent les profils principaux en cas de conflit.

Note : la fusion est additive, donc les profils principaux sont toujours disponibles comme secours. Une authentification complètement isolée par agent n'est pas encore supportée.

## Annonce

Les sous-agents rapportent via une étape d'annonce :

-   L'étape d'annonce s'exécute à l'intérieur de la session du sous-agent (pas de la session du demandeur).
-   Si le sous-agent répond exactement `ANNOUNCE_SKIP`, rien n'est publié.
-   Sinon, la livraison dépend de la profondeur du demandeur :
    -   les sessions demandeurs de niveau supérieur utilisent un appel `agent` de suivi avec livraison externe (`deliver=true`)
    -   les sessions demandeurs de sous-agents imbriquées reçoivent une injection interne de suivi (`deliver=false`) pour que l'orchestrateur puisse synthétiser les résultats des enfants en session
    -   si une session demandeur de sous-agent imbriquée a disparu, OpenClaw revient au demandeur de cette session lorsqu'il est disponible
-   L'agrégation des finalisations d'enfants est limitée à l'exécution demandeur actuelle lors de la construction des résultats de finalisation imbriqués, empêchant les sorties d'enfants d'exécutions précédentes obsolètes de fuiter dans l'annonce actuelle.
-   Les réponses d'annonce préservent le routage thread/sujet lorsqu'il est disponible sur les adaptateurs de canal.
-   Le contexte d'annonce est normalisé en un bloc d'événement interne stable :
    -   source (`subagent` ou `cron`)
    -   clé/id de session enfant
    -   type d'annonce + libellé de tâche
    -   ligne de statut dérivée du résultat d'exécution (`success`, `error`, `timeout`, ou `unknown`)
    -   contenu du résultat de l'étape d'annonce (ou `(no output)` si manquant)
    -   une instruction de suivi décrivant quand répondre vs rester silencieux
-   `Status` n'est pas déduit de la sortie du modèle ; il provient des signaux de résultat d'exécution.

Les charges utiles d'annonce incluent une ligne de statistiques à la fin (même lorsqu'elles sont encapsulées) :

-   Temps d'exécution (ex. `runtime 5m12s`)
-   Utilisation de jetons (entrée/sortie/total)
-   Coût estimé lorsque la tarification des modèles est configurée (`models.providers.*.models[].cost`)
-   `sessionKey`, `sessionId`, et chemin du transcript (pour que l'agent principal puisse récupérer l'historique via `sessions_history` ou inspecter le fichier sur disque)
-   Les métadonnées internes sont destinées uniquement à l'orchestration ; les réponses destinées à l'utilisateur doivent être reformulées avec une voix d'assistant normale.

## Politique d'Outils (outils de sous-agents)

Par défaut, les sous-agents obtiennent **tous les outils sauf les outils de session** et les outils système :

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

Lorsque `maxSpawnDepth >= 2`, les sous-agents orchestrateurs de profondeur 1 reçoivent en plus `sessions_spawn`, `subagents`, `sessions_list`, et `sessions_history` pour pouvoir gérer leurs enfants. Remplacez via la configuration :

```json
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny l'emporte
        deny: ["gateway", "cron"],
        // si allow est défini, cela devient allow-only (deny l'emporte toujours)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrence

Les sous-agents utilisent une voie de file d'attente en processus dédiée :

-   Nom de la voie : `subagent`
-   Concurrence : `agents.defaults.subagents.maxConcurrent` (par défaut `8`)

## Arrêt

-   Envoyer `/stop` dans le chat du demandeur interrompt la session du demandeur et arrête toutes les exécutions de sous-agents actives lancées depuis celle-ci, en cascade vers les enfants imbriqués.
-   `/subagents kill ` arrête un sous-agent spécifique et se propage à ses enfants.

## Limitations

-   L'annonce de sous-agent est **best-effort**. Si la passerelle redémarre, le travail d'"annonce de retour" en attente est perdu.
-   Les sous-agents partagent toujours les mêmes ressources de processus de passerelle ; traitez `maxConcurrent` comme une soupape de sécurité.
-   `sessions_spawn` est toujours non bloquant : il retourne `{ status: "accepted", runId, childSessionKey }` immédiatement.
-   Le contexte du sous-agent n'injecte que `AGENTS.md` + `TOOLS.md` (pas `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, ou `BOOTSTRAP.md`).
-   La profondeur d'imbrication maximale est de 5 (`maxSpawnDepth` plage : 1–5). La profondeur 2 est recommandée pour la plupart des cas d'utilisation.
-   `maxChildrenPerAgent` limite les enfants actifs par session (par défaut : 5, plage : 1–20).

[Agent Send](./agent-send.md)[ACP Agents](./acp-agents.md)