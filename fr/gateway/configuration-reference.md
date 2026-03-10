title: "Guide de référence et de configuration de la passerelle OpenClaw"
description: "Référence complète de tous les champs de configuration de la passerelle OpenClaw. Apprenez à configurer les canaux, les politiques de MP, les remplacements de modèles, et à configurer WhatsApp, Telegram et Discord."
keywords: ["configuration openclaw", "configuration de la passerelle", "configuration des canaux", "politique de mp", "remplacements de modèles", "bot whatsapp", "bot telegram", "bot discord"]
---

  Configuration et opérations

  
# Référence de configuration

Tous les champs disponibles dans `~/.openclaw/openclaw.json`. Pour une vue orientée tâches, consultez [Configuration](./configuration.md). Le format de configuration est **JSON5** (commentaires + virgules finales autorisées). Tous les champs sont optionnels — OpenClaw utilise des valeurs par défaut sûres lorsqu'ils sont omis.

* * *

## Canaux

Chaque canal démarre automatiquement lorsque sa section de configuration existe (sauf si `enabled: false`).

### Accès en MP et groupes

Tous les canaux prennent en charge les politiques de MP et les politiques de groupe :

| Politique de MP | Comportement |
| --- | --- |
| `pairing` (par défaut) | Les expéditeurs inconnus reçoivent un code d'appairage unique ; le propriétaire doit approuver |
| `allowlist` | Seuls les expéditeurs dans `allowFrom` (ou le magasin d'autorisations appairé) |
| `open` | Autoriser tous les MP entrants (nécessite `allowFrom: ["*"]`) |
| `disabled` | Ignorer tous les MP entrants |

| Politique de groupe | Comportement |
| --- | --- |
| `allowlist` (par défaut) | Seuls les groupes correspondant à la liste d'autorisation configurée |
| `open` | Contourner les listes d'autorisation de groupe (la mention conditionnelle s'applique toujours) |
| `disabled` | Bloquer tous les messages de groupe/salle |

> **ℹ️** `channels.defaults.groupPolicy` définit la valeur par défaut lorsque `groupPolicy` d'un fournisseur n'est pas défini. Les codes d'appairage expirent après 1 heure. Les demandes d'appairage de MP en attente sont limitées à **3 par canal**. Si un bloc de fournisseur est entièrement absent (`channels.` absent), la politique de groupe au runtime revient à `allowlist` (échec fermé) avec un avertissement au démarrage.

### Remplacements de modèle par canal

Utilisez `channels.modelByChannel` pour épingler des identifiants de canal spécifiques à un modèle. Les valeurs acceptent `provider/model` ou des alias de modèle configurés. Le mappage de canal s'applique lorsqu'une session n'a pas déjà de remplacement de modèle (par exemple, défini via `/model`).

```json
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
```

### Valeurs par défaut des canaux et pulsation

Utilisez `channels.defaults` pour un comportement partagé de politique de groupe et de pulsation entre les fournisseurs :

```json
{
  channels: {
    defaults: {
      groupPolicy: "allowlist", // open | allowlist | disabled
      heartbeat: {
        showOk: false,
        showAlerts: true,
        useIndicator: true,
      },
    },
  },
}
```

-   `channels.defaults.groupPolicy` : politique de groupe de secours lorsque `groupPolicy` au niveau du fournisseur n'est pas défini.
-   `channels.defaults.heartbeat.showOk` : inclure les statuts de canal sains dans la sortie de pulsation.
-   `channels.defaults.heartbeat.showAlerts` : inclure les statuts dégradés/erreurs dans la sortie de pulsation.
-   `channels.defaults.heartbeat.useIndicator` : afficher une sortie de pulsation compacte de type indicateur.

### WhatsApp

WhatsApp fonctionne via le canal web de la passerelle (Baileys Web). Il démarre automatiquement lorsqu'une session liée existe.

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // coches bleues (false en mode self-chat)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

