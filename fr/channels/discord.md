

  Plateformes de messagerie

  
# Discord

Statut : prêt pour les messages privés et les canaux de guilde via la passerelle officielle Discord.

## Configuration rapide

Vous devrez créer une nouvelle application avec un bot, ajouter le bot à votre serveur et l'appairer à OpenClaw. Nous recommandons d'ajouter votre bot à votre propre serveur privé. Si vous n'en avez pas encore, [créez-en un d'abord](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (choisissez **Create My Own > For me and my friends**).

### Étape 1 : Créer une application Discord et un bot

Allez sur le [Portail Développeur Discord](https://discord.com/developers/applications) et cliquez sur **New Application**. Nommez-la par exemple "OpenClaw". Cliquez sur **Bot** dans la barre latérale. Définissez le **Username** comme vous appelez votre agent OpenClaw.

### Étape 2 : Activer les intents privilégiés

Toujours sur la page **Bot**, descendez jusqu'à **Privileged Gateway Intents** et activez :

-   **Message Content Intent** (requis)
-   **Server Members Intent** (recommandé ; requis pour les listes d'autorisation par rôle et la correspondance nom-vers-ID)
-   **Presence Intent** (optionnel ; seulement nécessaire pour les mises à jour de présence)

### Étape 3 : Copier votre token de bot

Remontez sur la page **Bot** et cliquez sur **Reset Token**.

> **ℹ️** Malgré le nom, cela génère votre premier token — rien n'est "réinitialisé".

Copiez le token et sauvegardez-le. C'est votre **Bot Token** et vous en aurez besoin bientôt.

### Étape 4 : Générer une URL d'invitation et ajouter le bot à votre serveur

Cliquez sur **OAuth2** dans la barre latérale. Vous allez générer une URL d'invitation avec les bonnes permissions pour ajouter le bot à votre serveur. Descendez jusqu'à **OAuth2 URL Generator** et activez :

-   `bot`
-   `applications.commands`

Une section **Bot Permissions** apparaîtra en dessous. Activez :

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (optionnel)

Copiez l'URL générée en bas, collez-la dans votre navigateur, sélectionnez votre serveur et cliquez sur **Continue** pour connecter. Vous devriez maintenant voir votre bot dans le serveur Discord.

### Étape 5 : Activer le Mode Développeur et collecter vos IDs

De retour dans l'application Discord, vous devez activer le Mode Développeur pour pouvoir copier les IDs internes.

1.  Cliquez sur **User Settings** (icône d'engrenage à côté de votre avatar) → **Advanced** → activez **Developer Mode**
2.  Faites un clic droit sur l'**icône de votre serveur** dans la barre latérale → **Copy Server ID**
3.  Faites un clic droit sur **votre propre avatar** → **Copy User ID**

Sauvegardez votre **Server ID** et votre **User ID** avec votre Bot Token — vous enverrez les trois à OpenClaw à l'étape suivante.

### Étape 6 : Autoriser les messages privés des membres du serveur

Pour que l'appairage fonctionne, Discord doit autoriser votre bot à vous envoyer des messages privés. Faites un clic droit sur l'**icône de votre serveur** → **Privacy Settings** → activez **Direct Messages**. Cela permet aux membres du serveur (y compris les bots) de vous envoyer des messages privés. Gardez ceci activé si vous voulez utiliser les messages privés Discord avec OpenClaw. Si vous prévoyez uniquement d'utiliser les canaux de guilde, vous pouvez désactiver les messages privés après l'appairage.

### Étape 7 : Étape 0 : Définir votre token de bot de manière sécurisée (ne l'envoyez pas dans le chat)

Votre token de bot Discord est un secret (comme un mot de passe). Définissez-le sur la machine exécutant OpenClaw avant de contacter votre agent.

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

Si OpenClaw est déjà en cours d'exécution en tant que service en arrière-plan, utilisez `openclaw gateway restart` à la place.

### Étape 8 : Configurer OpenClaw et appairer

Discutez avec votre agent OpenClaw sur n'importe quel canal existant (par ex. Telegram) et dites-lui. Si Discord est votre premier canal, utilisez plutôt l'interface CLI / config.

> “I already set my Discord bot token in config. Please finish Discord setup with User ID `<user_id>` and Server ID `<server_id>`.”

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

### Étape 9 : Approuver le premier appairage par message privé

Attendez que la passerelle soit en cours d'exécution, puis envoyez un message privé à votre bot sur Discord. Il répondra avec un code d'appairage.

Envoyez le code d'appairage à votre agent sur votre canal existant :

> “Approve this Discord pairing code: ``”

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

Les codes d'appairage expirent après 1 heure. Vous devriez maintenant pouvoir discuter avec votre agent sur Discord via message privé.

 

> **ℹ️** La résolution des tokens tient compte du compte. Les valeurs de token dans la config l'emportent sur la valeur de repli d'environnement. `DISCORD_BOT_TOKEN` n'est utilisé que pour le compte par défaut.

## Recommandé : Configurer un espace de travail de guilde

Une fois que les messages privés fonctionnent, vous pouvez configurer votre serveur Discord comme un espace de travail complet où chaque canal obtient sa propre session d'agent avec son propre contexte. Ceci est recommandé pour les serveurs privés où il n'y a que vous et votre bot.

### Étape 1 : Ajouter votre serveur à la liste d'autorisation de guilde

Cela permet à votre agent de répondre dans n'importe quel canal de votre serveur, pas seulement dans les messages privés.

> “Add my Discord Server ID `<server_id>` to the guild allowlist”

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

### Étape 2 : Autoriser les réponses sans @mention

Par défaut, votre agent ne répond dans les canaux de guilde que lorsqu'il est @mentionné. Pour un serveur privé, vous voulez probablement qu'il réponde à chaque message.

> “Allow my agent to respond on this server without having to be @mentioned”

```json
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

### Étape 3 : Planifier la mémoire dans les canaux de guilde

Par défaut, la mémoire à long terme (MEMORY.md) ne se charge que dans les sessions de messages privés. Les canaux de guilde ne chargent pas automatiquement MEMORY.md.

> “When I ask questions in Discord channels, use memory\_search or memory\_get if you need long-term context from MEMORY.md.”

Si vous avez besoin d'un contexte partagé dans chaque canal, placez les instructions stables dans `AGENTS.md` ou `USER.md` (elles sont injectées pour chaque session). Gardez les notes à long terme dans `MEMORY.md` et accédez-y à la demande avec les outils de mémoire.

 Maintenant, créez quelques canaux sur votre serveur Discord et commencez à discuter. Votre agent peut voir le nom du canal, et chaque canal obtient sa propre session isolée — vous pouvez donc configurer `#coding`, `#home`, `#research`, ou tout ce qui correspond à votre flux de travail.

## Modèle d'exécution

-   La passerelle possède la connexion Discord.
-   Le routage des réponses est déterministe : les réponses entrantes de Discord reviennent vers Discord.
-   Par défaut (`session.dmScope=main`), les discussions directes partagent la session principale de l'agent (`agent:main:main`).
-   Les canaux de guilde ont des clés de session isolées (`agent::discord:channel:`).
-   Les messages privés de groupe sont ignorés par défaut (`channels.discord.dm.groupEnabled=false`).
-   Les commandes slash natives s'exécutent dans des sessions de commande isolées (`agent::discord:slash:`), tout en portant `CommandTargetSessionKey` vers la session de conversation routée.

## Canaux de forum

Les canaux de forum et média Discord n'acceptent que les publications dans les fils de discussion. OpenClaw prend en charge deux façons de les créer :

-   Envoyez un message au parent du forum (`channel:`) pour créer automatiquement un fil. Le titre du fil utilise la première ligne non vide de votre message.
-   Utilisez `openclaw message thread create` pour créer un fil directement. Ne passez pas `--message-id` pour les canaux de forum.

Exemple : envoyer au parent du forum pour créer un fil

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

Exemple : créer un fil de forum explicitement

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

Les parents de forum n'acceptent pas les composants Discord. Si vous avez besoin de composants, envoyez au fil lui-même (`channel:`).

## Composants interactifs

OpenClaw prend en charge les conteneurs de composants Discord v2 pour les messages de l'agent. Utilisez l'outil de message avec une charge utile `components`. Les résultats des interactions sont routés vers l'agent en tant que messages entrants normaux et suivent les paramètres Discord `replyToMode` existants. Blocs pris en charge :

-   `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
-   Les rangées d'actions permettent jusqu'à 5 boutons ou un seul menu de sélection
-   Types de sélection : `string`, `user`, `role`, `mentionable`, `channel`

Par défaut, les composants sont à usage unique. Définissez `components.reusable=true` pour permettre aux boutons, menus de sélection et formulaires d'être utilisés plusieurs fois jusqu'à expiration. Pour restreindre qui peut cliquer sur un bouton, définissez `allowedUsers` sur ce bouton (IDs utilisateur Discord, tags, ou `*`). Lorsqu'il est configuré, les utilisateurs non correspondants reçoivent un refus éphémère. Les commandes slash `/model` et `/models` ouvrent un sélecteur de modèle interactif avec des menus déroulants de fournisseur et de modèle ainsi qu'une étape Submit. La réponse du sélecteur est éphémère et seul l'utilisateur invoquant peut l'utiliser. Pièces jointes de fichiers :

-   Les blocs `file` doivent pointer vers une référence de pièce jointe (`attachment://`)
-   Fournissez la pièce jointe via `media`/`path`/`filePath` (fichier unique) ; utilisez `media-gallery` pour plusieurs fichiers
-   Utilisez `filename` pour remplacer le nom de téléchargement lorsqu'il doit correspondre à la référence de pièce jointe

Formulaires modaux :

-   Ajoutez `components.modal` avec jusqu'à 5 champs
-   Types de champs : `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
-   OpenClaw ajoute automatiquement un bouton de déclenchement

Exemple :

```json
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## Contrôle d'accès et routage

`channels.discord.dmPolicy` contrôle l'accès aux messages privés (hérité : `channels.discord.dm.policy`) :

-   `pairing` (par défaut)
-   `allowlist`
-   `open` (nécessite que `channels.discord.allowFrom` inclue `"*"` ; hérité : `channels.discord.dm.allowFrom`)
-   `disabled`

Si la politique MP n'est pas ouverte, les utilisateurs inconnus sont bloqués (ou invités à s'appairer en mode `pairing`). Prépondérance multi-compte :

-   `channels.discord.accounts.default.allowFrom` s'applique uniquement au compte `default`.
-   Les comptes nommés héritent de `channels.discord.allowFrom` lorsque leur propre `allowFrom` n'est pas défini.
-   Les comptes nommés n'héritent pas de `channels.discord.accounts.default.allowFrom`.

Format de cible MP pour la livraison :

-   `user:`
-   Mention `<@id>`

Les IDs numériques bruts sont ambigus et rejetés à moins qu'un type de cible utilisateur/canal explicite ne soit fourni.

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          ignoreOtherMentions: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

Les messages de guilde sont verrouillés par mention par défaut. La détection de mention inclut :

-   mention explicite du bot
-   modèles de mention configurés (`agents.list[].groupChat.mentionPatterns`, repli `messages.groupChat.mentionPatterns`)
-   comportement implicite de réponse-au-bot dans les cas pris en charge

`requireMention` est configuré par guilde/canal (`channels.discord.guilds...`). `ignoreOtherMentions` supprime optionnellement les messages qui mentionnent un autre utilisateur/rôle mais pas le bot (à l'exclusion de @everyone/@here). MP de groupe :

-   par défaut : ignorés (`dm.groupEnabled=false`)
-   liste d'autorisation optionnelle via `dm.groupChannels` (IDs de canal ou slugs)

### Routage d'agent basé sur les rôles

Utilisez `bindings[].match.roles` pour router les membres de guilde Discord vers différents agents par ID de rôle. Les liaisons basées sur les rôles n'acceptent que les IDs de rôle et sont évaluées après les liaisons de pair ou parent-pair et avant les liaisons de guilde uniquement. Si une liaison définit également d'autres champs de correspondance (par exemple `peer` + `guildId` + `roles`), tous les champs configurés doivent correspondre.

```json
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuration du Portail Développeur

1.  Portail Développeur Discord -> **Applications** -> **New Application**
2.  **Bot** -> **Add Bot**
3.  Copier le token du bot

Dans **Bot -> Privileged Gateway Intents**, activez :

-   Message Content Intent
-   Server Members Intent (recommandé)

L'intent de présence est optionnel et seulement requis si vous voulez recevoir des mises à jour de présence. Définir la présence du bot (`setPresence`) ne nécessite pas d'activer les mises à jour de présence pour les membres.

Générateur d'URL OAuth :

-   scopes : `bot`, `applications.commands`

Permissions de base typiques :

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (optionnel)

Évitez `Administrator` sauf si explicitement nécessaire.

Activez le Mode Développeur Discord, puis copiez :

-   ID du serveur
-   ID du canal
-   ID de l'utilisateur

Préférez les IDs numériques dans la config OpenClaw pour des audits et sondages fiables.

## Commandes natives et authentification des commandes

-   `commands.native` est par défaut `"auto"` et est activé pour Discord.
-   Remplacement par canal : `channels.discord.commands.native`.
-   `commands.native=false` efface explicitement les commandes natives Discord précédemment enregistrées.
-   L'authentification des commandes natives utilise les mêmes listes d'autorisation/politiques Discord que le traitement normal des messages.
-   Les commandes peuvent toujours être visibles dans l'interface Discord pour les utilisateurs non autorisés ; l'exécution applique toujours l'authentification OpenClaw et renvoie "not authorized".

Voir [Commandes slash](../tools/slash-commands.md) pour le catalogue de commandes et le comportement. Paramètres par défaut des commandes slash :

-   `ephemeral: true`

## Détails des fonctionnalités

Discord prend en charge les tags de réponse dans la sortie de l'agent :

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

Contrôlé par `channels.discord.replyToMode` :

-   `off` (par défaut)
-   `first`
-   `all`

Note : `off` désactive le threading de réponse implicite. Les tags explicites `[[reply_to_*]]` sont toujours honorés. Les IDs de message sont exposés dans le contexte/historique pour que les agents puissent cibler des messages spécifiques.

OpenClaw peut diffuser des réponses brouillons en envoyant un message temporaire et en l'éditant à l'arrivée du texte.

-   `channels.discord.streaming` contrôle la diffusion d'aperçu (`off` | `partial` | `block` | `progress`, par défaut : `off`).
-   `progress` est accepté pour la cohérence inter-canaux et correspond à `partial` sur Discord.
-   `channels.discord.streamMode` est un alias hérité et est migré automatiquement.
-   `partial` édite un seul message d'aperçu à l'arrivée des tokens.
-   `block` émet des morceaux de taille brouillon (utilisez `draftChunk` pour ajuster la taille et les points de rupture).

Exemple :

```json
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

Découpage par défaut du mode `block` (limité à `channels.discord.textChunkLimit`) :

```json
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

La diffusion d'aperçu est texte uniquement ; les réponses avec média reviennent à la livraison normale. Note : la diffusion d'aperçu est séparée de la diffusion par blocs. Lorsque la diffusion par blocs est explicitement activée pour Discord, OpenClaw ignore le flux d'aperçu pour éviter une double diffusion.

Contexte historique de la guilde :

-   `channels.discord.historyLimit` par défaut `20`
-   repli : `messages.groupChat.historyLimit`
-   `0` désactive

Contrôles d'historique des MP :

-   `channels.discord.dmHistoryLimit`
-   `channels.discord.dms["<user_id>"].historyLimit`

Comportement des fils :

-   Les fils Discord sont routés comme des sessions de canal
-   Les métadonnées du fil parent peuvent être utilisées pour la liaison parent-session
-   La config du fil hérite de la config du canal parent sauf si une entrée spécifique au fil existe

Les sujets des canaux sont injectés en tant que contexte **non fiable** (pas comme prompt système).

Discord peut lier un fil à une cible de session afin que les messages de suivi dans ce fil continuent à router vers la même session (y compris les sessions de sous-agent). Commandes :

-   `/focus ` lier le fil actuel/nouveau à une cible de sous-agent/session
-   `/unfocus` supprimer la liaison du fil actuel
-   `/agents` afficher les exécutions actives et l'état des liaisons
-   `/session idle <duration|off>` inspecter/mettre à jour l'inactivité auto-déliaison pour les liaisons focus
-   `/session max-age <duration|off>` inspecter/mettre à jour la durée de vie maximale pour les liaisons focus

Config :

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in
      },
    },
  },
}
```

Notes :

-   `session.threadBindings.*` définit les valeurs globales par défaut.
-   `channels.discord.threadBindings.*` remplace le comportement Discord.
-   `spawnSubagentSessions` doit être vrai pour créer/lier automatiquement des fils pour `sessions_spawn({ thread: true })`.
-   `spawnAcpSessions` doit être vrai pour créer/lier automatiquement des fils pour ACP (`/acp spawn ... --thread ...` ou `sessions_spawn({ runtime: "acp", thread: true })`).
-   Si les liaisons de fils sont désactivées pour un compte, `/focus` et les opérations de liaison de fils associées sont indisponibles.

Voir [Sous-agents](../tools/subagents.md), [Agents ACP](../tools/acp-agents.md), et [Référence de configuration](../gateway/configuration-reference.md).

Pour des espaces de travail ACP "toujours actifs" stables, configurez des liaisons ACP typées de haut niveau ciblant des conversations Discord. Chemin de config :

-   `bindings[]` avec `type: "acp"` et `match.channel: "discord"`

Exemple :

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

Notes :

-   Les messages de fil peuvent hériter de la liaison ACP du canal parent.
-   Dans un canal ou fil lié, `/new` et `/reset` réinitialisent la même session ACP sur place.
-   Les liaisons de fil temporaires fonctionnent toujours et peuvent remplacer la résolution de cible tant qu'elles sont actives.

Voir [Agents ACP](../tools/acp-agents.md) pour les détails du comportement des liaisons.

Mode de notification de réaction par guilde :

-   `off`
-   `own` (par défaut)
-   `all`
-   `allowlist` (utilise `guilds..users`)

Les événements de réaction sont transformés en événements système et attachés à la session Discord routée.

`ackReaction` envoie un emoji d'accusé de réception pendant qu'OpenClaw traite un message entrant. Ordre de résolution :

-   `channels.discord.accounts..ackReaction`
-   `channels.discord.ackReaction`
-   `messages.ackReaction`
-   repli emoji d'identité de l'agent (`agents.list[].identity.emoji`, sinon ”👀”)

Notes :

-   Discord accepte les emojis unicode ou les noms d'emoji personnalisés.
-   Utilisez `""` pour désactiver la réaction pour un canal ou compte.

Les écritures de config initiées par le canal sont activées par défaut. Cela affecte les flux `/config set|unset` (lorsque les fonctionnalités de commande sont activées). Désactiver :

```json
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

Routez le trafic WebSocket de la passerelle Discord et les recherches REST de démarrage (ID d'application + résolution de liste d'autorisation) via un proxy HTTP(S) avec `channels.discord.proxy`.

```json
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

Remplacement par compte :

```json
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

Activez la résolution PluralKit pour mapper les messages proxy à l'identité du membre du système :

```json
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optionnel ; nécessaire pour les systèmes privés
      },
    },
  },
}
```

Notes :

-   les listes d'autorisation peuvent utiliser `pk:`
-   les noms d'affichage des membres sont appariés par nom/slug uniquement lorsque `channels.discord.dangerouslyAllowNameMatching: true`
-   les recherches utilisent l'ID de message original et sont limitées par une fenêtre temporelle
-   si la recherche échoue, les messages proxy sont traités comme des messages de bot et ignorés sauf si `allowBots=true`

Les mises à jour de présence sont appliquées lorsque vous définissez un statut ou un champ d'activité, ou lorsque vous activez la présence automatique. Exemple statut uniquement :

```json
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

