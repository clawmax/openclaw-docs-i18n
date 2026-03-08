

  Fondamentaux

  
# Prompt système

OpenClaw construit un prompt système personnalisé pour chaque exécution d'agent. Le prompt est **propriété d'OpenClaw** et n'utilise pas le prompt par défaut de pi-coding-agent. Le prompt est assemblé par OpenClaw et injecté dans chaque exécution d'agent.

## Structure

Le prompt est intentionnellement compact et utilise des sections fixes :

-   **Outillage** : liste d'outils actuels + courtes descriptions.
-   **Sécurité** : court rappel des garde-fous pour éviter les comportements de recherche de pouvoir ou de contournement de la supervision.
-   **Compétences** (lorsque disponibles) : indique au modèle comment charger les instructions des compétences à la demande.
-   **Auto-mise à jour OpenClaw** : comment exécuter `config.apply` et `update.run`.
-   **Espace de travail** : répertoire de travail (`agents.defaults.workspace`).
-   **Documentation** : chemin local vers la documentation OpenClaw (dépôt ou package npm) et quand la lire.
-   **Fichiers de l'espace de travail (injectés)** : indique que les fichiers d'amorçage sont inclus ci-dessous.
-   **Sandbox** (lorsqu'activé) : indique un environnement d'exécution isolé, les chemins du sandbox, et si l'exécution élevée est disponible.
-   **Date et heure actuelles** : heure locale de l'utilisateur, fuseau horaire et format d'heure.
-   **Balises de réponse** : syntaxe de balises de réponse optionnelle pour les fournisseurs pris en charge.
-   **Heartbeats** : prompt de heartbeat et comportement d'accusé de réception.
-   **Runtime** : hôte, OS, node, modèle, racine du dépôt (lorsque détectée), niveau de réflexion (une ligne).
-   **Raisonnement** : niveau de visibilité actuel + indice de bascule /reasoning.

Les garde-fous de sécurité dans le prompt système sont consultatifs. Ils guident le comportement du modèle mais n'appliquent pas de politique. Utilisez la politique des outils, les approbations d'exécution, le sandboxing et les listes d'autorisation de canaux pour une application stricte ; les opérateurs peuvent désactiver ces éléments par conception.

## Modes de prompt

OpenClaw peut générer des prompts système plus petits pour les sous-agents. Le runtime définit un `promptMode` pour chaque exécution (ce n'est pas une configuration visible par l'utilisateur) :

-   `full` (par défaut) : inclut toutes les sections ci-dessus.
-   `minimal` : utilisé pour les sous-agents ; omet **Compétences**, **Rappel de mémoire**, **Auto-mise à jour OpenClaw**, **Alias de modèle**, **Identité utilisateur**, **Balises de réponse**, **Messagerie**, **Réponses silencieuses** et **Heartbeats**. L'outillage, la **Sécurité**, l'Espace de travail, le Sandbox, la Date et heure actuelles (lorsque connues), le Runtime et le contexte injecté restent disponibles.
-   `none` : retourne uniquement la ligne d'identité de base.

Lorsque `promptMode=minimal`, les prompts injectés supplémentaires sont étiquetés **Contexte du sous-agent** au lieu de **Contexte du chat de groupe**.

## Injection de l'amorçage de l'espace de travail

Les fichiers d'amorçage sont tronqués et ajoutés sous **Contexte du projet** afin que le modèle voie le contexte d'identité et de profil sans avoir besoin de lectures explicites :

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md` (uniquement sur les espaces de travail tout nouveaux)
-   `MEMORY.md` et/ou `memory.md` (lorsqu'ils sont présents dans l'espace de travail ; l'un ou l'autre, ou les deux, peuvent être injectés)

Tous ces fichiers sont **injectés dans la fenêtre de contexte** à chaque tour, ce qui signifie qu'ils consomment des tokens. Gardez-les concis — en particulier `MEMORY.md`, qui peut grossir avec le temps et entraîner une utilisation de contexte inattendue et une compaction plus fréquente.

> **Note :** Les fichiers journaliers `memory/*.md` ne sont **pas** injectés automatiquement. Ils sont consultés à la demande via les outils `memory_search` et `memory_get`, donc ils ne comptent pas dans la fenêtre de contexte à moins que le modèle ne les lise explicitement.

Les fichiers volumineux sont tronqués avec un marqueur. La taille maximale par fichier est contrôlée par `agents.defaults.bootstrapMaxChars` (par défaut : 20000). Le contenu total injecté d'amorçage à travers les fichiers est plafonné par `agents.defaults.bootstrapTotalMaxChars` (par défaut : 150000). Les fichiers manquants injectent un court marqueur de fichier manquant. Lorsqu'une troncature se produit, OpenClaw peut injecter un bloc d'avertissement dans le Contexte du projet ; contrôlez cela avec `agents.defaults.bootstrapPromptTruncationWarning` (`off`, `once`, `always` ; par défaut : `once`). Les sessions de sous-agent n'injectent que `AGENTS.md` et `TOOLS.md` (les autres fichiers d'amorçage sont filtrés pour garder le contexte du sous-agent petit). Des hooks internes peuvent intercepter cette étape via `agent:bootstrap` pour modifier ou remplacer les fichiers d'amorçage injectés (par exemple, remplacer `SOUL.md` par une autre persona). Pour inspecter la contribution de chaque fichier injecté (brut vs injecté, troncature, plus la surcharge du schéma d'outil), utilisez `/context list` ou `/context detail`. Voir [Contexte](./context.md).

## Gestion du temps

Le prompt système inclut une section dédiée **Date et heure actuelles** lorsque le fuseau horaire de l'utilisateur est connu. Pour garder le prompt stable en cache, il n'inclut désormais que le **fuseau horaire** (pas d'horloge dynamique ou de format d'heure). Utilisez `session_status` lorsque l'agent a besoin de l'heure actuelle ; la carte de statut inclut une ligne d'horodatage. Configurez avec :

-   `agents.defaults.userTimezone`
-   `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Voir [Date & Heure](../date-time.md) pour les détails complets du comportement.

## Compétences

Lorsque des compétences éligibles existent, OpenClaw injecte une **liste compacte de compétences disponibles** (`formatSkillsForPrompt`) qui inclut le **chemin du fichier** pour chaque compétence. Le prompt indique au modèle d'utiliser `read` pour charger le fichier SKILL.md à l'emplacement listé (espace de travail, géré ou intégré). Si aucune compétence n'est éligible, la section Compétences est omise.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Cela permet de garder le prompt de base petit tout en permettant une utilisation ciblée des compétences.

## Documentation

Lorsqu'elle est disponible, le prompt système inclut une section **Documentation** qui pointe vers le répertoire de documentation OpenClaw local (soit `docs/` dans l'espace de travail du dépôt, soit la documentation du package npm intégré) et note également le miroir public, le dépôt source, le Discord communautaire et ClawHub ([https://clawhub.com](https://clawhub.com)) pour la découverte de compétences. Le prompt indique au modèle de consulter d'abord la documentation locale pour le comportement, les commandes, la configuration ou l'architecture d'OpenClaw, et d'exécuter lui-même `openclaw status` lorsque possible (en demandant à l'utilisateur uniquement lorsqu'il n'a pas accès).

[Boucle d'agent](./agent-loop.md)[Contexte](./context.md)