```json
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

-   Les commandes sortantes utilisent par défaut le compte `default` s'il est présent ; sinon le premier identifiant de compte configuré (trié).
-   `channels.whatsapp.defaultAccount` optionnel remplace cette sélection de compte par défaut de secours lorsqu'il correspond à un identifiant de compte configuré.
-   Le répertoire d'authentification Baileys à compte unique hérité est migré par `openclaw doctor` dans `whatsapp/default`.
-   Remplacements par compte : `channels.whatsapp.accounts..sendReadReceipts`, `channels.whatsapp.accounts..dmPolicy`, `channels.whatsapp.accounts..allowFrom`.

### Telegram

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Gardez les réponses courtes.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Restez sur le sujet.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Sauvegarde Git" },
        { command: "generate", description: "Créer une image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (par défaut : off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 100,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

-   Jeton du bot : `channels.telegram.botToken` ou `channels.telegram.tokenFile`, avec `TELEGRAM_BOT_TOKEN` comme secours pour le compte par défaut.
-   `channels.telegram.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   Dans les configurations multi-comptes (2+ identifiants de compte), définissez un compte par défaut explicite (`channels.telegram.defaultAccount` ou `channels.telegram.accounts.default`) pour éviter le routage de secours ; `openclaw doctor` avertit lorsque ceci est manquant ou invalide.
-   `configWrites: false` bloque les écritures de configuration initiées par Telegram (migrations d'ID de supergroupe, `/config set|unset`).
-   Les entrées `bindings[]` de haut niveau avec `type: "acp"` configurent des liaisons ACP persistantes pour les sujets de forum (utilisez `chatId:topic:topicId` canonique dans `match.peer.id`). La sémantique des champs est partagée dans [ACP Agents](../tools/acp-agents.md#channel-specific-settings).
-   Les aperçus de flux Telegram utilisent `sendMessage` + `editMessageText` (fonctionne dans les discussions directes et de groupe).
-   Politique de nouvelle tentative : voir [Politique de nouvelle tentative](../concepts/retry.md).

### Discord

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          ignoreOtherMentions: true,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Réponses courtes uniquement.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress correspond à partial sur Discord)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in pour sessions_spawn({ thread: true })
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

-   Jeton : `channels.discord.token`, avec `DISCORD_BOT_TOKEN` comme secours pour le compte par défaut.
-   `channels.discord.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   Utilisez `user:` (MP) ou `channel:` (canal de guilde) pour les cibles de livraison ; les identifiants numériques nus sont rejetés.
-   Les slugs de guilde sont en minuscules avec les espaces remplacés par `-` ; les clés de canal utilisent le nom slugifié (pas de `#`). Préférez les identifiants de guilde.
-   Les messages écrits par le bot sont ignorés par défaut. `allowBots: true` les active ; utilisez `allowBots: "mentions"` pour n'accepter que les messages de bot qui mentionnent le bot (les propres messages sont toujours filtrés).
-   `channels.discord.guilds..ignoreOtherMentions` (et les remplacements de canal) supprime les messages qui mentionnent un autre utilisateur ou rôle mais pas le bot (excluant @everyone/@here).
-   `maxLinesPerMessage` (par défaut 17) divise les messages longs même en dessous de 2000 caractères.
-   `channels.discord.threadBindings` contrôle le routage lié aux fils Discord :
    -   `enabled` : remplacement Discord pour les fonctionnalités de session liées aux fils (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`, et livraison/routage lié)
    -   `idleHours` : remplacement Discord pour le désengagement automatique d'inactivité en heures (`0` désactive)
    -   `maxAgeHours` : remplacement Discord pour l'âge maximum absolu en heures (`0` désactive)
    -   `spawnSubagentSessions` : interrupteur opt-in pour `sessions_spawn({ thread: true })` création/liaison automatique de fil
-   Les entrées `bindings[]` de haut niveau avec `type: "acp"` configurent des liaisons ACP persistantes pour les canaux et fils (utilisez l'identifiant de canal/fil dans `match.peer.id`). La sémantique des champs est partagée dans [ACP Agents](../tools/acp-agents.md#channel-specific-settings).
-   `channels.discord.ui.components.accentColor` définit la couleur d'accent pour les conteneurs de composants Discord v2.
-   `channels.discord.voice` active les conversations dans les canaux vocaux Discord et les remplacements d'auto-rejoindre + TTS optionnels.
-   `channels.discord.voice.daveEncryption` et `channels.discord.voice.decryptionFailureTolerance` sont transmis aux options DAVE de `@discordjs/voice` (`true` et `24` par défaut).
-   OpenClaw tente en plus une récupération de réception vocale en quittant/rejoignant une session vocale après des échecs de décryptage répétés.
-   `channels.discord.streaming` est la clé de mode de flux canonique. Les valeurs héritées `streamMode` et booléenne `streaming` sont auto-migrées.
-   `channels.discord.autoPresence` mappe la disponibilité au runtime à la présence du bot (sain => en ligne, dégradé => inactif, épuisé => ne pas déranger) et permet des remplacements de texte de statut optionnels.
-   `channels.discord.dangerouslyAllowNameMatching` réactive la correspondance de nom/étiquette mutable (mode de compatibilité casse-cou).

**Modes de notification de réaction :** `off` (aucune), `own` (messages du bot, par défaut), `all` (tous les messages), `allowlist` (depuis `guilds..users` sur tous les messages).

### Google Chat

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

-   JSON de compte de service : en ligne (`serviceAccount`) ou basé sur fichier (`serviceAccountFile`).
-   SecretRef de compte de service est également pris en charge (`serviceAccountRef`).
-   Secours d'environnement : `GOOGLE_CHAT_SERVICE_ACCOUNT` ou `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
-   Utilisez `spaces/` ou `users/` pour les cibles de livraison.
-   `channels.googlechat.dangerouslyAllowNameMatching` réactive la correspondance de principal email mutable (mode de compatibilité casse-cou).

### Slack

```json
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Réponses courtes uniquement.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      typingReaction: "hourglass_flowing_sand",
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (mode aperçu)
      nativeStreaming: true, // utiliser l'API de streaming native de Slack lorsque streaming=partial
      mediaMaxMb: 20,
    },
  },
}
```

-   **Mode socket** nécessite à la fois `botToken` et `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` pour le secours d'environnement du compte par défaut).
-   **Mode HTTP** nécessite `botToken` plus `signingSecret` (à la racine ou par compte).
-   `configWrites: false` bloque les écritures de configuration initiées par Slack.
-   `channels.slack.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   `channels.slack.streaming` est la clé de mode de flux canonique. Les valeurs héritées `streamMode` et booléenne `streaming` sont auto-migrées.
-   Utilisez `user:` (MP) ou `channel:` pour les cibles de livraison.

**Modes de notification de réaction :** `off`, `own` (par défaut), `all`, `allowlist` (depuis `reactionAllowlist`). **Isolation de session de fil :** `thread.historyScope` est par fil (par défaut) ou partagé à travers le canal. `thread.inheritParent` copie la transcription du canal parent vers les nouveaux fils.

-   `typingReaction` ajoute une réaction temporaire au message Slack entrant pendant qu'une réponse est en cours d'exécution, puis la retire à la fin. Utilisez un code court d'emoji Slack tel que `"hourglass_flowing_sand"`.

| Groupe d'actions | Par défaut | Notes |
| --- | --- | --- |
| reactions | activé | Réagir + lister les réactions |
| messages | activé | Lire/envoyer/modifier/supprimer |
| pins | activé | Épingler/désépingler/lister |
| memberInfo | activé | Informations sur les membres |
| emojiList | activé | Liste d'emojis personnalisés |

### Mattermost

Mattermost est livré en tant que plugin : `openclaw plugins install @openclaw/mattermost`.

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      commands: {
        native: true, // opt-in
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // URL explicite optionnelle pour les déploiements en reverse-proxy/public
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Modes de discussion : `oncall` (répondre sur @-mention, par défaut), `onmessage` (chaque message), `onchar` (messages commençant par un préfixe déclencheur). Lorsque les commandes natives Mattermost sont activées :

-   `commands.callbackPath` doit être un chemin (par exemple `/api/channels/mattermost/command`), pas une URL complète.
-   `commands.callbackUrl` doit résoudre vers le point de terminaison de la passerelle OpenClaw et être accessible depuis le serveur Mattermost.
-   Pour les hôtes de rappel privés/tailnet/internes, Mattermost peut nécessiter que `ServiceSettings.AllowedUntrustedInternalConnections` inclue l'hôte/domaine de rappel. Utilisez des valeurs d'hôte/domaine, pas des URL complètes.
-   `channels.mattermost.configWrites` : autoriser ou refuser les écritures de configuration initiées par Mattermost.
-   `channels.mattermost.requireMention` : exiger `@mention` avant de répondre dans les canaux.
-   `channels.mattermost.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.

### Signal

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // liaison de compte optionnelle
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Modes de notification de réaction :** `off`, `own` (par défaut), `all`, `allowlist` (depuis `reactionAllowlist`).

-   `channels.signal.account` : épingle le démarrage du canal à une identité de compte Signal spécifique.
-   `channels.signal.configWrites` : autoriser ou refuser les écritures de configuration initiées par Signal.
-   `channels.signal.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.

### BlueBubbles

BlueBubbles est le chemin iMessage recommandé (supporté par plugin, configuré sous `channels.bluebubbles`).

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, contrôles de groupe, et actions avancées :
      // voir /channels/bluebubbles
    },
  },
}
```

-   Chemins de clés principaux couverts ici : `channels.bluebubbles`, `channels.bluebubbles.dmPolicy`.
-   `channels.bluebubbles.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   La configuration complète du canal BlueBubbles est documentée dans [BlueBubbles](../channels/bluebubbles.md).

### iMessage

OpenClaw lance `imsg rpc` (JSON-RPC sur stdio). Aucun démon ou port requis.

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

-   `channels.imessage.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   Nécessite un accès complet au disque à la base de données Messages.
-   Préférez les cibles `chat_id:`. Utilisez `imsg chats --limit 20` pour lister les discussions.
-   `cliPath` peut pointer vers un wrapper SSH ; définissez `remoteHost` (`host` ou `user@host`) pour la récupération de pièces jointes SCP.
-   `attachmentRoots` et `remoteAttachmentRoots` restreignent les chemins de pièces jointes entrants (par défaut : `/Users/*/Library/Messages/Attachments`).
-   SCP utilise une vérification stricte de clé d'hôte, assurez-vous donc que la clé d'hôte de relais existe déjà dans `~/.ssh/known_hosts`.
-   `channels.imessage.configWrites` : autoriser ou refuser les écritures de configuration initiées par iMessage.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Microsoft Teams

Microsoft Teams est supporté par extension et configuré sous `channels.msteams`.

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, politiques par équipe/canal :
      // voir /channels/msteams
    },
  },
}
```

-   Chemins de clés principaux couverts ici : `channels.msteams`, `channels.msteams.configWrites`.
-   La configuration complète de Teams (identifiants, webhook, politique de MP/groupe, remplacements par équipe/par canal) est documentée dans [Microsoft Teams](../channels/msteams.md).

### IRC

IRC est supporté par extension et configuré sous `channels.irc`.

```json
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