Exemple d'activité (le statut personnalisé est le type d'activité par défaut) :

```json
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

Exemple de streaming :

```json
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

Correspondance des types d'activité :

-   0: Playing
-   1: Streaming (nécessite `activityUrl`)
-   2: Listening
-   3: Watching
-   4: Custom (utilise le texte d'activité comme état du statut ; emoji optionnel)
-   5: Competing

Exemple de présence automatique (signal de santé d'exécution) :

```json
{
  channels: {
    discord: {
      autoPresence: {
        enabled: true,
        intervalMs: 30000,
        minUpdateIntervalMs: 15000,
        exhaustedText: "token exhausted",
      },
    },
  },
}
```

La présence automatique mappe la disponibilité d'exécution au statut Discord : sain => en ligne, dégradé ou inconnu => inactif, épuisé ou indisponible => ne pas déranger. Remplacements de texte optionnels :

-   `autoPresence.healthyText`
-   `autoPresence.degradedText`
-   `autoPresence.exhaustedText` (prend en charge le placeholder `{reason}`)

Discord prend en charge les approbations d'exécution basées sur des boutons dans les MP et peut optionnellement publier les invites d'approbation dans le canal d'origine. Chemin de config :

-   `channels.discord.execApprovals.enabled`
-   `channels.discord.execApprovals.approvers`
-   `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, par défaut : `dm`)
-   `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

Lorsque `target` est `channel` ou `both`, l'invite d'approbation est visible dans le canal. Seuls les approbateurs configurés peuvent utiliser les boutons ; les autres utilisateurs reçoivent un refus éphémère. Les invites d'approbation incluent le texte de la commande, donc n'activez la livraison par canal que dans les canaux de confiance. Si l'ID du canal ne peut pas être dérivé de la clé de session, OpenClaw revient à la livraison par MP.

Si les approbations échouent avec des IDs d'approbation inconnus, vérifiez la liste des approbateurs et l'activation de la fonctionnalité.

Documentation connexe : [Approbations d'exécution](../tools/exec-approvals.md)

## Outils et portes d'action

## Sécurité et opérations

-   Traitez les jetons de bot comme des secrets (`DISCORD_BOT_TOKEN` préféré dans les environnements supervisés).
-   Accordez les permissions Discord de moindre privilège.
-   Si le déploiement de commande / l'état est périmé, redémarrez la passerelle et revérifiez avec `openclaw channels status --probe`.

## Connexe

-   [Appariement](./pairing.md)
-   [Routage des canaux](./channel-routing.md)
-   [Routage multi-agent](../concepts/multi-agent.md)
-   [Dépannage](./troubleshooting.md)
-   [Commandes slash](../tools/slash-commands.md)
