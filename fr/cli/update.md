

  Commandes CLI

  
# update

Mettez à jour OpenClaw en toute sécurité et basculez entre les canaux stable/bêta/dev. Si vous avez installé via **npm/pnpm** (installation globale, sans métadonnées git), les mises à jour se font via le flux du gestionnaire de paquets décrit dans [Mise à jour](../install/updating.md).

## Utilisation

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## Options

-   `--no-restart` : ne pas redémarrer le service Gateway après une mise à jour réussie.
-   `--channel <stable|beta|dev>` : définir le canal de mise à jour (git + npm ; persistant dans la configuration).
-   `--tag <dist-tag|version>` : remplacer le dist-tag npm ou la version uniquement pour cette mise à jour.
-   `--dry-run` : prévisualiser les actions de mise à jour planifiées (flux canal/tag/cible/redémarrage) sans écrire la configuration, installer, synchroniser les plugins ou redémarrer.
-   `--json` : afficher un JSON `UpdateRunResult` lisible par machine.
-   `--timeout ` : délai d'expiration par étape (par défaut 1200s).

Remarque : les rétrogradations nécessitent une confirmation car les anciennes versions peuvent casser la configuration.

## update status

Affiche le canal de mise à jour actif + le tag/branche/SHA git (pour les installations depuis les sources), ainsi que la disponibilité des mises à jour.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options :

-   `--json` : afficher un JSON de statut lisible par machine.
-   `--timeout ` : délai d'expiration pour les vérifications (par défaut 3s).

## update wizard

Flux interactif pour choisir un canal de mise à jour et confirmer s'il faut redémarrer la Gateway après la mise à jour (par défaut, redémarre). Si vous sélectionnez `dev` sans dépôt git, il propose d'en créer un.

## Ce que cela fait

Lorsque vous changez de canal explicitement (`--channel ...`), OpenClaw aligne également la méthode d'installation :

-   `dev` → s'assure d'avoir un dépôt git (par défaut : `~/openclaw`, à remplacer par `OPENCLAW_GIT_DIR`), le met à jour, et installe le CLI global depuis ce dépôt.
-   `stable`/`beta` → installe depuis npm en utilisant le dist-tag correspondant.

L'auto-mise à jour du cœur de la Gateway (lorsqu'elle est activée via la configuration) réutilise ce même chemin de mise à jour.

## Flux d'installation depuis Git

Canaux :

-   `stable` : extrait le dernier tag non-bêta, puis construit + exécute doctor.
-   `beta` : extrait le dernier tag `-beta`, puis construit + exécute doctor.
-   `dev` : extrait `main`, puis fetch + rebase.

Aperçu général :

1.  Nécessite un arbre de travail propre (pas de modifications non validées).
2.  Bascule vers le canal sélectionné (tag ou branche).
3.  Récupère les modifications en amont (dev uniquement).
4.  Dev uniquement : pré-vérification lint + construction TypeScript dans un arbre de travail temporaire ; si la pointe échoue, remonte jusqu'à 10 commits pour trouver la dernière construction propre.
5.  Rebase sur le commit sélectionné (dev uniquement).
6.  Installe les dépendances (pnpm préféré ; npm en secours).
7.  Construit + construit l'interface de contrôle (Control UI).
8.  Exécute `openclaw doctor` comme vérification finale de "mise à jour sûre".
9.  Synchronise les plugins vers le canal actif (dev utilise les extensions incluses ; stable/bêta utilise npm) et met à jour les plugins installés via npm.

## Raccourci \--update

`openclaw --update` est réécrit en `openclaw update` (utile pour les shells et les scripts de lancement).

## Voir aussi

-   `openclaw doctor` (propose d'exécuter la mise à jour en premier sur les installations depuis git)
-   [Canaux de développement](../install/development-channels.md)
-   [Mise à jour](../install/updating.md)
-   [Référence CLI](../cli.md)

[uninstall](./uninstall.md)[voicecall](./voicecall.md)

---