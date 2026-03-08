

  Guides

  
# Référence d'intégration CLI

Cette page est la référence complète pour `openclaw onboard`. Pour le guide rapide, consultez [Assistant d'intégration (CLI)](./wizard.md).

## Ce que fait l'assistant

Le mode local (par défaut) vous guide à travers :

-   La configuration des modèles et de l'authentification (OAuth d'abonnement OpenAI Code, clé API Anthropic ou jeton de configuration, plus les options MiniMax, GLM, Moonshot et AI Gateway)
-   L'emplacement de l'espace de travail et les fichiers d'amorçage
-   Les paramètres de la passerelle (port, liaison, authentification, tailscale)
-   Les canaux et fournisseurs (Telegram, WhatsApp, Discord, Google Chat, plugin Mattermost, Signal)
-   L'installation du démon (LaunchAgent ou unité utilisateur systemd)
-   Le contrôle d'intégrité
-   La configuration des compétences

Le mode distant configure cette machine pour se connecter à une passerelle ailleurs. Il n'installe ni ne modifie quoi que ce soit sur l'hôte distant.

## Détails du flux local

### Étape 1 : Détection de configuration existante

-   Si `~/.openclaw/openclaw.json` existe, choisissez de Conserver, Modifier ou Réinitialiser.
-   Relancer l'assistant n'efface rien sauf si vous choisissez explicitement Réinitialiser (ou passez `--reset`).
-   L'option CLI `--reset` utilise par défaut `config+creds+sessions` ; utilisez `--reset-scope full` pour supprimer également l'espace de travail.
-   Si la configuration est invalide ou contient des clés obsolètes, l'assistant s'arrête et vous demande d'exécuter `openclaw doctor` avant de continuer.
-   La réinitialisation utilise `trash` et propose des périmètres :
    -   Configuration uniquement
    -   Configuration + identifiants + sessions
    -   Réinitialisation complète (supprime également l'espace de travail)

### Étape 2 : Modèle et authentification

-   La matrice complète des options se trouve dans [Options d'authentification et de modèle](#auth-and-model-options).

### Étape 3 : Espace de travail

-   Par défaut `~/.openclaw/workspace` (configurable).
-   Sème les fichiers d'espace de travail nécessaires pour le rituel d'amorçage du premier démarrage.
-   Organisation de l'espace de travail : [Espace de travail de l'agent](../concepts/agent-workspace.md).

### Étape 4 : Passerelle

-   Demande le port, la liaison, le mode d'authentification et l'exposition tailscale.
-   Recommandé : gardez l'authentification par jeton activée même pour la boucle locale afin que les clients WS locaux doivent s'authentifier.
-   En mode jeton, l'intégration interactive propose :
    -   **Générer/stocker un jeton en clair** (par défaut)
    -   **Utiliser SecretRef** (opt-in)
-   En mode mot de passe, l'intégration interactive prend également en charge le stockage en clair ou via SecretRef.
-   Chemin SecretRef de jeton non interactif : `--gateway-token-ref-env <ENV_VAR>`.
    -   Nécessite une variable d'environnement non vide dans l'environnement du processus d'intégration.
    -   Ne peut pas être combiné avec `--gateway-token`.
-   Désactivez l'authentification uniquement si vous faites entièrement confiance à chaque processus local.
-   Les liaisons non locales nécessitent toujours une authentification.

### Étape 5 : Canaux

-   [WhatsApp](../channels/whatsapp.md) : connexion QR optionnelle
-   [Telegram](../channels/telegram.md) : jeton du bot
-   [Discord](../channels/discord.md) : jeton du bot
-   [Google Chat](../channels/googlechat.md) : JSON de compte de service + audience du webhook
-   Plugin [Mattermost](../channels/mattermost.md) : jeton du bot + URL de base
-   [Signal](../channels/signal.md) : installation optionnelle de `signal-cli` + configuration du compte
-   [BlueBubbles](../channels/bluebubbles.md) : recommandé pour iMessage ; URL du serveur + mot de passe + webhook
-   [iMessage](../channels/imessage.md) : chemin CLI `imsg` hérité + accès à la base de données
-   Sécurité des messages privés : le comportement par défaut est l'appairage. Le premier message privé envoie un code ; approuvez via `openclaw pairing approve  ` ou utilisez des listes d'autorisation.

### Étape 6 : Installation du démon

-   macOS : LaunchAgent
    -   Nécessite une session utilisateur connectée ; pour un système sans interface, utilisez un LaunchDaemon personnalisé (non fourni).
-   Linux et Windows via WSL2 : unité utilisateur systemd
    -   L'assistant tente `loginctl enable-linger ` pour que la passerelle reste active après la déconnexion.
    -   Peut demander sudo (écrit dans `/var/lib/systemd/linger`) ; il essaie d'abord sans sudo.
-   Sélection de l'environnement d'exécution : Node (recommandé ; requis pour WhatsApp et Telegram). Bun n'est pas recommandé.

### Étape 7 : Contrôle d'intégrité

-   Démarre la passerelle (si nécessaire) et exécute `openclaw health`.
-   `openclaw status --deep` ajoute des sondes de santé de la passerelle à la sortie de statut.

### Étape 8 : Compétences

-   Lit les compétences disponibles et vérifie les prérequis.
-   Vous permet de choisir le gestionnaire de nœuds : npm ou pnpm (bun non recommandé).
-   Installe les dépendances optionnelles (certaines utilisent Homebrew sur macOS).

### Étape 9 : Fin

-   Résumé et prochaines étapes, y compris les options d'application iOS, Android et macOS.

 

> **ℹ️** Si aucune interface graphique n'est détectée, l'assistant affiche des instructions de transfert de port SSH pour l'interface de contrôle au lieu d'ouvrir un navigateur. Si les ressources de l'interface de contrôle sont manquantes, l'assistant tente de les construire ; la solution de repli est `pnpm ui:build` (installe automatiquement les dépendances de l'interface).

## Détails du mode distant

Le mode distant configure cette machine pour se connecter à une passerelle ailleurs.

> **ℹ️** Le mode distant n'installe ni ne modifie quoi que ce soit sur l'hôte distant.

 Ce que vous configurez :

-   URL de la passerelle distante (`ws://...`)
-   Jeton si l'authentification de la passerelle distante est requise (recommandé)

> **ℹ️** -   Si la passerelle est en boucle locale uniquement, utilisez le tunneling SSH ou un tailnet.
> -   Indices de découverte :
>     -   macOS : Bonjour (`dns-sd`)
>     -   Linux : Avahi (`avahi-browse`)

## Options d'authentification et de modèle

Utilise `ANTHROPIC_API_KEY` s'il est présent ou demande une clé, puis l'enregistre pour une utilisation par le démon.

-   macOS : vérifie l'élément du trousseau "Claude Code-credentials"
-   Linux et Windows : réutilise `~/.claude/.credentials.json` s'il est présent

Sur macOS, choisissez "Toujours autoriser" pour que les démarrages par launchd ne soient pas bloqués.

Exécutez `claude setup-token` sur n'importe quelle machine, puis collez le jeton. Vous pouvez le nommer ; laisser vide utilise la valeur par défaut.

Si `~/.codex/auth.json` existe, l'assistant peut le réutiliser.

Flux navigateur ; collez `code#state`.Définit `agents.defaults.model` sur `openai-codex/gpt-5.4` lorsque le modèle n'est pas défini ou est `openai/*`.

Utilise `OPENAI_API_KEY` s'il est présent ou demande une clé, puis stocke l'identifiant dans les profils d'authentification.Définit `agents.defaults.model` sur `openai/gpt-5.1-codex` lorsque le modèle n'est pas défini, est `openai/*`, ou `openai-codex/*`.

Demande `XAI_API_KEY` et configure xAI comme fournisseur de modèle.

Demande `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`). URL de configuration : [opencode.ai/auth](https://opencode.ai/auth).

Stocke la clé pour vous.

Demande `AI_GATEWAY_API_KEY`. Plus de détails : [Vercel AI Gateway](../providers/vercel-ai-gateway.md).

Demande l'ID de compte, l'ID de passerelle et `CLOUDFLARE_AI_GATEWAY_API_KEY`. Plus de détails : [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md).

La configuration est écrite automatiquement. Plus de détails : [MiniMax](../providers/minimax.md).

Demande `SYNTHETIC_API_KEY`. Plus de détails : [Synthetic](../providers/synthetic.md).

Les configurations Moonshot (Kimi K2) et Kimi Coding sont écrites automatiquement. Plus de détails : [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md).

Fonctionne avec les points de terminaison compatibles OpenAI et compatibles Anthropic.L'intégration interactive prend en charge les mêmes choix de stockage de clé API que les autres flux de clé API de fournisseur :

-   **Coller la clé API maintenant** (en clair)
-   **Utiliser une référence secrète** (référence d'environnement ou référence de fournisseur configurée, avec validation préalable)

Indicateurs non interactifs :

-   `--auth-choice custom-api-key`
-   `--custom-base-url`
-   `--custom-model-id`
-   `--custom-api-key` (optionnel ; revient à `CUSTOM_API_KEY`)
-   `--custom-provider-id` (optionnel)
-   `--custom-compatibility <openai|anthropic>` (optionnel ; par défaut `openai`)

Laisse l'authentification non configurée.

 Comportement du modèle :

-   Choisissez le modèle par défaut parmi les options détectées, ou entrez manuellement le fournisseur et le modèle.
-   L'assistant exécute une vérification du modèle et avertit si le modèle configuré est inconnu ou manque d'authentification.

Chemins des identifiants et profils :

-   Identifiants OAuth : `~/.openclaw/credentials/oauth.json`
-   Profils d'authentification (clés API + OAuth) : `~/.openclaw/agents//agent/auth-profiles.json`

Mode de stockage des identifiants :

-   Le comportement d'intégration par défaut persiste les clés API en tant que valeurs en clair dans les profils d'authentification.
-   `--secret-input-mode ref` active le mode référence au lieu du stockage de clé en clair. Dans l'intégration interactive, vous pouvez choisir soit :
    -   une référence de variable d'environnement (par exemple `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`)
    -   une référence de fournisseur configurée (`file` ou `exec`) avec un alias de fournisseur + un identifiant
-   Le mode référence interactif exécute une validation préalable rapide avant l'enregistrement.
    -   Références d'environnement : valide le nom de la variable + une valeur non vide dans l'environnement d'intégration actuel.
    -   Références de fournisseur : valide la configuration du fournisseur et résout l'identifiant demandé.
    -   Si la validation préalable échoue, l'intégration affiche l'erreur et vous permet de réessayer.
-   En mode non interactif, `--secret-input-mode ref` est uniquement basé sur l'environnement.
    -   Définissez la variable d'environnement du fournisseur dans l'environnement du processus d'intégration.
    -   Les indicateurs de clé en ligne (par exemple `--openai-api-key`) nécessitent que cette variable d'environnement soit définie ; sinon, l'intégration échoue rapidement.
    -   Pour les fournisseurs personnalisés, le mode `ref` non interactif stocke `models.providers..apiKey` comme `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.
    -   Dans ce cas de fournisseur personnalisé, `--custom-api-key` nécessite que `CUSTOM_API_KEY` soit défini ; sinon, l'intégration échoue rapidement.
-   Les identifiants d'authentification de la passerelle prennent en charge les choix en clair et SecretRef dans l'intégration interactive :
    -   Mode jeton : **Générer/stocker un jeton en clair** (par défaut) ou **Utiliser SecretRef**.
    -   Mode mot de passe : en clair ou SecretRef.
-   Chemin SecretRef de jeton non interactif : `--gateway-token-ref-env <ENV_VAR>`.
-   Les configurations en clair existantes continuent de fonctionner sans modification.

> **ℹ️** Astuce pour les systèmes sans interface et serveurs : complétez l'OAuth sur une machine avec un navigateur, puis copiez `~/.openclaw/credentials/oauth.json` (ou `$OPENCLAW_STATE_DIR/credentials/oauth.json`) sur l'hôte de la passerelle.

## Sorties et fonctionnement interne

Champs typiques dans `~/.openclaw/openclaw.json` :

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (si Minimax est choisi)
-   `tools.profile` (l'intégration locale utilise par défaut `"coding"` lorsqu'il n'est pas défini ; les valeurs explicites existantes sont conservées)
-   `gateway.*` (mode, liaison, authentification, tailscale)
-   `session.dmScope` (l'intégration locale utilise par défaut `per-channel-peer` lorsqu'il n'est pas défini ; les valeurs explicites existantes sont conservées)
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Listes d'autorisation de canaux (Slack, Discord, Matrix, Microsoft Teams) lorsque vous optez pour elles pendant les invites (les noms sont résolus en ID lorsque possible)
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` écrit `agents.list[]` et éventuellement `bindings`. Les identifiants WhatsApp vont sous `~/.openclaw/credentials/whatsapp//`. Les sessions sont stockées sous `~/.openclaw/agents//sessions/`.

> **ℹ️** Certains canaux sont livrés sous forme de plugins. Lorsqu'ils sont sélectionnés pendant l'intégration, l'assistant demande d'installer le plugin (npm ou chemin local) avant la configuration du canal.

 RPC de l'assistant de passerelle :

-   `wizard.start`
-   `wizard.next`
-   `wizard.cancel`
-   `wizard.status`

Les clients (application macOS et interface de contrôle) peuvent afficher les étapes sans réimplémenter la logique d'intégration. Comportement de configuration de Signal :

-   Télécharge l'asset de version approprié
-   Le stocke sous `~/.openclaw/tools/signal-cli//`
-   Écrit `channels.signal.cliPath` dans la configuration
-   Les builds JVM nécessitent Java 21
-   Les builds natifs sont utilisés lorsqu'ils sont disponibles
-   Windows utilise WSL2 et suit le flux signal-cli Linux à l'intérieur de WSL

## Documentation associée

-   Centre d'intégration : [Assistant d'intégration (CLI)](./wizard.md)
-   Automatisation et scripts : [Automatisation CLI](./wizard-cli-automation.md)
-   Référence des commandes : [`openclaw onboard`](../cli/onboard.md)

[Configuration de l'assistant personnel](./openclaw.md)[Automatisation CLI](./wizard-cli-automation.md)