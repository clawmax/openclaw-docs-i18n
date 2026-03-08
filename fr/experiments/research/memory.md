

  Expériences

  
# Recherche sur la mémoire de l'espace de travail

Cible : Espace de travail de type Clawd (`agents.defaults.workspace`, par défaut `~/.openclaw/workspace`) où la « mémoire » est stockée sous forme d'un fichier Markdown par jour (`memory/YYYY-MM-DD.md`) plus un petit ensemble de fichiers stables (ex. `memory.md`, `SOUL.md`). Ce document propose une architecture de mémoire **offline-first** qui conserve Markdown comme source de vérité canonique et révisable, mais ajoute un **rappel structuré** (recherche, résumés d'entités, mises à jour de confiance) via un index dérivé.

## Pourquoi changer ?

La configuration actuelle (un fichier par jour) est excellente pour :

-   la journalisation « en ajout uniquement »
-   l'édition humaine
-   la durabilité + auditabilité avec git
-   la capture à faible friction (« il suffit de l'écrire »)

Elle est faible pour :

-   la récupération à haut rappel (« qu'avons-nous décidé à propos de X ? », « la dernière fois que nous avons essayé Y ? »)
-   les réponses centrées sur les entités (« parle-moi d'Alice / Le Château / warelay ») sans relire de nombreux fichiers
-   la stabilité des opinions/préférences (et les preuves lorsqu'elles changent)
-   les contraintes temporelles (« qu'est-ce qui était vrai en novembre 2025 ? ») et la résolution des conflits

## Objectifs de conception

-   **Hors ligne** : fonctionne sans réseau ; peut tourner sur un ordinateur portable/Château ; aucune dépendance au cloud.
-   **Explicable** : les éléments récupérés doivent être attribuables (fichier + emplacement) et séparables de l'inférence.
-   **Faible cérémonie** : la journalisation quotidienne reste en Markdown, pas de travail de schéma lourd.
-   **Incrémentiel** : la v1 est utile avec seulement la FTS ; les améliorations sémantiques/vectorielles et les graphes sont optionnelles.
-   **Adapté aux agents** : facilite le « rappel dans les limites de tokens » (retourne de petits paquets de faits).

## Modèle étoile du Nord (Hindsight × Letta)

Deux pièces à combiner :

1.  **Boucle de contrôle de type Letta/MemGPT**

-   garder un petit « noyau » toujours en contexte (persona + faits clés sur l'utilisateur)
-   tout le reste est hors contexte et récupéré via des outils
-   les écritures en mémoire sont des appels d'outils explicites (ajouter/remplacer/insérer), persistées, puis réinjectées au tour suivant

2.  **Substrat mémoire de type Hindsight**

-   séparer ce qui est observé de ce qui est cru de ce qui est résumé
-   supporter retenir/rappeler/réfléchir
-   opinions avec confiance qui peuvent évoluer avec les preuves
-   récupération sensible aux entités + requêtes temporelles (même sans graphes de connaissances complets)

## Architecture proposée (Source de vérité Markdown + index dérivé)

### Stockage canonique (compatible git)

Conserver `~/.openclaw/workspace` comme mémoire canonique lisible par l'humain. Organisation suggérée de l'espace de travail :

```
~/.openclaw/workspace/
  memory.md                    # petit : faits durables + préférences (type noyau)
  memory/
    YYYY-MM-DD.md              # journal quotidien (ajout ; narratif)
  bank/                        # pages de mémoire « typées » (stables, révisables)
    world.md                   # faits objectifs sur le monde
    experience.md              # ce que l'agent a fait (à la première personne)
    opinions.md                # préférences/jugements subjectifs + confiance + pointeurs de preuve
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes :

-   **Le journal quotidien reste un journal quotidien**. Pas besoin de le transformer en JSON.
-   Les fichiers `bank/` sont **curatés**, produits par des tâches de réflexion, et peuvent toujours être édités à la main.
-   `memory.md` reste « petit + type noyau » : les choses que vous voulez que Clawd voie à chaque session.

### Stockage dérivé (rappel machine)

Ajouter un index dérivé sous l'espace de travail (pas nécessairement suivi par git) :

```
~/.openclaw/workspace/.memory/index.sqlite
```

Le supporter avec :

-   un schéma SQLite pour les faits + liens d'entités + métadonnées d'opinion
-   **FTS5** de SQLite pour le rappel lexical (rapide, minuscule, hors ligne)
-   une table d'embeddings optionnelle pour le rappel sémantique (toujours hors ligne)

L'index est toujours **reconstructible à partir du Markdown**.

## Retenir / Rappeler / Réfléchir (boucle opérationnelle)

### Retenir : normaliser les journaux quotidiens en « faits »

L'idée clé de Hindsight qui importe ici : stocker des **faits narratifs, autonomes**, pas de minuscules extraits. Règle pratique pour `memory/YYYY-MM-DD.md` :

-   à la fin de la journée (ou pendant), ajouter une section `## Retenir` avec 2 à 5 puces qui sont :
    -   narratives (le contexte inter-tours est préservé)
    -   autonomes (ont du sens seules plus tard)
    -   étiquetées avec un type + mentions d'entités

Exemple :

```markdown
## Retenir
- W @Peter : Actuellement à Marrakech (27 nov – 1er déc 2025) pour l'anniversaire d'Andy.
- B @warelay : J'ai corrigé le crash WS de Baileys en encapsulant les gestionnaires connection.update dans try/catch (voir memory/2025-11-27.md).
- O(c=0.95) @Peter : Préfère les réponses concises (<1500 caractères) sur WhatsApp ; les contenus longs vont dans des fichiers.
```

Analyse minimale :

-   Préfixe de type : `W` (monde), `B` (expérience/biographique), `O` (opinion), `S` (observation/résumé ; généralement généré)
-   Entités : `@Peter`, `@warelay`, etc. (les slugs correspondent à `bank/entities/*.md`)
-   Confiance de l'opinion : `O(c=0.0..1.0)` optionnel

Si vous ne voulez pas que les auteurs y pensent : la tâche de réflexion peut inférer ces puces à partir du reste du journal, mais avoir une section `## Retenir` explicite est le « levier de qualité » le plus simple.

### Rappeler : requêtes sur l'index dérivé

Le rappel doit supporter :

-   **lexical** : « trouver des termes/noms/commandes exacts » (FTS5)
-   **entité** : « parle-moi de X » (pages d'entité + faits liés aux entités)
-   **temporel** : « que s'est-il passé autour du 27 novembre » / « depuis la semaine dernière »
-   **opinion** : « que préfère Peter ? » (avec confiance + preuve)

Le format de retour doit être adapté aux agents et citer les sources :

-   `kind` (`world|experience|opinion|observation`)
-   `timestamp` (jour source, ou plage de temps extraite si présente)
-   `entities` (`["Peter","warelay"]`)
-   `content` (le fait narratif)
-   `source` (`memory/2025-11-27.md#L12` etc.)

### Réfléchir : produire des pages stables + mettre à jour les croyances

La réflexion est une tâche planifiée (quotidienne ou battement `ultrathink`) qui :

-   met à jour `bank/entities/*.md` à partir des faits récents (résumés d'entités)
-   met à jour la confiance dans `bank/opinions.md` basée sur le renforcement/la contradiction
-   propose optionnellement des modifications à `memory.md` (faits durables « type noyau »)

Évolution de l'opinion (simple, explicable) :

-   chaque opinion a :
    -   un énoncé
    -   une confiance `c ∈ [0,1]`
    -   une date de dernière mise à jour
    -   des liens de preuve (IDs de faits supportant + contredisant)
-   quand de nouveaux faits arrivent :
    -   trouver des opinions candidates par chevauchement d'entités + similarité (FTS d'abord, embeddings plus tard)
    -   mettre à jour la confiance par de petits deltas ; les grands sauts nécessitent une forte contradiction + des preuves répétées

## Intégration CLI : autonome vs intégration profonde

Recommandation : **intégration profonde dans OpenClaw**, mais garder une bibliothèque cœur séparable.

### Pourquoi intégrer dans OpenClaw ?

-   OpenClaw connaît déjà :
    -   le chemin de l'espace de travail (`agents.defaults.workspace`)
    -   le modèle de session + les battements de cœur
    -   les modèles de journalisation + dépannage
-   Vous voulez que l'agent lui-même appelle les outils :
    -   `openclaw memory recall "…" --k 25 --since 30d`
    -   `openclaw memory reflect --since 7d`

### Pourquoi toujours séparer une bibliothèque ?

-   garder la logique mémoire testable sans passerelle/runtime
-   réutilisation dans d'autres contextes (scripts locaux, future application de bureau, etc.)

Forme : L'outillage mémoire est destiné à être une petite couche CLI + bibliothèque, mais ceci est uniquement exploratoire.

## « S-Collide » / SuCo : quand l'utiliser (recherche)

Si « S-Collide » fait référence à **SuCo (Subspace Collision)** : c'est une approche de récupération ANN qui cible des compromis rappel/latence forts en utilisant des collisions apprises/structurées dans des sous-espaces (article : arXiv 2411.14754, 2024). Prise pragmatique pour `~/.openclaw/workspace` :

-   **ne pas commencer** avec SuCo.
-   commencer avec SQLite FTS + (optionnel) des embeddings simples ; vous obtiendrez la plupart des gains d'UX immédiatement.
-   considérer les solutions de classe SuCo/HNSW/ScaNN seulement quand :
    -   le corpus est grand (dizaines/centaines de milliers de segments)
    -   la recherche d'embeddings par force brute devient trop lente
    -   la qualité du rappel est significativement limitée par la recherche lexicale

Alternatives adaptées au hors ligne (par complexité croissante) :

-   SQLite FTS5 + filtres de métadonnées (zéro ML)
-   Embeddings + force brute (fonctionne étonnamment loin si le nombre de segments est faible)
-   Index HNSW (commun, robuste ; nécessite une liaison de bibliothèque)
-   SuCo (niveau recherche ; attrayant s'il y a une implémentation solide que vous pouvez intégrer)

Question ouverte :

-   quel est le **meilleur** modèle d'embedding hors ligne pour la « mémoire d'assistant personnel » sur vos machines (ordinateur portable + bureau) ?
    -   si vous avez déjà Ollama : embedder avec un modèle local ; sinon, embarquer un petit modèle d'embedding dans la chaîne d'outils.

## Pilote minimal utile

Si vous voulez une version minimale, toujours utile :

-   Ajouter des pages d'entité `bank/` et une section `## Retenir` dans les journaux quotidiens.
-   Utiliser SQLite FTS pour le rappel avec citations (chemin + numéros de ligne).
-   Ajouter des embeddings seulement si la qualité du rappel ou l'échelle l'exige.

## Références

-   Concepts Letta / MemGPT : « blocs de mémoire cœur » + « mémoire d'archivage » + mémoire à auto-édition pilotée par outils.
-   Rapport technique Hindsight : « retenir / rappeler / réfléchir », mémoire à quatre réseaux, extraction de faits narratifs, évolution de la confiance des opinions.
-   SuCo : arXiv 2411.14754 (2024) : « Subspace Collision » récupération approximative des plus proches voisins.

[Plan d'agrégation de session indépendant du canal](../plans/session-binding-channel-agnostic.md)[Exploration de configuration de modèle](../proposals/model-config.md)

---