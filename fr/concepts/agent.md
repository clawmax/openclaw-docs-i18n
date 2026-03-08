

  Fondamentaux

  
# Environnement d'exécution de l'agent

OpenClaw exécute un seul environnement d'exécution d'agent embarqué dérivé de **pi-mono**.

## Espace de travail (obligatoire)

OpenClaw utilise un seul répertoire d'espace de travail pour l'agent (`agents.defaults.workspace`) comme répertoire de travail **unique** (`cwd`) de l'agent pour les outils et le contexte. Recommandé : utilisez `openclaw setup` pour créer `~/.openclaw/openclaw.json` s'il est manquant et initialiser les fichiers de l'espace de travail. Disposition complète de l'espace de travail + guide de sauvegarde : [Espace de travail de l'agent](./agent-workspace.md) Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent remplacer ceci par des espaces de travail par session sous `agents.defaults.sandbox.workspaceRoot` (voir [Configuration de la passerelle](../gateway/configuration.md)).

## Fichiers d'amorçage (injectés)

À l'intérieur de `agents.defaults.workspace`, OpenClaw attend ces fichiers modifiables par l'utilisateur :

-   `AGENTS.md` — instructions de fonctionnement + « mémoire »
-   `SOUL.md` — persona, limites, ton
-   `TOOLS.md` — notes d'outils maintenues par l'utilisateur (par ex. `imsg`, `sag`, conventions)
-   `BOOTSTRAP.md` — rituel de premier lancement unique (supprimé après achèvement)
-   `IDENTITY.md` — nom/vibe/emoji de l'agent
-   `USER.md` — profil utilisateur + adresse préférée

Lors du premier tour d'une nouvelle session, OpenClaw injecte le contenu de ces fichiers directement dans le contexte de l'agent. Les fichiers vides sont ignorés. Les fichiers volumineux sont tronqués et coupés avec un marqueur pour que les invites restent légères (lisez le fichier pour le contenu complet). Si un fichier est manquant, OpenClaw injecte une seule ligne de marqueur « fichier manquant » (et `openclaw setup` créera un modèle par défaut sûr). `BOOTSTRAP.md` n'est créé que pour un **espace de travail entièrement nouveau** (aucun autre fichier d'amorçage présent). Si vous le supprimez après avoir terminé le rituel, il ne devrait pas être recréé lors des redémarrages ultérieurs. Pour désactiver entièrement la création de fichiers d'amorçage (pour les espaces de travail pré-ensemencés), définissez :

```json
{ agent: { skipBootstrap: true } }
```

## Outils intégrés

Les outils de base (read/exec/edit/write et les outils système associés) sont toujours disponibles, sous réserve de la politique des outils. `apply_patch` est optionnel et contrôlé par `tools.exec.applyPatch`. `TOOLS.md` ne **contrôle pas** quels outils existent ; c'est un guide pour la manière dont *vous* souhaitez qu'ils soient utilisés.

## Compétences

OpenClaw charge les compétences depuis trois emplacements (l'espace de travail l'emporte en cas de conflit de nom) :

-   Empaquetées (fournies avec l'installation)
-   Gérées/locales : `~/.openclaw/skills`
-   Espace de travail : `/skills`

Les compétences peuvent être contrôlées par la configuration/l'environnement (voir `skills` dans [Configuration de la passerelle](../gateway/configuration.md)).

## Intégration pi-mono

OpenClaw réutilise des parties de la base de code pi-mono (modèles/outils), mais **la gestion des sessions, la découverte et le câblage des outils sont gérés par OpenClaw**.

-   Aucun environnement d'exécution d'agent pi-coding.
-   Aucun paramètre `~/.pi/agent` ou `/.pi` n'est consulté.

## Sessions

Les transcriptions des sessions sont stockées au format JSONL à l'emplacement :

-   `~/.openclaw/agents//sessions/.jsonl`

L'ID de session est stable et choisi par OpenClaw. Les dossiers de session Pi/Tau hérités ne sont **pas** lus.

## Pilotage pendant la diffusion en continu

Lorsque le mode de file d'attente est `steer`, les messages entrants sont injectés dans l'exécution en cours. La file d'attente est vérifiée **après chaque appel d'outil** ; si un message en file d'attente est présent, les appels d'outils restants du message d'assistant actuel sont ignorés (les résultats des outils d'erreur indiquent « Skipped due to queued user message. »), puis le message utilisateur en file d'attente est injecté avant la prochaine réponse de l'assistant. Lorsque le mode de file d'attente est `followup` ou `collect`, les messages entrants sont conservés jusqu'à la fin du tour actuel, puis un nouveau tour d'agent commence avec les charges utiles en file d'attente. Voir [File d'attente](./queue.md) pour le comportement des modes + anti-rebond/limite. La diffusion en continu par blocs envoie les blocs d'assistant terminés dès qu'ils se terminent ; elle est **désactivée par défaut** (`agents.defaults.blockStreamingDefault: "off"`). Ajustez la limite via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end` ; par défaut text_end). Contrôlez le découpage en blocs logiciels avec `agents.defaults.blockStreamingChunk` (par défaut 800–1200 caractères ; préfère les sauts de paragraphe, puis les nouvelles lignes ; les phrases en dernier). Fusionnez les morceaux diffusés avec `agents.defaults.blockStreamingCoalesce` pour réduire le spam de lignes uniques (fusion basée sur l'inactivité avant envoi). Les canaux non-Telegram nécessitent un `*.blockStreaming: true` explicite pour activer les réponses par blocs. Les résumés d'outils verbeux sont émis au début de l'outil (pas d'anti-rebond) ; L'interface de contrôle diffuse la sortie de l'outil via les événements de l'agent lorsque disponible. Plus de détails : [Diffusion en continu + découpage](./streaming.md).

## Références de modèles

Les références de modèles dans la configuration (par exemple `agents.defaults.model` et `agents.defaults.models`) sont analysées en divisant sur la **première** `/`.

-   Utilisez `fournisseur/modèle` lors de la configuration des modèles.
-   Si l'ID du modèle contient lui-même `/` (style OpenRouter), incluez le préfixe du fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
-   Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement lorsqu'il n'y a pas de `/` dans l'ID du modèle).

## Configuration (minimale)

Au minimum, définissez :

-   `agents.defaults.workspace`
-   `channels.whatsapp.allowFrom` (fortement recommandé)

* * *

*Suivant : [Chats de groupe](../channels/group-messages.md)* 🦞

[Architecture de la passerelle](./architecture.md)[Boucle de l'agent](./agent-loop.md)