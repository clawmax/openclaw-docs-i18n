

  Maintenance

  
# Mise à jour

OpenClaw évolue rapidement (pré-"1.0"). Traitez les mises à jour comme le déploiement d'infrastructure : mise à jour → exécution des vérifications → redémarrage (ou utilisez `openclaw update`, qui redémarre) → vérification.

## Recommandé : ré-exécuter l'installateur du site web (mise à niveau sur place)

Le chemin de mise à jour **préféré** est de ré-exécuter l'installateur depuis le site web. Il détecte les installations existantes, effectue une mise à niveau sur place, et exécute `openclaw doctor` si nécessaire.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Notes :

-   Ajoutez `--no-onboard` si vous ne voulez pas que l'assistant de configuration initiale se ré-exécute.
-   Pour les **installations depuis les sources**, utilisez :
    
    Copier
    
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
    ```
    
    L'installateur effectuera un `git pull --rebase` **uniquement** si le dépôt est propre.
-   Pour les **installations globales**, le script utilise `npm install -g openclaw@latest` en interne.
-   Note historique : `clawdbot` reste disponible en tant que couche de compatibilité.

## Avant de mettre à jour

-   Sachez comment vous avez installé : **global** (npm/pnpm) vs **depuis les sources** (git clone).
-   Sachez comment votre Gateway s'exécute : **terminal au premier plan** vs **service supervisé** (launchd/systemd).
-   Faites un instantané de votre configuration personnalisée :
    -   Configuration : `~/.openclaw/openclaw.json`
    -   Identifiants : `~/.openclaw/credentials/`
    -   Espace de travail : `~/.openclaw/workspace`

## Mise à jour (installation globale)

Installation globale (choisissez une méthode) :

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Nous ne recommandons **pas** Bun pour l'exécution de la Gateway (bugs WhatsApp/Telegram). Pour changer de canal de mise à jour (installations git + npm) :

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Utilisez `--tag <dist-tag|version>` pour une installation ponctuelle avec un tag/version spécifique. Voir [Canaux de développement](./development-channels.md) pour la sémantique des canaux et les notes de version. Note : sur les installations npm, la passerelle enregistre un indice de mise à jour au démarrage (vérifie le tag du canal actuel). Désactivez via `update.checkOnStart: false`.

### Auto-updateur intégré (optionnel)

L'auto-updateur est **désactivé par défaut** et est une fonctionnalité de base de la Gateway (pas un plugin).

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

Comportement :

-   `stable` : lorsqu'une nouvelle version est détectée, OpenClaw attend `stableDelayHours` puis applique un décalage déterministe par installation dans `stableJitterHours` (déploiement étalé).
-   `beta` : vérifie selon la cadence `betaCheckIntervalHours` (par défaut : toutes les heures) et applique lorsqu'une mise à jour est disponible.
-   `dev` : pas d'application automatique ; utilisez `openclaw update` manuel.

Utilisez `openclaw update --dry-run` pour prévisualiser les actions de mise à jour avant d'activer l'automatisation. Ensuite :

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notes :

-   Si votre Gateway s'exécute en tant que service, `openclaw gateway restart` est préférable à l'arrêt des PID.
-   Si vous êtes épinglé à une version spécifique, voir "Revenir en arrière / épinglage" ci-dessous.

## Mise à jour (openclaw update)

Pour les **installations depuis les sources** (copie git), préférez :

```bash
openclaw update
```

Il exécute un flux de mise à jour relativement sûr :

-   Nécessite un arbre de travail propre.
-   Bascule vers le canal sélectionné (tag ou branche).
-   Récupère et rebase par rapport à la source distante configurée (canal dev).
-   Installe les dépendances, construit, construit l'interface de contrôle (Control UI) et exécute `openclaw doctor`.
-   Redémarre la passerelle par défaut (utilisez `--no-restart` pour ignorer).

Si vous avez installé via **npm/pnpm** (pas de métadonnées git), `openclaw update` tentera de mettre à jour via votre gestionnaire de paquets. S'il ne peut pas détecter l'installation, utilisez plutôt "Mise à jour (installation globale)".

## Mise à jour (Interface de contrôle / RPC)

L'Interface de contrôle a une option **Mettre à jour et redémarrer** (RPC : `update.run`). Elle :

1.  Exécute le même flux de mise à jour depuis les sources que `openclaw update` (uniquement pour les copies git).
2.  Écrit un indicateur de redémarrage avec un rapport structuré (extrait de stdout/stderr).
3.  Redémarre la passerelle et envoie un ping à la dernière session active avec le rapport.

Si le rebase échoue, la passerelle abandonne et redémarre sans appliquer la mise à jour.

## Mise à jour (depuis les sources)

Depuis la copie du dépôt : Méthode préférée :

```bash
openclaw update
```

Manuelle (équivalente) :

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # installe automatiquement les dépendances de l'UI au premier lancement
openclaw doctor
openclaw health
```

