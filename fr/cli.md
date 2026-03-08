

  Commandes CLI

  
# Référence CLI

Cette page décrit le comportement actuel du CLI. Si les commandes changent, mettez à jour cette documentation.

## Pages de commandes

-   [`setup`](./cli/setup.md)
-   [`onboard`](./cli/onboard.md)
-   [`configure`](./cli/configure.md)
-   [`config`](./cli/config.md)
-   [`completion`](./cli/completion.md)
-   [`doctor`](./cli/doctor.md)
-   [`dashboard`](./cli/dashboard.md)
-   [`reset`](./cli/reset.md)
-   [`uninstall`](./cli/uninstall.md)
-   [`update`](./cli/update.md)
-   [`message`](./cli/message.md)
-   [`agent`](./cli/agent.md)
-   [`agents`](./cli/agents.md)
-   [`acp`](./cli/acp.md)
-   [`status`](./cli/status.md)
-   [`health`](./cli/health.md)
-   [`sessions`](./cli/sessions.md)
-   [`gateway`](./cli/gateway.md)
-   [`logs`](./cli/logs.md)
-   [`system`](./cli/system.md)
-   [`models`](./cli/models.md)
-   [`memory`](./cli/memory.md)
-   [`directory`](./cli/directory.md)
-   [`nodes`](./cli/nodes.md)
-   [`devices`](./cli/devices.md)
-   [`node`](./cli/node.md)
-   [`approvals`](./cli/approvals.md)
-   [`sandbox`](./cli/sandbox.md)
-   [`tui`](./cli/tui.md)
-   [`browser`](./cli/browser.md)
-   [`cron`](./cli/cron.md)
-   [`dns`](./cli/dns.md)
-   [`docs`](./cli/docs.md)
-   [`hooks`](./cli/hooks.md)
-   [`webhooks`](./cli/webhooks.md)
-   [`pairing`](./cli/pairing.md)
-   [`qr`](./cli/qr.md)
-   [`plugins`](./cli/plugins.md) (commandes plugin)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (alias historique pour les commandes de service gateway)
-   [`clawbot`](./cli/clawbot.md) (espace de noms historique)
-   [`voicecall`](./cli/voicecall.md) (plugin ; si installé)

## Options globales

-   `--dev` : isole l'état sous `~/.openclaw-dev` et décale les ports par défaut.
-   `--profile ` : isole l'état sous `~/.openclaw-`.
-   `--no-color` : désactive les couleurs ANSI.
-   `--update` : raccourci pour `openclaw update` (installations depuis la source uniquement).
-   `-V`, `--version`, `-v` : affiche la version et quitte.

## Style de sortie

-   Les couleurs ANSI et les indicateurs de progression ne s'affichent que dans les sessions TTY.
-   Les hyperliens OSC-8 s'affichent comme des liens cliquables dans les terminaux qui les supportent ; sinon, nous revenons aux URL simples.
-   `--json` (et `--plain` là où supporté) désactive le style pour une sortie propre.
-   `--no-color` désactive le style ANSI ; `NO_COLOR=1` est également respecté.
-   Les commandes de longue durée affichent un indicateur de progression (OSC 9;4 quand supporté).

## Palette de couleurs

OpenClaw utilise une palette "lobster" pour la sortie CLI.

