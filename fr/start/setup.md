

  Configuration développeur

  
# Configuration

> **ℹ️** Si vous configurez pour la première fois, commencez par [Démarrage](./getting-started.md). Pour les détails de l'assistant, voir [Assistant d'intégration](./wizard.md).

 Dernière mise à jour : 2026-01-01

## TL;DR

-   **La personnalisation vit en dehors du dépôt :** `~/.openclaw/workspace` (espace de travail) + `~/.openclaw/openclaw.json` (config).
-   **Workflow stable :** installez l'application macOS ; laissez-la exécuter la Gateway incluse.
-   **Workflow de pointe :** exécutez la Gateway vous-même via `pnpm gateway:watch`, puis laissez l'application macOS s'attacher en mode Local.

## Prérequis (à partir des sources)

-   Node `>=22`
-   `pnpm`
-   Docker (optionnel ; uniquement pour la configuration/e2e conteneurisée — voir [Docker](../install/docker.md))

## Stratégie de personnalisation (pour que les mises à jour ne nuisent pas)

Si vous voulez une configuration "100% adaptée à moi" *et* des mises à jour faciles, gardez vos personnalisations dans :

-   **Config :** `~/.openclaw/openclaw.json` (JSON/JSON5-ish)
-   **Espace de travail :** `~/.openclaw/workspace` (compétences, invites, mémoires ; faites-en un dépôt git privé)

Amorçage une fois :

```bash
openclaw setup
```

Depuis ce dépôt, utilisez l'entrée CLI locale :

```bash
openclaw setup
```

Si vous n'avez pas encore d'installation globale, exécutez-la via `pnpm openclaw setup`.

## Exécuter la Gateway depuis ce dépôt

Après `pnpm build`, vous pouvez exécuter la CLI empaquetée directement :

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## Workflow stable (application macOS d'abord)

1.  Installez + lancez **OpenClaw.app** (barre de menus).
2.  Complétez la liste de contrôle d'intégration/autorisations (invites TCC).
3.  Assurez-vous que la Gateway est en **Local** et qu'elle tourne (l'application la gère).
4.  Liez des surfaces (exemple : WhatsApp) :

```bash
openclaw channels login
```

5.  Vérification de bon fonctionnement :

```bash
openclaw health
```

Si l'intégration n'est pas disponible dans votre build :

-   Exécutez `openclaw setup`, puis `openclaw channels login`, puis démarrez la Gateway manuellement (`openclaw gateway`).

## Workflow de pointe (Gateway dans un terminal)

Objectif : travailler sur la Gateway TypeScript, obtenir le rechargement à chaud, garder l'interface de l'application macOS attachée.

### 0) (Optionnel) Exécutez aussi l'application macOS depuis les sources

Si vous voulez aussi l'application macOS en version de pointe :

```
./scripts/restart-mac.sh
```

### 1) Démarrez la Gateway de développement

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` exécute la gateway en mode surveillance et la recharge sur les changements TypeScript.

### 2) Pointez l'application macOS vers votre Gateway en cours d'exécution

Dans **OpenClaw.app** :

-   Mode de connexion : **Local** L'application s'attachera à la gateway en cours d'exécution sur le port configuré.

### 3) Vérifiez

-   Le statut de la Gateway dans l'application devrait indiquer **"Utilisation de la gateway existante …"**
-   Ou via la CLI :

```bash
openclaw health
```

### Pièges courants

-   **Mauvais port :** La Gateway WS utilise par défaut `ws://127.0.0.1:18789` ; gardez l'application et la CLI sur le même port.
-   **Où vit l'état :**
    -   Identifiants : `~/.openclaw/credentials/`
    -   Sessions : `~/.openclaw/agents//sessions/`
    -   Logs : `/tmp/openclaw/`

## Carte de stockage des identifiants

Utilisez ceci lors du débogage de l'authentification ou pour décider quoi sauvegarder :

-   **WhatsApp** : `~/.openclaw/credentials/whatsapp//creds.json`
-   **Jeton de bot Telegram** : config/env ou `channels.telegram.tokenFile`
-   **Jeton de bot Discord** : config/env ou SecretRef (fournisseurs env/file/exec)
-   **Jetons Slack** : config/env (`channels.slack.*`)
-   **Listes d'autorisation d'appairage** :
    -   `~/.openclaw/credentials/-allowFrom.json` (compte par défaut)
    -   `~/.openclaw/credentials/--allowFrom.json` (comptes non par défaut)
-   **Profils d'authentification de modèle** : `~/.openclaw/agents//agent/auth-profiles.json`
-   **Charge utile de secrets basée sur des fichiers (optionnelle)** : `~/.openclaw/secrets.json`
-   **Import OAuth hérité** : `~/.openclaw/credentials/oauth.json` Plus de détails : [Sécurité](../gateway/security.md#credential-storage-map).

## Mise à jour (sans casser votre configuration)

-   Gardez `~/.openclaw/workspace` et `~/.openclaw/` comme "vos affaires" ; ne mettez pas vos invites/config personnelles dans le dépôt `openclaw`.
-   Mise à jour des sources : `git pull` + `pnpm install` (quand le lockfile change) + continuez à utiliser `pnpm gateway:watch`.

## Linux (service utilisateur systemd)

Les installations Linux utilisent un service **utilisateur** systemd. Par défaut, systemd arrête les services utilisateur à la déconnexion/inactivité, ce qui tue la Gateway. L'intégration tente d'activer le mode persistant pour vous (peut demander sudo). Si c'est toujours désactivé, exécutez :

```bash
sudo loginctl enable-linger $USER
```

Pour les serveurs toujours actifs ou multi-utilisateurs, envisagez un service **système** au lieu d'un service utilisateur (pas besoin de mode persistant). Voir le [runbook de la Gateway](../gateway.md) pour les notes systemd.

## Documentation associée

-   [Runbook de la Gateway](../gateway.md) (drapeaux, supervision, ports)
-   [Configuration de la Gateway](../gateway/configuration.md) (schéma de config + exemples)
-   [Discord](../channels/discord.md) et [Telegram](../channels/telegram.md) (balises de réponse + paramètres replyToMode)
-   [Configuration de l'assistant OpenClaw](./openclaw.md)
-   [Application macOS](../platforms/macos.md) (cycle de vie de la gateway)

[Plongée approfondie dans la gestion des sessions](../reference/session-management-compaction.md)[Workflow de développement Pi](../pi-dev.md)

---