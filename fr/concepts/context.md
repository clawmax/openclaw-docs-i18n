

  Fondamentaux

  
# Contexte

Le « Contexte » est **tout ce qu'OpenClaw envoie au modèle pour une exécution**. Il est limité par la **fenêtre de contexte** (limite de tokens) du modèle. Modèle mental pour débutants :

-   **Prompt système** (construit par OpenClaw) : règles, outils, liste de compétences, heure/exécution, et fichiers de l'espace de travail injectés.
-   **Historique de la conversation** : vos messages + les messages de l'assistant pour cette session.
-   **Appels/résultats d'outils + pièces jointes** : sortie de commande, lectures de fichiers, images/audio, etc.

Le contexte *n'est pas la même chose* que la « mémoire » : la mémoire peut être stockée sur disque et rechargée plus tard ; le contexte est ce qui se trouve dans la fenêtre actuelle du modèle.

## Démarrage rapide (inspecter le contexte)

-   `/status` → vue rapide « à quel point ma fenêtre est-elle pleine ? » + paramètres de session.
-   `/context list` → ce qui est injecté + tailles approximatives (par fichier + totaux).
-   `/context detail` → analyse plus détaillée : par fichier, tailles des schémas d'outils, tailles des entrées de compétences, et taille du prompt système.
-   `/usage tokens` → ajoute un pied de page d'utilisation par réponse aux réponses normales.
-   `/compact` → résume l'historique ancien en une entrée compacte pour libérer de l'espace dans la fenêtre.

Voir aussi : [Commandes slash](../tools/slash-commands.md), [Utilisation et coût des tokens](../reference/token-use.md), [Compaction](./compaction.md).

## Exemple de sortie

Les valeurs varient selon le modèle, le fournisseur, la politique d'outils et le contenu de votre espace de travail.

### /context list

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### /context detail

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Ce qui compte dans la fenêtre de contexte

Tout ce que le modèle reçoit compte, y compris :

-   Le prompt système (toutes les sections).
-   L'historique de la conversation.
-   Les appels d'outils + les résultats d'outils.
-   Les pièces jointes/transcriptions (images/audio/fichiers).
-   Les résumés de compaction et les artefacts d'élagage.
-   Les « en-têtes » ou « wrappers » du fournisseur (non visibles, mais comptabilisés).

## Comment OpenClaw construit le prompt système

Le prompt système est **propriété d'OpenClaw** et est reconstruit à chaque exécution. Il inclut :

-   La liste des outils + leurs courtes descriptions.
-   La liste des compétences (métadonnées uniquement ; voir ci-dessous).
-   L'emplacement de l'espace de travail.
-   L'heure (UTC + heure utilisateur convertie si configurée).
-   Les métadonnées d'exécution (hôte/OS/modèle/réflexion).
-   Les fichiers de bootstrap de l'espace de travail injectés sous **Contexte du Projet**.

Analyse complète : [Prompt système](./system-prompt.md).

## Fichiers de l'espace de travail injectés (Contexte du Projet)

Par défaut, OpenClaw injecte un ensemble fixe de fichiers de l'espace de travail (s'ils sont présents) :

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md` (première exécution uniquement)

Les fichiers volumineux sont tronqués par fichier en utilisant `agents.defaults.bootstrapMaxChars` (par défaut `20000` caractères). OpenClaw applique également un plafond total d'injection de bootstrap sur l'ensemble des fichiers avec `agents.defaults.bootstrapTotalMaxChars` (par défaut `150000` caractères). `/context` montre les tailles **brutes vs injectées** et si une troncature a eu lieu. Lorsqu'une troncature se produit, l'exécution peut injecter un bloc d'avertissement dans le prompt sous Contexte du Projet. Configurez ceci avec `agents.defaults.bootstrapPromptTruncationWarning` (`off`, `once`, `always` ; par défaut `once`).

## Compétences : ce qui est injecté vs chargé à la demande

Le prompt système inclut une **liste de compétences** compacte (nom + description + emplacement). Cette liste a un coût réel. Les instructions des compétences ne sont *pas* incluses par défaut. Le modèle est censé `read` le `SKILL.md` de la compétence **uniquement quand c'est nécessaire**.

## Outils : il y a deux coûts

Les outils affectent le contexte de deux manières :

1.  **Le texte de la liste d'outils** dans le prompt système (ce que vous voyez comme « Outillage »).
2.  **Les schémas d'outils** (JSON). Ils sont envoyés au modèle pour qu'il puisse appeler les outils. Ils comptent dans le contexte même si vous ne les voyez pas sous forme de texte brut.

`/context detail` décompose les plus gros schémas d'outils pour que vous puissiez voir ce qui domine.

## Commandes, directives et « raccourcis en ligne »

Les commandes slash sont gérées par la Gateway. Il existe quelques comportements différents :

-   **Commandes autonomes** : un message qui est uniquement `/...` s'exécute comme une commande.
-   **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` sont supprimés avant que le modèle ne voie le message.
    -   Les messages contenant uniquement une directive conservent les paramètres de session.
    -   Les directives en ligne dans un message normal agissent comme des indications par message.
-   **Raccourcis en ligne** (expéditeurs autorisés uniquement) : certains tokens `/...` à l'intérieur d'un message normal peuvent s'exécuter immédiatement (exemple : « hey /status »), et sont supprimés avant que le modèle ne voie le texte restant.

Détails : [Commandes slash](../tools/slash-commands.md).

## Sessions, compaction et élagage (ce qui persiste)

Ce qui persiste entre les messages dépend du mécanisme :

-   **L'historique normal** persiste dans la transcription de la session jusqu'à ce qu'il soit compacté/élagué par la politique.
-   **La compaction** persiste un résumé dans la transcription et garde les messages récents intacts.
-   **L'élagage** supprime les anciens résultats d'outils du prompt *en mémoire* pour une exécution, mais ne réécrit pas la transcription.

Documentation : [Session](./session.md), [Compaction](./compaction.md), [Élagage de session](./session-pruning.md). Par défaut, OpenClaw utilise le moteur de contexte intégré `legacy` pour l'assemblage et la compaction. Si vous installez un plugin qui fournit `kind: "context-engine"` et que vous le sélectionnez avec `plugins.slots.contextEngine`, OpenClaw délègue l'assemblage du contexte, `/compact`, et les hooks de cycle de vie du contexte des sous-agents associés à ce moteur à la place.

## Ce que /context rapporte réellement

`/context` préfère le rapport de prompt système **construit par la dernière exécution** lorsqu'il est disponible :

-   `System prompt (run)` = capturé depuis la dernière exécution intégrée (avec outils) et conservé dans le stockage de session.
-   `System prompt (estimate)` = calculé à la volée lorsqu'aucun rapport d'exécution n'existe (ou lors de l'exécution via un backend CLI qui ne génère pas le rapport).

Dans les deux cas, il rapporte les tailles et les principaux contributeurs ; il **ne** déverse **pas** le prompt système complet ou les schémas d'outils.

[Prompt système](./system-prompt.md)[Espace de travail de l'agent](./agent-workspace.md)