-   `accent` (#FF5A2D) : titres, étiquettes, surbrillances principales.
-   `accentBright` (#FF7A3D) : noms de commandes, emphase.
-   `accentDim` (#D14A22) : texte de surbrillance secondaire.
-   `info` (#FF8A5B) : valeurs informatives.
-   `success` (#2FBF71) : états de succès.
-   `warn` (#FFB020) : avertissements, solutions de repli, attention.
-   `error` (#E23D2D) : erreurs, échecs.
-   `muted` (#8B7F77) : désaccentuation, métadonnées.

Source de vérité de la palette : `src/terminal/palette.ts` (alias "lobster seam").

## Arborescence des commandes

```bash
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

Note : les plugins peuvent ajouter des commandes de premier niveau supplémentaires (par exemple `openclaw voicecall`).

## Sécurité

-   `openclaw security audit` — audit de la configuration + état local pour les erreurs de sécurité courantes.
-   `openclaw security audit --deep` — sonde live de la Gateway au mieux possible.
-   `openclaw security audit --fix` — renforce les paramètres par défaut sécurisés et chmod l'état/configuration.

## Secrets

-   `openclaw secrets reload` — re-résout les références et échange atomiquement l'instantané d'exécution.
-   `openclaw secrets audit` — recherche les résidus en texte clair, les références non résolues et les dérives de priorité.
-   `openclaw secrets configure` — assistant interactif pour la configuration du fournisseur + mappage SecretRef + prévol/application.
-   `openclaw secrets apply --from <plan.json>` — applique un plan généré précédemment (`--dry-run` supporté).

## Plugins

Gérez les extensions et leur configuration :

-   `openclaw plugins list` — découvrez les plugins (utilisez `--json` pour une sortie machine).
-   `openclaw plugins info ` — affiche les détails d'un plugin.
-   `openclaw plugins install <path|.tgz|npm-spec>` — installe un plugin (ou ajoute un chemin de plugin à `plugins.load.paths`).
-   `openclaw plugins enable ` / `disable ` — active/désactive `plugins.entries..enabled`.
-   `openclaw plugins doctor` — signale les erreurs de chargement de plugin.

La plupart des changements de plugin nécessitent un redémarrage de la gateway. Voir [/plugin](./tools/plugin.md).

## Mémoire

Recherche vectorielle sur `MEMORY.md` + `memory/*.md` :

-   `openclaw memory status` — affiche les statistiques de l'index.
-   `openclaw memory index` — réindexe les fichiers de mémoire.
-   `openclaw memory search "<requête>"` (ou `--query "<requête>"`) — recherche sémantique sur la mémoire.

## Commandes slash de chat

Les messages de chat supportent les commandes `/...` (texte et natives). Voir [/tools/slash-commands](./tools/slash-commands.md). Points forts :

-   `/status` pour un diagnostic rapide.
-   `/config` pour les changements de configuration persistants.
-   `/debug` pour les surcharges de configuration en temps d'exécution uniquement (mémoire, pas disque ; nécessite `commands.debug: true`).

## Installation + intégration

### setup

Initialise la configuration + l'espace de travail. Options :

-   `--workspace <répertoire>` : chemin de l'espace de travail de l'agent (par défaut `~/.openclaw/workspace`).
-   `--wizard` : exécute l'assistant d'intégration.
-   `--non-interactive` : exécute l'assistant sans invites.
-   `--mode <local|remote>` : mode de l'assistant.
-   `--remote-url ` : URL de la Gateway distante.
-   `--remote-token ` : token de la Gateway distante.

L'assistant s'exécute automatiquement quand des options d'assistant sont présentes (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### onboard

Assistant interactif pour configurer la gateway, l'espace de travail et les compétences. Options :

-   `--workspace <répertoire>`
-   `--reset` (réinitialise la configuration + les identifiants + les sessions avant l'assistant)
-   `--reset-scope <config|config+creds+sessions|full>` (par défaut `config+creds+sessions` ; utilisez `full` pour supprimer aussi l'espace de travail)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (manual est un alias pour advanced)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (non interactif ; utilisé avec `--auth-choice token`)
-   `--token ` (non interactif ; utilisé avec `--auth-choice token`)
-   `--token-profile-id ` (non interactif ; par défaut : `:manual`)
-   `--token-expires-in <durée>` (non interactif ; par ex. `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (par défaut `plaintext` ; utilisez `ref` pour stocker les références d'environnement par défaut du fournisseur au lieu des clés en texte clair)
-   `--anthropic-api-key <clé>`
-   `--openai-api-key <clé>`
-   `--mistral-api-key <clé>`
-   `--openrouter-api-key <clé>`
-   `--ai-gateway-api-key <clé>`
-   `--moonshot-api-key <clé>`
-   `--kimi-code-api-key <clé>`
-   `--gemini-api-key <clé>`
-   `--zai-api-key <clé>`
-   `--minimax-api-key <clé>`
-   `--opencode-zen-api-key <clé>`
-   `--custom-base-url ` (non interactif ; utilisé avec `--auth-choice custom-api-key`)
-   `--custom-model-id ` (non interactif ; utilisé avec `--auth-choice custom-api-key`)
-   `--custom-api-key <clé>` (non interactif ; optionnel ; utilisé avec `--auth-choice custom-api-key` ; utilise `CUSTOM_API_KEY` si omis)
-   `--custom-provider-id ` (non interactif ; identifiant de fournisseur personnalisé optionnel)
-   `--custom-compatibility <openai|anthropic>` (non interactif ; optionnel ; par défaut `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (non interactif ; stocke `gateway.auth.token` comme une SecretRef d'environnement ; nécessite que la variable d'env soit définie ; ne peut pas être combiné avec `--gateway-token`)
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (alias : `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (pnpm recommandé ; bun non recommandé pour le runtime Gateway)
-   `--json`

### configure

Assistant de configuration interactif (modèles, canaux, compétences, gateway).

### config

Assistants de configuration non interactifs (get/set/unset/file/validate). Exécuter `openclaw config` sans sous-commande lance l'assistant. Sous-commandes :

-   `config get ` : affiche une valeur de configuration (chemin pointé/crocheté).
-   `config set  ` : définit une valeur (JSON5 ou chaîne brute).
-   `config unset ` : supprime une valeur.
-   `config file` : affiche le chemin du fichier de configuration actif.
-   `config validate` : valide la configuration actuelle par rapport au schéma sans démarrer la gateway.
-   `config validate --json` : émet une sortie JSON lisible par machine.

### doctor

Contrôles de santé + corrections rapides (configuration + gateway + services historiques). Options :

-   `--no-workspace-suggestions` : désactive les suggestions de mémoire de l'espace de travail.
-   `--yes` : accepte les valeurs par défaut sans invite (sans interface).
-   `--non-interactive` : ignore les invites ; applique uniquement les migrations sûres.
-   `--deep` : scanne les services système pour des installations supplémentaires de gateway.

## Assistants de canaux

### channels

Gérez les comptes de canaux de discussion (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams). Sous-commandes :

-   `channels list` : affiche les canaux configurés et les profils d'authentification.
-   `channels status` : vérifie l'accessibilité de la gateway et la santé des canaux (`--probe` exécute des vérifications supplémentaires ; utilisez `openclaw health` ou `openclaw status --deep` pour les sondes de santé de la gateway).
-   Astuce : `channels status` affiche des avertissements avec des corrections suggérées quand il peut détecter des configurations incorrectes courantes (puis vous oriente vers `openclaw doctor`).
-   `channels logs` : affiche les journaux récents des canaux depuis le fichier journal de la gateway.
-   `channels add` : configuration de type assistant quand aucune option n'est passée ; les options passent en mode non interactif.
    -   Lors de l'ajout d'un compte non par défaut à un canal utilisant encore la configuration de premier niveau à compte unique, OpenClaw déplace les valeurs limitées au compte dans `channels..accounts.default` avant d'écrire le nouveau compte.
    -   `channels add` non interactif ne crée/mette pas à niveau automatiquement les liaisons ; les liaisons limitées au canal continuent de correspondre au compte par défaut.
-   `channels remove` : désactive par défaut ; passez `--delete` pour supprimer les entrées de configuration sans invites.
-   `channels login` : connexion interactive au canal (WhatsApp Web uniquement).
-   `channels logout` : déconnexion d'une session de canal (si supporté).

Options communes :

-   `--channel ` : `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account ` : identifiant du compte de canal (par défaut `default`)
-   `--name <étiquette>` : nom d'affichage pour le compte

Options `channels login` :

-   `--channel ` (par défaut `whatsapp` ; supporte `whatsapp`/`web`)
-   `--account `
-   `--verbose`

Options `channels logout` :

-   `--channel ` (par défaut `whatsapp`)
-   `--account `

Options `channels list` :

-   `--no-usage` : ignore les instantanés d'utilisation/quota des fournisseurs de modèles (OAuth/API uniquement).
-   `--json` : sortie JSON (inclut l'utilisation sauf si `--no-usage` est défini).

Options `channels logs` :

-   `--channel <nom|all>` (par défaut `all`)
-   `--lines ` (par défaut `200`)
-   `--json`

Plus de détails : [/concepts/oauth](./concepts/oauth.md) Exemples :

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

Listez et inspectez les compétences disponibles ainsi que les informations de préparation. Sous-commandes :

-   `skills list` : liste les compétences (par défaut quand aucune sous-commande).
-   `skills info ` : affiche les détails d'une compétence.
-   `skills check` : résumé des compétences prêtes vs exigences manquantes.

Options :

-   `--eligible` : affiche uniquement les compétences prêtes.
-   `--json` : sortie JSON (pas de style).
-   `-v`, `--verbose` : inclut les détails des exigences manquantes.

Astuce : utilisez `npx clawhub` pour rechercher, installer et synchroniser les compétences.

### pairing

Approuvez les demandes d'appariement en message privé à travers les canaux. Sous-commandes :

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

Gérez les entrées d'appariement d'appareils de la gateway et les jetons d'appareil par rôle. Sous-commandes :

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role <rôle> [--scope <portée...>]`
-   `devices revoke --device  --role <rôle>`

### webhooks gmail

Configuration + exécution du hook Gmail Pub/Sub. Voir [/automation/gmail-pubsub](./automation/gmail-pubsub.md). Sous-commandes :

-   `webhooks gmail setup` (nécessite `--account ` ; supporte `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
-   `webhooks gmail run` (surcharges d'exécution pour les mêmes options)

### dns setup

Assistant DNS de découverte à large zone (CoreDNS + Tailscale). Voir [/gateway/discovery](./gateway/discovery.md). Options :

-   `--apply` : installe/met à jour la configuration CoreDNS (nécessite sudo ; macOS uniquement).

## Messagerie + agent

### message

Messagerie sortante unifiée + actions de canal. Voir : [/cli/message](./cli/message.md) Sous-commandes :

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

Exemples :

-   `openclaw message send --target +15555550123 --message "Salut"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "Snack ?" --poll-option Pizza --poll-option Sushi`

### agent

Exécute un tour d'agent via la Gateway (ou `--local` intégré). Requis :

-   `--message `

Options :

-   `--to ` (pour la clé de session et la livraison optionnelle)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (modèles GPT-5.2 + Codex uniquement)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

Gérez les agents isolés (espaces de travail + authentification + routage).

#### agents list

Liste les agents configurés. Options :

-   `--json`
-   `--bindings`

#### agents add \[nom\]

Ajoute un nouvel agent isolé. Exécute l'assistant guidé sauf si des options (ou `--non-interactive`) sont passées ; `--workspace` est requis en mode non interactif. Options :

-   `--workspace <répertoire>`
-   `--model `
-   `--agent-dir <répertoire>`
-   `--bind <canal[:accountId]>` (répétable)
-   `--non-interactive`
-   `--json`

Les spécifications de liaison utilisent `canal[:accountId]`. Quand `accountId` est omis, OpenClaw peut résoudre la portée du compte via les valeurs par défaut du canal/les hooks du plugin ; sinon c'est une liaison de canal sans portée de compte explicite.

#### agents bindings

Liste les liaisons de routage. Options :

-   `--agent `
-   `--json`

#### agents bind

Ajoute des liaisons de routage pour un agent. Options :

-   `--agent `
-   `--bind <canal[:accountId]>` (répétable)
-   `--json`

#### agents unbind

Supprime des liaisons de routage pour un agent. Options :

-   `--agent `
-   `--bind <canal[:accountId]>` (répétable)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

Supprime un agent et nettoie son espace de travail + état. Options :

-   `--force`
-   `--json`

### acp

Exécute le pont ACP qui connecte les IDEs à la Gateway. Voir [`acp`](./cli/acp.md) pour toutes les options et exemples.

### status

Affiche la santé des sessions liées et les destinataires récents. Options :

-   `--json`
-   `--all` (diagnostic complet ; lecture seule, collable)
-   `--deep` (sonde les canaux)
-   `--usage` (affiche l'utilisation/quota des fournisseurs de modèles)
-   `--timeout `
-   `--verbose`
-   `--debug` (alias pour `--verbose`)

Notes :

-   L'aperçu inclut le statut de la Gateway + du service hôte du nœud quand disponible.

### Suivi de l'utilisation

OpenClaw peut afficher l'utilisation/quota des fournisseurs quand les identifiants OAuth/API sont disponibles. Affiche :

-   `/status` (ajoute une courte ligne d'utilisation du fournisseur quand disponible)
-   `openclaw status --usage` (affiche la répartition complète des fournisseurs)
-   Barre de menus macOS (section Utilisation sous Contexte)

Notes :

-   Les données proviennent directement des points de terminaison d'utilisation des fournisseurs (pas d'estimations).
-   Fournisseurs : Anthropic, GitHub Copilot, OpenAI Codex OAuth, plus Gemini CLI/Antigravity quand ces plugins de fournisseur sont activés.
-   Si aucune identifiants correspondants n'existent, l'utilisation est masquée.
-   Détails : voir [Suivi de l'utilisation](./concepts/usage-tracking.md).

### health

Récupère l'état de santé de la Gateway en cours d'exécution. Options :

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

Liste les sessions de conversation stockées. Options :

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## Réinitialisation / Désinstallation

### reset

Réinitialise la configuration/état local (conserve le CLI installé). Options :

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notes :

-   `--non-interactive` nécessite `--scope` et `--yes`.

### uninstall

Désinstalle le service gateway + les données locales (le CLI reste). Options :

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notes :

-   `--non-interactive` nécessite `--yes` et des portées explicites (ou `--all`).

## Gateway

### gateway

Exécute la Gateway WebSocket. Options :

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (réinitialise la configuration dev + les identifiants + les sessions + l'espace de travail)
-   `--force` (tue l'écouteur existant sur le port)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (alias pour `--ws-log compact`)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

Gère le service Gateway (launchd/systemd/schtasks). Sous-commandes :

-   `gateway status` (sonde la RPC de la Gateway par défaut)
-   `gateway install` (installation du service)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

Notes :

-   `gateway status` sonde la RPC de la Gateway par défaut en utilisant le port/config résolu du service (surchargez avec `--url/--token/--password`).
-   `gateway status` supporte `--no-probe`, `--deep`, et `--json` pour le scriptage.
-   `gateway status` signale également les services gateway historiques ou supplémentaires quand il peut les détecter (`--deep` ajoute des scans au niveau système). Les services OpenClaw nommés par profil sont traités comme de premier ordre et ne sont pas signalés comme "supplémentaires".
-   `gateway status` affiche quel chemin de configuration le CLI utilise vs quelle configuration le service utilise probablement (env du service), plus l'URL cible de la sonde résolue.
-   `gateway install|uninstall|start|stop|restart` supporte `--json` pour le scriptage (la sortie par défaut reste conviviale).
-   `gateway install` utilise par défaut le runtime Node ; bun est **non recommandé** (bugs WhatsApp/Telegram).
-   Options `gateway install` : `--port`, `--runtime`, `--token`, `--force`, `--json`.

### logs

Affiche la fin des journaux de fichiers de la Gateway via RPC. Notes :

-   Les sessions TTY affichent une vue colorée et structurée ; les non-TTY reviennent au texte brut.
-   `--json` émet du JSON délimité par des lignes (un événement de journal par ligne).

Exemples :

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### gateway &lt;sous-commande&gt;

Assistants CLI de la Gateway (utilisez `--url`, `--token`, `--password`, `--timeout`, `--expect-final` pour les sous-commandes RPC). Quand vous passez `--url`, le CLI n'applique pas automatiquement la configuration ou les identifiants d'environnement. Incluez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur. Sous-commandes :

-   `gateway call <méthode> [--params ]`
-   `gateway health`
-   `gateway status`
-   `gateway probe`
-   `gateway discover`
-   `gateway install|uninstall|start|stop|restart`
-   `gateway run`

RPCs courantes :

-   `config.apply` (valide + écrit la configuration + redémarre + réveille)
-   `config.patch` (fusionne une mise à jour partielle + redémarre + réveille)
-   `update.run` (exécute la mise