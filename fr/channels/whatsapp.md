

  Plateformes de messagerie

  
# WhatsApp

Statut : prêt pour la production via WhatsApp Web (Baileys). La passerelle possède la/les session(s) liée(s).

## Configuration rapide

### Étape 1 : Configurer la politique d'accès WhatsApp

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### Étape 2 : Lier WhatsApp (QR)

```bash
openclaw channels login --channel whatsapp
```

Pour un compte spécifique :

```bash
openclaw channels login --channel whatsapp --account work
```

### Étape 3 : Démarrer la passerelle

```bash
openclaw gateway
```

### Étape 4 : Approuver la première demande d'appairage (si mode appairage)

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

Les demandes d'appairage expirent après 1 heure. Les demandes en attente sont limitées à 3 par canal.

 

> **ℹ️** OpenClaw recommande d'exécuter WhatsApp sur un numéro séparé lorsque c'est possible. (Les métadonnées du canal et le flux d'intégration sont optimisés pour cette configuration, mais les configurations avec numéro personnel sont également prises en charge.)

## Modèles de déploiement

C'est le mode opérationnel le plus propre :

-   identité WhatsApp distincte pour OpenClaw
-   listes d'autorisation DM et limites de routage plus claires
-   risque réduit de confusion d'auto-conversation

Modèle de politique minimal :

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

L'intégration prend en charge le mode numéro personnel et écrit une base adaptée à l'auto-conversation :

-   `dmPolicy: "allowlist"`
-   `allowFrom` inclut votre numéro personnel
-   `selfChatMode: true`

À l'exécution, les protections d'auto-conversation s'appuient sur le numéro personnel lié et `allowFrom`.

Le canal de plateforme de messagerie est basé sur WhatsApp Web (`Baileys`) dans l'architecture actuelle des canaux OpenClaw. Il n'existe pas de canal de messagerie WhatsApp Twilio séparé dans le registre des canaux de chat intégrés.

## Modèle d'exécution

-   La passerelle possède le socket WhatsApp et la boucle de reconnexion.
-   Les envois sortants nécessitent un écouteur WhatsApp actif pour le compte cible.
-   Les chats de statut et de diffusion sont ignorés (`@status`, `@broadcast`).
-   Les conversations directes utilisent les règles de session DM (`session.dmScope` ; par défaut `main` regroupe les DM vers la session principale de l'agent).
-   Les sessions de groupe sont isolées (`agent::whatsapp:group:`).

## Contrôle d'accès et activation

`channels.whatsapp.dmPolicy` contrôle l'accès aux conversations directes :

-   `pairing` (par défaut)
-   `allowlist`
-   `open` (nécessite que `allowFrom` inclue `"*"`)
-   `disabled`

`allowFrom` accepte les numéros au format E.164 (normalisés en interne). Surcharge multi-compte : `channels.whatsapp.accounts..dmPolicy` (et `allowFrom`) prennent la priorité sur les valeurs par défaut au niveau du canal pour ce compte. Détails du comportement d'exécution :

-   les appairages sont persistés dans le magasin d'autorisation du canal et fusionnés avec `allowFrom` configuré
-   si aucune liste d'autorisation n'est configurée, le numéro personnel lié est autorisé par défaut
-   les DM sortants `fromMe` ne sont jamais auto-appairés

L'accès aux groupes a deux couches :

1.  **Liste d'autorisation d'appartenance au groupe** (`channels.whatsapp.groups`)
    -   si `groups` est omis, tous les groupes sont éligibles
    -   si `groups` est présent, il agit comme une liste d'autorisation de groupe (`"*"` autorisé)
2.  **Politique d'expéditeur de groupe** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
    -   `open` : la liste d'autorisation d'expéditeur est contournée
    -   `allowlist` : l'expéditeur doit correspondre à `groupAllowFrom` (ou `*`)
    -   `disabled` : bloquer toutes les entrées de groupe

Liste d'autorisation d'expéditeur de secours :

-   si `groupAllowFrom` n'est pas défini, l'exécution utilise `allowFrom` lorsqu'il est disponible
-   les listes d'autorisation d'expéditeur sont évaluées avant l'activation de mention/réponse

Note : si aucun bloc `channels.whatsapp` n'existe du tout, la politique de groupe de secours à l'exécution est `allowlist` (avec un avertissement dans les logs), même si `channels.defaults.groupPolicy` est défini.

Les réponses de groupe nécessitent une mention par défaut. La détection de mention inclut :

-   les mentions WhatsApp explicites de l'identité du bot
-   les modèles regex de mention configurés (`agents.list[].groupChat.mentionPatterns`, secours `messages.groupChat.mentionPatterns`)
-   la détection implicite de réponse-au-bot (l'expéditeur de la réponse correspond à l'identité du bot)

Note de sécurité :

-   la citation/réponse ne satisfait que la porte de mention ; elle **n'accorde pas** l'autorisation d'expéditeur
-   avec `groupPolicy: "allowlist"`, les expéditeurs non autorisés sont toujours bloqués même s'ils répondent à un message d'un utilisateur autorisé

Commande d'activation au niveau de la session :

-   `/activation mention`
-   `/activation always`

`activation` met à jour l'état de la session (pas la configuration globale). Elle est réservée au propriétaire.

## Comportement avec numéro personnel et auto-conversation

Lorsque le numéro personnel lié est également présent dans `allowFrom`, les protections d'auto-conversation WhatsApp s'activent :

-   ignorer les accusés de lecture pour les tours d'auto-conversation
-   ignorer le comportement de déclenchement automatique par mention-JID qui vous pinguerait autrement
-   si `messages.responsePrefix` n'est pas défini, les réponses d'auto-conversation utilisent par défaut `[{identity.name}]` ou `[openclaw]`

## Normalisation des messages et contexte

Les messages WhatsApp entrants sont encapsulés dans l'enveloppe entrante partagée. Si une réponse citée existe, le contexte est ajouté sous cette forme :

```json
[En réponse à <expéditeur> id:<stanzaId>]
<corps cité ou espace réservé pour média>
[/En réponse]
```

Les champs de métadonnées de réponse sont également renseignés lorsqu'ils sont disponibles (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, JID/E.164 de l'expéditeur).

Les messages entrants uniquement média sont normalisés avec des espaces réservés tels que :

-   `<media:image>`
-   `<media:video>`
-   `<media:audio>`
-   `<media:document>`
-   `<media:sticker>`

Les charges utiles de localisation et de contact sont normalisées en contexte textuel avant routage.

Pour les groupes, les messages non traités peuvent être mis en mémoire tampon et injectés comme contexte lorsque le bot est finalement déclenché.

-   limite par défaut : `50`
-   configuration : `channels.whatsapp.historyLimit`
-   secours : `messages.groupChat.historyLimit`
-   `0` désactive

Marqueurs d'injection :

-   `[Messages du chat depuis votre dernière réponse - pour contexte]`
-   `[Message actuel - répondez à celui-ci]`

Les accusés de lecture sont activés par défaut pour les messages WhatsApp entrants acceptés. Désactiver globalement :

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

Surcharge par compte :

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

Les tours d'auto-conversation ignorent les accusés de lecture même lorsqu'ils sont activés globalement.

## Livraison, fragmentation et médias

-   limite de fragment par défaut : `channels.whatsapp.textChunkLimit = 4000`
-   `channels.whatsapp.chunkMode = "length" | "newline"`
-   le mode `newline` préfère les limites de paragraphe (lignes vides), puis utilise la fragmentation sécurisée par longueur

-   prend en charge les charges utiles image, vidéo, audio (note vocale PTT) et document
-   `audio/ogg` est réécrit en `audio/ogg; codecs=opus` pour la compatibilité note vocale
-   la lecture de GIF animés est prise en charge via `gifPlayback: true` sur les envois vidéo
-   les légendes sont appliquées au premier élément média lors de l'envoi de charges utiles de réponse multi-médias
-   la source média peut être HTTP(S), `file://`, ou des chemins locaux

-   limite de sauvegarde des médias entrants : `channels.whatsapp.mediaMaxMb` (par défaut `50`)
-   limite d'envoi des médias sortants : `channels.whatsapp.mediaMaxMb` (par défaut `50`)
-   les surcharges par compte utilisent `channels.whatsapp.accounts..mediaMaxMb`
-   les images sont auto-optimisées (redimensionnement/qualité) pour respecter les limites
-   en cas d'échec d'envoi de média, le secours du premier élément envoie un avertissement texte au lieu de supprimer silencieusement la réponse

## Réactions d'accusé de réception

WhatsApp prend en charge les réactions d'accusé de réception immédiates sur réception entrante via `channels.whatsapp.ackReaction`.

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Notes de comportement :

-   envoyé immédiatement après l'acceptation de l'entrée (avant la réponse)
-   les échecs sont journalisés mais ne bloquent pas la livraison normale de la réponse
-   le mode groupe `mentions` réagit sur les tours déclenchés par mention ; l'activation de groupe `always` agit comme contournement pour cette vérification
-   WhatsApp utilise `channels.whatsapp.ackReaction` (l'ancien `messages.ackReaction` n'est pas utilisé ici)

## Multi-comptes et identifiants

-   les identifiants de compte proviennent de `channels.whatsapp.accounts`
-   sélection de compte par défaut : `default` s'il est présent, sinon le premier identifiant de compte configuré (trié)
-   les identifiants de compte sont normalisés en interne pour la recherche

-   chemin d'authentification actuel : `~/.openclaw/credentials/whatsapp//creds.json`
-   fichier de sauvegarde : `creds.json.bak`
-   l'authentification par défaut héritée dans `~/.openclaw/credentials/` est toujours reconnue/migrée pour les flux de compte par défaut

`openclaw channels logout --channel whatsapp [--account ]` efface l'état d'authentification WhatsApp pour ce compte. Dans les répertoires d'authentification hérités, `oauth.json` est préservé tandis que les fichiers d'authentification Baileys sont supprimés.

## Outils, actions et écritures de configuration

-   La prise en charge des outils d'agent inclut l'action de réaction WhatsApp (`react`).
-   Portes d'action :
    -   `channels.whatsapp.actions.reactions`
    -   `channels.whatsapp.actions.polls`
-   Les écritures de configuration initiées par le canal sont activées par défaut (désactiver via `channels.whatsapp.configWrites=false`).

## Dépannage

Symptôme : le statut du canal indique non lié. Correction :

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

Symptôme : compte lié avec déconnexions répétées ou tentatives de reconnexion. Correction :

```bash
openclaw doctor
openclaw logs --follow
```

Si nécessaire, re-lier avec `channels login`.

Les envois sortants échouent rapidement lorsqu'aucun écouteur de passerelle actif n'existe pour le compte cible. Assurez-vous que la passerelle est en cours d'exécution et que le compte est lié.

Vérifier dans cet ordre :

-   `groupPolicy`
-   `groupAllowFrom` / `allowFrom`
-   les entrées de la liste d'autorisation `groups`
-   la porte de mention (`requireMention` + modèles de mention)
-   les clés en double dans `openclaw.json` (JSON5) : les entrées ultérieures écrasent les précédentes, donc conservez une seule `groupPolicy` par portée

L'exécution de la passerelle WhatsApp doit utiliser Node. Bun est signalé comme incompatible pour une opération stable de la passerelle WhatsApp/Telegram.

## Pointeurs de référence de configuration

Référence principale :

-   [Référence de configuration - WhatsApp](../gateway/configuration-reference.md#whatsapp)

Champs WhatsApp à fort signal :

-   accès : `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
-   livraison : `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
-   multi-compte : `accounts..enabled`, `accounts..authDir`, surcharges au niveau du compte
-   opérations : `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
-   comportement de session : `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms..historyLimit`

## Liens connexes

-   [Appairage](./pairing.md)
-   [Routage des canaux](./channel-routing.md)
-   [Routage multi-agent](../concepts/multi-agent.md)
-   [Dépannage](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)