Notes :

-   `pnpm build` est important lorsque vous exécutez le binaire empaqueté `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) ou utilisez Node pour exécuter `dist/`.
-   Si vous exécutez depuis une copie de dépôt sans installation globale, utilisez `pnpm openclaw ...` pour les commandes CLI.
-   Si vous exécutez directement depuis TypeScript (`pnpm openclaw ...`), une reconstruction est généralement inutile, mais **les migrations de configuration s'appliquent toujours** → exécutez doctor.
-   Basculer entre les installations globales et git est facile : installez l'autre méthode, puis exécutez `openclaw doctor` pour que le point d'entrée du service de la passerelle soit réécrit pour l'installation actuelle.

## Toujours exécuter : openclaw doctor

Doctor est la commande de "mise à jour sûre". Elle est volontairement simple : réparation + migration + avertissement. Note : si vous êtes sur une **installation depuis les sources** (copie git), `openclaw doctor` proposera d'abord d'exécuter `openclaw update`. Ce qu'il fait typiquement :

-   Migre les clés de configuration obsolètes / les anciens emplacements de fichiers de configuration.
-   Audite les politiques de messages directs (DM) et avertit sur les paramètres "ouverts" risqués.
-   Vérifie l'état de santé de la Gateway et peut proposer un redémarrage.
-   Détecte et migre les anciens services de passerelle (launchd/systemd ; anciens schtasks) vers les services OpenClaw actuels.
-   Sur Linux, assure la persistance de session systemd (pour que la Gateway survive à la déconnexion).

Détails : [Doctor](../gateway/doctor.md)

## Démarrer / arrêter / redémarrer la Gateway

CLI (fonctionne quel que soit l'OS) :

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Si vous êtes supervisé :

-   macOS launchd (LaunchAgent intégré à l'application) : `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (utilisez `ai.openclaw.` ; l'ancien `com.openclaw.*` fonctionne toujours)
-   Service utilisateur systemd Linux : `systemctl --user restart openclaw-gateway[-].service`
-   Windows (WSL2) : `systemctl --user restart openclaw-gateway[-].service`
    -   `launchctl`/`systemctl` ne fonctionnent que si le service est installé ; sinon, exécutez `openclaw gateway install`.

Runbook + étiquettes exactes des services : [Runbook de la Gateway](../gateway.md)

## Revenir en arrière / épinglage (en cas de problème)

### Épingler (installation globale)

Installez une version connue comme stable (remplacez `` par la dernière version fonctionnelle) :

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Astuce : pour voir la version publiée actuelle, exécutez `npm view openclaw version`. Ensuite, redémarrez et ré-exécutez doctor :

```bash
openclaw doctor
openclaw gateway restart
```

### Épingler (sources) par date

Choisissez un commit à une date (exemple : "état de main au 2026-01-01") :

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Ensuite, réinstallez les dépendances + redémarrez :

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Si vous voulez revenir à la dernière version plus tard :

```bash
git checkout main
git pull
```

## Si vous êtes bloqué

-   Exécutez `openclaw doctor` à nouveau et lisez attentivement la sortie (il indique souvent la solution).
-   Consultez : [Dépannage](../gateway/troubleshooting.md)
-   Demandez sur Discord : [https://discord.gg/clawd](https://discord.gg/clawd)

[Bun (Expérimental)](./bun.md)[Guide de migration](./migrating.md)