

  Compétences

  
# ClawHub

ClawHub est le **registre public de compétences pour OpenClaw**. C'est un service gratuit : toutes les compétences sont publiques, ouvertes et visibles par tous pour le partage et la réutilisation. Une compétence est simplement un dossier contenant un fichier `SKILL.md` (plus des fichiers texte de support). Vous pouvez parcourir les compétences dans l'application web ou utiliser la CLI pour rechercher, installer, mettre à jour et publier des compétences. Site : [clawhub.ai](https://clawhub.ai)

## Qu'est-ce que ClawHub

-   Un registre public pour les compétences OpenClaw.
-   Un stockage versionné des bundles de compétences et des métadonnées.
-   Une surface de découverte pour la recherche, les tags et les signaux d'utilisation.

## Comment ça fonctionne

1.  Un utilisateur publie un bundle de compétences (fichiers + métadonnées).
2.  ClawHub stocke le bundle, analyse les métadonnées et attribue une version.
3.  Le registre indexe la compétence pour la recherche et la découverte.
4.  Les utilisateurs parcourent, téléchargent et installent les compétences dans OpenClaw.

## Ce que vous pouvez faire

-   Publier de nouvelles compétences et de nouvelles versions de compétences existantes.
-   Découvrir des compétences par nom, tags ou recherche.
-   Télécharger des bundles de compétences et inspecter leurs fichiers.
-   Signaler des compétences abusives ou non sécurisées.
-   Si vous êtes modérateur, masquer, démasquer, supprimer ou bannir.

## À qui cela s'adresse (débutant)

Si vous souhaitez ajouter de nouvelles capacités à votre agent OpenClaw, ClawHub est le moyen le plus simple de trouver et d'installer des compétences. Vous n'avez pas besoin de savoir comment fonctionne le backend. Vous pouvez :

-   Rechercher des compétences en langage naturel.
-   Installer une compétence dans votre espace de travail.
-   Mettre à jour les compétences plus tard avec une seule commande.
-   Sauvegarder vos propres compétences en les publiant.

## Démarrage rapide (non technique)

1.  Installez la CLI (voir la section suivante).
2.  Recherchez quelque chose dont vous avez besoin :
    -   `clawhub search "calendar"`
3.  Installez une compétence :
    -   `clawhub install <skill-slug>`
4.  Démarrez une nouvelle session OpenClaw pour qu'elle prenne en compte la nouvelle compétence.

## Installer la CLI

Choisissez une option :

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## Comment cela s'intègre à OpenClaw

Par défaut, la CLI installe les compétences dans `./skills` sous votre répertoire de travail actuel. Si un espace de travail OpenClaw est configuré, `clawhub` utilise cet espace de travail par défaut sauf si vous le remplacez avec `--workdir` (ou `CLAWHUB_WORKDIR`). OpenClaw charge les compétences de l'espace de travail depuis `/skills` et les prendra en compte dans la session **suivante**. Si vous utilisez déjà `~/.openclaw/skills` ou des compétences groupées, les compétences de l'espace de travail ont la priorité. Pour plus de détails sur le chargement, le partage et la restriction des compétences, consultez [Compétences](./skills.md).

## Aperçu du système de compétences

Une compétence est un bundle versionné de fichiers qui apprend à OpenClaw comment effectuer une tâche spécifique. Chaque publication crée une nouvelle version, et le registre conserve un historique des versions afin que les utilisateurs puissent auditer les changements. Une compétence typique comprend :

-   Un fichier `SKILL.md` avec la description principale et l'utilisation.
-   Des configurations, scripts ou fichiers de support optionnels utilisés par la compétence.
-   Des métadonnées telles que des tags, un résumé et des prérequis d'installation.

ClawHub utilise les métadonnées pour alimenter la découverte et exposer en toute sécurité les capacités des compétences. Le registre suit également les signaux d'utilisation (comme les étoiles et les téléchargements) pour améliorer le classement et la visibilité.

## Ce que le service fournit (fonctionnalités)

-   **Navigation publique** des compétences et de leur contenu `SKILL.md`.
-   **Recherche** alimentée par des embeddings (recherche vectorielle), pas seulement par mots-clés.
-   **Gestion de versions** avec semver, journaux des modifications et tags (y compris `latest`).
-   **Téléchargements** sous forme de zip par version.
-   **Étoiles et commentaires** pour les retours de la communauté.
-   **Crochets de modération** pour les approbations et audits.
-   **API adaptée à la CLI** pour l'automatisation et le scriptage.

## Sécurité et modération

ClawHub est ouvert par défaut. N'importe qui peut télécharger des compétences, mais un compte GitHub doit avoir au moins une semaine pour publier. Cela aide à ralentir les abus sans bloquer les contributeurs légitimes. Signalement et modération :

-   Tout utilisateur connecté peut signaler une compétence.
-   Les raisons du signalement sont obligatoires et enregistrées.
-   Chaque utilisateur peut avoir jusqu'à 20 signalements actifs à la fois.
-   Les compétences avec plus de 3 signalements uniques sont automatiquement masquées par défaut.
-   Les modérateurs peuvent voir les compétences masquées, les démasquer, les supprimer ou bannir des utilisateurs.
-   Abuser de la fonctionnalité de signalement peut entraîner le bannissement du compte.

Intéressé à devenir modérateur ? Demandez sur le Discord OpenClaw et contactez un modérateur ou un mainteneur.

## Commandes et paramètres de la CLI

Options globales (s'appliquent à toutes les commandes) :

-   `--workdir ` : Répertoire de travail (par défaut : répertoire actuel ; utilise l'espace de travail OpenClaw par défaut).
-   `--dir ` : Répertoire des compétences, relatif au workdir (par défaut : `skills`).
-   `--site ` : URL de base du site (connexion par navigateur).
-   `--registry ` : URL de base de l'API du registre.
-   `--no-input` : Désactiver les invites (non interactif).
-   `-V, --cli-version` : Afficher la version de la CLI.

Authentification :

-   `clawhub login` (flux navigateur) ou `clawhub login --token `
-   `clawhub logout`
-   `clawhub whoami`

Options :

-   `--token ` : Coller un jeton API.
-   `--label ` : Étiquette stockée pour les jetons de connexion navigateur (par défaut : `CLI token`).
-   `--no-browser` : Ne pas ouvrir de navigateur (nécessite `--token`).

Recherche :

-   `clawhub search "query"`
-   `--limit ` : Nombre maximum de résultats.

Installation :

-   `clawhub install `
-   `--version ` : Installer une version spécifique.
-   `--force` : Écraser si le dossier existe déjà.

Mise à jour :

-   `clawhub update `
-   `clawhub update --all`
-   `--version ` : Mettre à jour vers une version spécifique (un seul slug).
-   `--force` : Écraser lorsque les fichiers locaux ne correspondent à aucune version publiée.

Liste :

-   `clawhub list` (lit `.clawhub/lock.json`)

Publication :

-   `clawhub publish `
-   `--slug ` : Slug de la compétence.
-   `--name ` : Nom d'affichage.
-   `--version ` : Version semver.
-   `--changelog ` : Texte du journal des modifications (peut être vide).
-   `--tags ` : Tags séparés par des virgules (par défaut : `latest`).

Suppression/restauration (propriétaire/administrateur uniquement) :

-   `clawhub delete  --yes`
-   `clawhub undelete  --yes`

Synchronisation (analyse les compétences locales + publie les nouvelles/mises à jour) :

-   `clawhub sync`
-   `--root <dir...>` : Racines d'analyse supplémentaires.
-   `--all` : Tout téléverser sans invites.
-   `--dry-run` : Afficher ce qui serait téléversé.
-   `--bump ` : `patch|minor|major` pour les mises à jour (par défaut : `patch`).
-   `--changelog ` : Journal des modifications pour les mises à jour non interactives.
-   `--tags ` : Tags séparés par des virgules (par défaut : `latest`).
-   `--concurrency ` : Vérifications du registre (par défaut : 4).

## Flux de travail courants pour les agents

### Rechercher des compétences

```bash
clawhub search "postgres backups"
```

### Télécharger de nouvelles compétences

```bash
clawhub install my-skill-pack
```

### Mettre à jour les compétences installées

```bash
clawhub update --all
```

### Sauvegarder vos compétences (publier ou synchroniser)

Pour un dossier de compétence unique :

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Pour analyser et sauvegarder plusieurs compétences à la fois :

```bash
clawhub sync --all
```

## Détails avancés (technique)

### Gestion de versions et tags

-   Chaque publication crée une nouvelle **semver** `SkillVersion`.
-   Les tags (comme `latest`) pointent vers une version ; déplacer les tags permet de revenir en arrière.
-   Les journaux des modifications sont attachés par version et peuvent être vides lors de la synchronisation ou de la publication de mises à jour.

### Modifications locales vs versions du registre

Les mises à jour comparent le contenu local de la compétence aux versions du registre en utilisant un hachage de contenu. Si les fichiers locaux ne correspondent à aucune version publiée, la CLI demande avant d'écraser (ou nécessite `--force` dans les exécutions non interactives).

### Analyse de synchronisation et racines de repli

`clawhub sync` analyse d'abord votre répertoire de travail actuel. Si aucune compétence n'est trouvée, il utilise par défaut des emplacements hérités connus (par exemple `~/openclaw/skills` et `~/.openclaw/skills`). Cela est conçu pour trouver les installations de compétences plus anciennes sans drapeaux supplémentaires.

### Stockage et fichier de verrouillage

-   Les compétences installées sont enregistrées dans `.clawhub/lock.json` sous votre répertoire de travail.
-   Les jetons d'authentification sont stockés dans le fichier de configuration de la CLI ClawHub (remplaçable via `CLAWHUB_CONFIG_PATH`).

### Télémétrie (comptes d'installation)

Lorsque vous exécutez `clawhub sync` tout en étant connecté, la CLI envoie un instantané minimal pour calculer les comptes d'installation. Vous pouvez désactiver cela complètement :

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

## Variables d'environnement

-   `CLAWHUB_SITE` : Remplacer l'URL du site.
-   `CLAWHUB_REGISTRY` : Remplacer l'URL de l'API du registre.
-   `CLAWHUB_CONFIG_PATH` : Remplacer l'emplacement où la CLI stocke le jeton/la configuration.
-   `CLAWHUB_WORKDIR` : Remplacer le répertoire de travail par défaut.
-   `CLAWHUB_DISABLE_TELEMETRY=1` : Désactiver la télémétrie sur `sync`.

[Configuration des Compétences](./skills-config.md)[Plugins](./plugin.md)

---