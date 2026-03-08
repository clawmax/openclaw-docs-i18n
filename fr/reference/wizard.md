title: "Documentation technique et référence de l'assistant d'intégration OpenClaw"
description: "Référence technique complète pour l'assistant CLI openclaw onboard. Apprenez le flux étape par étape pour la configuration locale, l'authentification, l'espace de travail, la passerelle et la configuration du démon."
keywords: ["openclaw", "assistant d'intégration", "référence cli", "guide d'installation", "authentification", "configuration de la passerelle", "installation du démon", "configuration de l'espace de travail"]
---

  Référence technique

  
# Référence de l'assistant d'intégration

Ceci est la référence complète pour l'assistant CLI `openclaw onboard`. Pour une vue d'ensemble, consultez [Assistant d'intégration](../start/wizard.md).

## Détails du flux (mode local)

### Étape 1 : Détection de la configuration existante

-   Si `~/.openclaw/openclaw.json` existe, choisissez **Conserver / Modifier / Réinitialiser**.
-   Relancer l'assistant **ne** supprime rien sauf si vous choisissez explicitement **Réinitialiser** (ou passez `--reset`).
-   L'option CLI `--reset` utilise par défaut `config+creds+sessions` ; utilisez `--reset-scope full` pour supprimer également l'espace de travail.
-   Si la configuration est invalide ou contient des clés obsolètes, l'assistant s'arrête et vous demande d'exécuter `openclaw doctor` avant de continuer.
-   La réinitialisation utilise `trash` (jamais `rm`) et propose des périmètres :
    -   Configuration uniquement
    -   Configuration + identifiants + sessions
    -   Réinitialisation complète (supprime aussi l'espace de travail)

### Étape 2 : Modèle/Authentification

-   **Clé API Anthropic** : utilise `ANTHROPIC_API_KEY` si présent ou demande une clé, puis la sauvegarde pour l'utilisation du démon.
-   **OAuth Anthropic (Claude Code CLI)** : sur macOS, l'assistant vérifie l'élément du trousseau "Claude Code-credentials" (choisissez "Toujours autoriser" pour que les démarrages launchd ne soient pas bloqués) ; sur Linux/Windows, il réutilise `~/.claude/.credentials.json` si présent.
-   **Jeton Anthropic (coller setup-token)** : exécutez `claude setup-token` sur n'importe quelle machine, puis collez le jeton (vous pouvez le nommer ; vide = par défaut).
-   **Abonnement OpenAI Code (Codex) (Codex CLI)** : si `~/.codex/auth.json` existe, l'assistant peut le réutiliser.
-   **Abonnement OpenAI Code (Codex) (OAuth)** : flux navigateur ; collez le `code#state`.
    -   Définit `agents.defaults.model` sur `openai-codex/gpt-5.2` lorsque le modèle n'est pas défini ou est `openai/*`.
-   **Clé API OpenAI** : utilise `OPENAI_API_KEY` si présent ou demande une clé, puis la stocke dans les profils d'authentification.
-   **Clé API xAI (Grok)** : demande `XAI_API_KEY` et configure xAI comme fournisseur de modèle.
-   **OpenCode Zen (proxy multi-modèles)** : demande `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`, obtenez-la sur [https://opencode.ai/auth](https://opencode.ai/auth)).
-   **Clé API** : stocke la clé pour vous.
-   **Vercel AI Gateway (proxy multi-modèles)** : demande `AI_GATEWAY_API_KEY`.
-   Plus de détails : [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway** : demande l'ID de compte, l'ID de passerelle et `CLOUDFLARE_AI_GATEWAY_API_KEY`.
-   Plus de détails : [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5** : la configuration est écrite automatiquement.
-   Plus de détails : [MiniMax](../providers/minimax.md)
-   **Synthetic (compatible Anthropic)** : demande `SYNTHETIC_API_KEY`.
-   Plus de détails : [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)** : la configuration est écrite automatiquement.
-   **Kimi Coding** : la configuration est écrite automatiquement.
-   Plus de détails : [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **Ignorer** : aucune authentification configurée pour l'instant.
-   Choisissez un modèle par défaut parmi les options détectées (ou saisissez manuellement fournisseur/modèle). Pour la meilleure qualité et un risque d'injection de prompt plus faible, choisissez le modèle le plus puissant de dernière génération disponible dans votre pile de fournisseurs.
-   L'assistant exécute une vérification du modèle et avertit si le modèle configuré est inconnu ou manque d'authentification.
-   Le mode de stockage des clés API utilise par défaut des valeurs en texte clair dans les profils d'authentification. Utilisez `--secret-input-mode ref` pour stocker des références basées sur des variables d'environnement à la place (par exemple `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
-   Les identifiants OAuth se trouvent dans `~/.openclaw/credentials/oauth.json` ; les profils d'authentification se trouvent dans `~/.openclaw/agents//agent/auth-profiles.json` (clés API + OAuth).
-   Plus de détails : [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** Astuce pour les environnements headless/serveur : complétez l'OAuth sur une machine avec un navigateur, puis copiez `~/.openclaw/credentials/oauth.json` (ou `$OPENCLAW_STATE_DIR/credentials/oauth.json`) vers l'hôte de la passerelle.

### Étape 3 : Espace de travail

-   Par défaut `~/.openclaw/workspace` (configurable).
-   Initialise les fichiers de l'espace de travail nécessaires pour le rituel de démarrage de l'agent.
-   Disposition complète de l'espace de travail + guide de sauvegarde : [Espace de travail de l'agent](../concepts/agent-workspace.md)

### Étape 4 : Passerelle

-   Port, liaison, mode d'authentification, exposition tailscale.
-   Recommandation d'authentification : conservez **Jeton** même pour loopback afin que les clients WS locaux doivent s'authentifier.
-   En mode jeton, l'intégration interactive propose :
    -   **Générer/stocker un jeton en texte clair** (par défaut)
    -   **Utiliser SecretRef** (opt-in)
    -   Le démarrage rapide réutilise les SecretRef `gateway.auth.token` existantes entre les fournisseurs `env`, `file` et `exec` pour la sonde d'intégration/le démarrage du tableau de bord.
    -   Si cette SecretRef est configurée mais ne peut pas être résolue, l'intégration échoue tôt avec un message de correction clair au lieu de dégrader silencieusement l'authentification à l'exécution.
-   En mode mot de passe, l'intégration interactive prend également en charge le stockage en texte clair ou via SecretRef.
-   Chemin de la SecretRef de jeton en mode non interactif : `--gateway-token-ref-env <ENV_VAR>`.
    -   Nécessite une variable d'environnement non vide dans l'environnement du processus d'intégration.
    -   Ne peut pas être combiné avec `--gateway-token`.
-   Désactivez l'authentification seulement si vous faites entièrement confiance à chaque processus local.
-   Les liaisons non loopback nécessitent toujours une authentification.

### Étape 5 : Canaux

-   [WhatsApp](../channels/whatsapp.md) : connexion QR facultative.
-   [Telegram](../channels/telegram.md) : jeton du bot.
-   [Discord](../channels/discord.md) : jeton du bot.
-   [Google Chat](../channels/googlechat.md) : JSON du compte de service + audience du webhook.
-   [Mattermost](../channels/mattermost.md) (plugin) : jeton du bot + URL de base.
-   [Signal](../channels/signal.md) : installation facultative de `signal-cli` + configuration du compte.
-   [BlueBubbles](../channels/bluebubbles.md) : **recommandé pour iMessage** ; URL du serveur + mot de passe + webhook.
-   [iMessage](../channels/imessage.md) : chemin CLI `imsg` hérité + accès à la base de données.
-   Sécurité des messages privés : le comportement par défaut est l'appairage. Le premier message privé envoie un code ; approuvez via `openclaw pairing approve  ` ou utilisez des listes d'autorisation.

### Étape 6 : Recherche web

-   Choisissez un fournisseur : Perplexity, Brave, Gemini, Grok ou Kimi (ou ignorez).
-   Collez votre clé API (le démarrage rapide détecte automatiquement les clés depuis les variables d'environnement ou la configuration existante).
-   Ignorez avec `--skip-search`.
-   Configurez plus tard : `openclaw configure --section web`.

### Étape 7 : Installation du démon

-   macOS : LaunchAgent
    -   Nécessite une session utilisateur connectée ; pour les environnements headless, utilisez un LaunchDaemon personnalisé (non fourni).
-   Linux (et Windows via WSL2) : unité utilisateur systemd
    -   L'assistant tente d'activer le mode lingering via `loginctl enable-linger ` pour que la passerelle reste active après la déconnexion.
    -   Peut demander sudo (écrit dans `/var/lib/systemd/linger`) ; il essaie d'abord sans sudo.
-   **Sélection de l'environnement d'exécution :** Node (recommandé ; requis pour WhatsApp/Telegram). Bun est **non recommandé**.
-   Si l'authentification par jeton nécessite un jeton et que `gateway.auth.token` est géré par SecretRef, l'installation du démon le valide mais ne persiste pas les valeurs de jeton en texte clair résolues dans les métadonnées d'environnement du service superviseur.
-   Si l'authentification par jeton nécessite un jeton et que la SecretRef configurée n'est pas résolue, l'installation du démon est bloquée avec des instructions actionnables.
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, l'installation du démon est bloquée jusqu'à ce que le mode soit défini explicitement.

### Étape 8 : Vérification de l'état

-   Démarre la passerelle (si nécessaire) et exécute `openclaw health`.
-   Astuce : `openclaw status --deep` ajoute des sondes de santé de la passerelle à la sortie de statut (nécessite une passerelle accessible).

### Étape 9 : Compétences (recommandé)

-   Lit les compétences disponibles et vérifie les prérequis.
-   Vous laisse choisir un gestionnaire de nœuds : **npm / pnpm** (bun non recommandé).
-   Installe les dépendances optionnelles (certaines utilisent Homebrew sur macOS).

### Étape 10 : Terminer

-   Résumé + prochaines étapes, y compris les applications iOS/Android/macOS pour des fonctionnalités supplémentaires.

 

> **ℹ️** Si aucune interface graphique n'est détectée, l'assistant affiche des instructions de port-forward SSH pour l'interface de contrôle au lieu d'ouvrir un navigateur. Si les ressources de l'interface de contrôle sont manquantes, l'assistant tente de les construire ; le repli est `pnpm ui:build` (installe automatiquement les dépendances de l'interface).

## Mode non interactif

Utilisez `--non-interactive` pour automatiser ou scripter l'intégration :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Ajoutez `--json` pour un résumé lisible par machine. SecretRef de jeton de passerelle en mode non interactif :

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` et `--gateway-token-ref-env` sont mutuellement exclusifs. 

> **ℹ️** `--json` n'implique **pas** le mode non interactif. Utilisez `--non-interactive` (et `--workspace`) pour les scripts.

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### Ajouter un agent (non interactif)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## RPC de l'assistant de la passerelle

La passerelle expose le flux de l'assistant via RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). Les clients (application macOS, interface de contrôle) peuvent afficher les étapes sans réimplémenter la logique d'intégration.

## Configuration de Signal (signal-cli)

L'assistant peut installer `signal-cli` depuis les versions GitHub :

-   Télécharge l'asset de version approprié.
-   Le stocke sous `~/.openclaw/tools/signal-cli//`.
-   Écrit `channels.signal.cliPath` dans votre configuration.

Notes :

-   Les builds JVM nécessitent **Java 21**.
-   Les builds natifs sont utilisés lorsqu'ils sont disponibles.
-   Windows utilise WSL2 ; l'installation de signal-cli suit le flux Linux à l'intérieur de WSL.

## Ce que l'assistant écrit

Champs typiques dans `~/.openclaw/openclaw.json` :

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (si Minimax est choisi)
-   `tools.profile` (l'intégration locale utilise par défaut `"coding"` lorsqu'elle n'est pas définie ; les valeurs explicites existantes sont préservées)
-   `gateway.*` (mode, liaison, authentification, tailscale)
-   `session.dmScope` (détails du comportement : [Référence CLI d'intégration](../start/wizard-cli-reference.md#outputs-and-internals))
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Listes d'autorisation des canaux (Slack/Discord/Matrix/Microsoft Teams) lorsque vous optez pour pendant les invites (les noms sont résolus en IDs lorsque possible).
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` écrit `agents.list[]` et les `bindings` optionnels. Les identifiants WhatsApp vont sous `~/.openclaw/credentials/whatsapp//`. Les sessions sont stockées sous `~/.openclaw/agents//sessions/`. Certains canaux sont livrés sous forme de plugins. Lorsque vous en choisissez un pendant l'intégration, l'assistant vous demandera de l'installer (npm ou un chemin local) avant de pouvoir le configurer.

## Documents connexes

-   Vue d'ensemble de l'assistant : [Assistant d'intégration](../start/wizard.md)
-   Intégration de l'application macOS : [Intégration](../start/onboarding.md)
-   Référence de configuration : [Configuration de la passerelle](../gateway/configuration.md)
-   Fournisseurs : [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md), [Google Chat](../channels/googlechat.md), [Signal](../channels/signal.md), [BlueBubbles](../channels/bluebubbles.md) (iMessage), [iMessage](../channels/imessage.md) (hérité)
-   Compétences : [Compétences](../tools/skills.md), [Configuration des compétences](../tools/skills-config.md)

[USER](./templates/USER.md)[Utilisation et coûts des jetons](./token-use.md)

---