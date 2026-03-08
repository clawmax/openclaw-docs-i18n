

  Maintenance

  
# Guide de migration

Ce guide permet de migrer une passerelle OpenClaw d'une machine à une autre **sans refaire l'intégration**. La migration est simple conceptuellement :

-   Copiez le **répertoire d'état** (`$OPENCLAW_STATE_DIR`, par défaut : `~/.openclaw/`) — il contient la configuration, l'authentification, les sessions et l'état des canaux.
-   Copiez votre **espace de travail** (`~/.openclaw/workspace/` par défaut) — il contient vos fichiers d'agent (mémoire, prompts, etc.).

Mais il existe des pièges courants concernant les **profils**, les **permissions** et les **copies partielles**.

## Avant de commencer (ce que vous migrez)

### 1) Identifiez votre répertoire d'état

La plupart des installations utilisent la valeur par défaut :

-   **Répertoire d'état :** `~/.openclaw/`

Mais il peut être différent si vous utilisez :

-   `--profile ` (devient souvent `~/.openclaw-/`)
-   `OPENCLAW_STATE_DIR=/chemin/personnalisé`

Si vous n'êtes pas sûr, exécutez sur l'**ancienne** machine :

```bash
openclaw status
```

Cherchez les mentions de `OPENCLAW_STATE_DIR` / du profil dans la sortie. Si vous exécutez plusieurs passerelles, répétez pour chaque profil.

### 2) Identifiez votre espace de travail

Valeurs par défaut courantes :

-   `~/.openclaw/workspace/` (espace de travail recommandé)
-   un dossier personnalisé que vous avez créé

Votre espace de travail est l'endroit où se trouvent des fichiers comme `MEMORY.md`, `USER.md` et `memory/*.md`.

### 3) Comprenez ce que vous allez préserver

Si vous copiez **à la fois** le répertoire d'état et l'espace de travail, vous conservez :

-   La configuration de la passerelle (`openclaw.json`)
-   Les profils d'authentification / clés API / jetons OAuth
-   L'historique des sessions + l'état de l'agent
-   L'état des canaux (ex. : connexion/session WhatsApp)
-   Vos fichiers d'espace de travail (mémoire, notes de compétences, etc.)

Si vous copiez **uniquement** l'espace de travail (ex. via Git), vous **ne** conservez **pas** :

-   les sessions
-   les identifiants
-   les connexions aux canaux

Ceux-ci se trouvent sous `$OPENCLAW_STATE_DIR`.

## Étapes de migration (recommandées)

### Étape 0 — Faites une sauvegarde (ancienne machine)

Sur l'**ancienne** machine, arrêtez d'abord la passerelle pour que les fichiers ne changent pas pendant la copie :

```bash
openclaw gateway stop
```

(Optionnel mais recommandé) archivez le répertoire d'état et l'espace de travail :

```bash
# Ajustez les chemins si vous utilisez un profil ou des emplacements personnalisés
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Si vous avez plusieurs profils/répertoires d'état (ex. `~/.openclaw-main`, `~/.openclaw-work`), archivez chacun.

### Étape 1 — Installez OpenClaw sur la nouvelle machine

Sur la **nouvelle** machine, installez la CLI (et Node si nécessaire) :

-   Voir : [Installation](../install.md)

À ce stade, ce n'est pas grave si l'intégration crée un nouveau `~/.openclaw/` — vous l'écraserez à l'étape suivante.

### Étape 2 — Copiez le répertoire d'état + l'espace de travail sur la nouvelle machine

Copiez **les deux** :

-   `$OPENCLAW_STATE_DIR` (par défaut `~/.openclaw/`)
-   votre espace de travail (par défaut `~/.openclaw/workspace/`)

Approches courantes :

-   `scp` les archives tar et extrayez-les
-   `rsync -a` via SSH
-   disque externe

Après la copie, assurez-vous que :

-   Les répertoires cachés ont été inclus (ex. `.openclaw/`)
-   La propriété des fichiers est correcte pour l'utilisateur exécutant la passerelle

### Étape 3 — Exécutez Doctor (migrations + réparation des services)

Sur la **nouvelle** machine :

```bash
openclaw doctor
```

Doctor est la commande "sûre et ennuyeuse". Il répare les services, applique les migrations de configuration et signale les incohérences. Ensuite :

```bash
openclaw gateway restart
openclaw status
```

## Pièges courants (et comment les éviter)

### Piège : incohérence de profil / répertoire d'état

Si vous avez exécuté l'ancienne passerelle avec un profil (ou `OPENCLAW_STATE_DIR`), et que la nouvelle passerelle en utilise un différent, vous verrez des symptômes comme :

-   les modifications de configuration ne prennent pas effet
-   les canaux manquent / sont déconnectés
-   l'historique des sessions est vide

Solution : exécutez la passerelle/le service en utilisant le **même** profil/répertoire d'état que vous avez migré, puis réexécutez :

```bash
openclaw doctor
```

### Piège : copier uniquement openclaw.json

`openclaw.json` ne suffit pas. De nombreux fournisseurs stockent l'état sous :

-   `$OPENCLAW_STATE_DIR/credentials/`
-   `$OPENCLAW_STATE_DIR/agents//...`

Migrez toujours l'intégralité du dossier `$OPENCLAW_STATE_DIR`.

### Piège : permissions / propriété

Si vous avez copié en tant que root ou changé d'utilisateur, la passerelle peut échouer à lire les identifiants/sessions. Solution : assurez-vous que le répertoire d'état + l'espace de travail appartiennent à l'utilisateur exécutant la passerelle.

### Piège : migration entre modes distant/local

-   Si votre interface (WebUI/TUI) pointe vers une passerelle **distante**, l'hôte distant possède le magasin de sessions + l'espace de travail.
-   Migrer votre ordinateur portable ne déplacera pas l'état de la passerelle distante.

Si vous êtes en mode distant, migrez l'**hôte de la passerelle**.

### Piège : secrets dans les sauvegardes

`$OPENCLAW_STATE_DIR` contient des secrets (clés API, jetons OAuth, identifiants WhatsApp). Traitez les sauvegardes comme des secrets de production :

-   stockez-les chiffrées
-   évitez de les partager sur des canaux non sécurisés
-   faites tourner les clés si vous soupçonnez une exposition

## Liste de vérification

Sur la nouvelle machine, confirmez que :

-   `openclaw status` montre la passerelle en cours d'exécution
-   Vos canaux sont toujours connectés (ex. : WhatsApp ne nécessite pas de re-jumelage)
-   Le tableau de bord s'ouvre et affiche les sessions existantes
-   Vos fichiers d'espace de travail (mémoire, configurations) sont présents

## Liens connexes

-   [Doctor](../gateway/doctor.md)
-   [Dépannage de la passerelle](../gateway/troubleshooting.md)
-   [Où OpenClaw stocke-t-il ses données ?](../help/faq.md#where-does-openclaw-store-its-data)

[Mise à jour](./updating.md)[Désinstallation](./uninstall.md)