-   Chemins de clés principaux couverts ici : `channels.irc`, `channels.irc.dmPolicy`, `channels.irc.configWrites`, `channels.irc.nickserv.*`.
-   `channels.irc.defaultAccount` optionnel remplace la sélection de compte par défaut lorsqu'il correspond à un identifiant de compte configuré.
-   La configuration complète du canal IRC (hôte/port/TLS/canaux/listes d'autorisation/mention conditionnelle) est documentée dans [IRC](../channels/irc.md).

### Multi-compte (tous les canaux)

Exécutez plusieurs comptes par canal (chacun avec son propre `accountId`) :

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Bot principal",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Bot d'alertes",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   `default` est utilisé lorsque `accountId` est omis (CLI + routage).
-   Les jetons d'environnement s'appliquent uniquement au compte **par défaut**.
-   Les paramètres de canal de base s'appliquent à tous les comptes sauf remplacement par compte.
-   Utilisez `bindings[].match.accountId` pour router chaque compte vers un agent différent.
-   Si vous ajoutez un compte non par défaut via `openclaw channels add` (ou l'intégration du canal) tout en étant encore sur une configuration de canal à compte unique de haut niveau, OpenClaw déplace d'abord les valeurs de compte unique de haut niveau dans `channels..accounts.default` pour que le compte original continue de fonctionner.
-   Les liaisons existantes uniquement par canal (sans `accountId`) continuent de correspondre au compte par défaut ; les liaisons par compte restent optionnelles.
-   `openclaw doctor --fix` répare également les formes mixtes en déplaçant les valeurs de compte unique de haut niveau dans `accounts.default` lorsque des comptes nommés existent mais `default` est manquant.

### Autres canaux d'extension

De nombreux canaux d'extension sont configurés comme `channels.` et documentés dans leurs pages de canal dédiées (par exemple Feishu, Matrix, LINE, Nostr, Zalo, Nextcloud Talk, Synology Chat et Twitch). Voir l'index complet des canaux : [Canaux](../channels.md).

### Mention conditionnelle dans les discussions de groupe

Les messages de groupe nécessitent par défaut **une mention** (mention de métadonnées ou motifs regex). S'applique aux discussions de groupe WhatsApp, Telegram, Discord, Google Chat et iMessage. **Types de mention :**

-   **Mentions de métadonnées** : @-mentions natives de la plateforme. Ignorées en mode self-chat WhatsApp.
-   **Motifs de texte** : motifs regex dans `agents.list[].groupChat.mentionPatterns`. Toujours vérifiés.
-   La mention conditionnelle n'est appliquée que lorsque la détection est possible (mentions natives ou au moins un motif).

```json
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` définit la valeur par défaut globale. Les canaux peuvent remplacer avec `channels..historyLimit` (ou par compte). Définissez `0` pour désactiver.

#### Limites d'historique des MP

```json
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Résolution : remplacement par MP → valeur par défaut du fournisseur → pas de limite (tout conservé). Pris en charge : `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Mode self-chat

Incluez votre propre numéro dans `allowFrom` pour activer le mode self-chat (ignore les @-mentions natives, ne répond qu'aux motifs de texte) :

```json
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Commandes (gestion des commandes de discussion)

```json
{
  commands: {
    native: "auto", // enregistrer les commandes natives lorsqu'elles sont supportées
    text: true, // analyser les /commandes dans les messages de discussion
    bash: false, // autoriser ! (alias : /bash)
    bashForegroundMs: 2000,
    config: false, // autoriser /config
    debug: false, // autoriser /debug
    restart: false, // autoriser /restart + outil de redémarrage de la passerelle
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   Les commandes texte doivent être des messages **autonomes** avec un `/` en tête.
-   `native: "auto"` active les commandes natives pour Discord/Telegram, laisse Slack désactivé.
-   Remplacement par canal : `channels.discord.commands.native` (booléen ou `"auto"`). `false` efface les commandes précédemment enregistrées.
-   `channels.telegram.customCommands` ajoute des entrées de menu de bot Telegram supplémentaires.
-   `bash: true` active `! ` pour le shell hôte. Nécessite `tools.elevated.enabled` et l'expéditeur dans `tools.elevated.allowFrom.`.
-   `config: true` active `/config` (lit/écrit `openclaw.json`). Pour les clients gateway `chat.send`, les écritures persistantes `/config set|unset` nécessitent également `operator.admin` ; la lecture seule `/config show` reste disponible pour les clients opérateur normaux.
-   `channels..configWrites` gère les mutations de configuration par canal (par défaut : true).
-   `allowFrom` est par fournisseur. Lorsqu'il est défini, c'est la **seule** source d'autorisation (listes d'autorisation de canal/appairage et `useAccessGroups` sont ignorés).
-   `useAccessGroups: false` permet aux commandes de contourner les politiques de groupe d'accès quando `allowFrom` n'est pas défini.

* * *

## Valeurs par défaut de l'agent

### agents.defaults.workspace

Par défaut : `~/.openclaw/workspace`.

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```