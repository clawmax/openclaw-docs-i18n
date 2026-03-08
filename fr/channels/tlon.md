

  Plateformes de messagerie

  
# Tlon

Tlon est un messager décentralisé construit sur Urbit. OpenClaw se connecte à votre vaisseau Urbit et peut répondre aux messages privés et aux messages des discussions de groupe. Les réponses en groupe nécessitent par défaut une mention @ et peuvent être davantage restreintes via des listes d'autorisation. Statut : pris en charge via plugin. Les messages privés, les mentions en groupe, les réponses dans les fils de discussion, la mise en forme de texte enrichi et le téléversement d'images sont pris en charge. Les réactions et les sondages ne sont pas encore pris en charge.

## Plugin requis

Tlon est fourni sous forme de plugin et n'est pas inclus dans l'installation de base. Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/tlon
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/tlon
```

Détails : [Plugins](../tools/plugin.md)

## Configuration

1.  Installez le plugin Tlon.
2.  Récupérez l'URL de votre vaisseau et votre code de connexion.
3.  Configurez `channels.tlon`.
4.  Redémarrez la passerelle.
5.  Envoyez un message privé au bot ou mentionnez-le dans un canal de groupe.

Configuration minimale (compte unique) :

```json
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // recommandé : votre vaisseau, toujours autorisé
    },
  },
}
```

## Vaisseaux privés/LAN

Par défaut, OpenClaw bloque les noms d'hôte et les plages d'adresses IP privés/internes pour la protection contre les attaques SSRF. Si votre vaisseau fonctionne sur un réseau privé (localhost, IP LAN ou nom d'hôte interne), vous devez l'autoriser explicitement :

```json
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

Cela s'applique aux URL comme :

-   `http://localhost:8080`
-   `http://192.168.x.x:8080`
-   `http://my-ship.local:8080`

⚠️ Activez ceci uniquement si vous faites confiance à votre réseau local. Ce paramètre désactive les protections SSRF pour les requêtes vers l'URL de votre vaisseau.

## Canaux de groupe

La découverte automatique est activée par défaut. Vous pouvez également épingler manuellement des canaux :

```json
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Désactiver la découverte automatique :

```json
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Contrôle d'accès

Liste d'autorisation pour les messages privés (vide = aucun MP autorisé, utilisez `ownerShip` pour un flux d'approbation) :

```json
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Autorisation de groupe (restreinte par défaut) :

```json
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Propriétaire et système d'approbation

Définissez un vaisseau propriétaire pour recevoir les demandes d'approbation lorsque des utilisateurs non autorisés tentent d'interagir :

```json
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

Le vaisseau propriétaire est **automatiquement autorisé partout** — les invitations en MP sont automatiquement acceptées et les messages dans les canaux sont toujours autorisés. Vous n'avez pas besoin d'ajouter le propriétaire à `dmAllowlist` ou `defaultAuthorizedShips`. Lorsqu'il est défini, le propriétaire reçoit des notifications en MP pour :

-   Les demandes de MP de vaisseaux absents de la liste d'autorisation
-   Les mentions dans les canaux sans autorisation
-   Les demandes d'invitation à un groupe

## Paramètres d'acceptation automatique

Accepter automatiquement les invitations en MP (pour les vaisseaux dans dmAllowlist) :

```json
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

Accepter automatiquement les invitations de groupe :

```json
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
    },
  },
}
```

## Cibles d'envoi (CLI/cron)

Utilisez celles-ci avec `openclaw message send` ou l'envoi par cron :

-   MP : `~sampel-palnet` ou `dm/~sampel-palnet`
-   Groupe : `chat/~host-ship/channel` ou `group:~host-ship/channel`

## Compétence incluse

Le plugin Tlon inclut une compétence intégrée ([`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill)) qui fournit un accès CLI aux opérations Tlon :

-   **Contacts** : obtenir/mettre à jour les profils, lister les contacts
-   **Canaux** : lister, créer, publier des messages, récupérer l'historique
-   **Groupes** : lister, créer, gérer les membres
-   **MP** : envoyer des messages, réagir aux messages
-   **Réactions** : ajouter/supprimer des réactions emoji aux publications et MP
-   **Paramètres** : gérer les permissions du plugin via des commandes slash

La compétence est automatiquement disponible lorsque le plugin est installé.

## Fonctionnalités

| Fonctionnalité | Statut |
| --- | --- |
| Messages privés | ✅ Pris en charge |
| Groupes/Canaux | ✅ Pris en charge (conditionné à une mention par défaut) |
| Fils de discussion | ✅ Pris en charge (réponses automatiques dans le fil) |
| Texte enrichi | ✅ Markdown converti au format Tlon |
| Images | ✅ Téléversées vers le stockage Tlon |
| Réactions | ✅ Via la [compétence incluse](#bundled-skill) |
| Sondages | ❌ Pas encore pris en charge |
| Commandes natives | ✅ Pris en charge (propriétaire uniquement par défaut) |

## Dépannage

Exécutez cette échelle en premier :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

Échecs courants :

-   **MP ignorés** : l'expéditeur n'est pas dans `dmAllowlist` et aucun `ownerShip` n'est configuré pour le flux d'approbation.
-   **Messages de groupe ignorés** : canal non découvert ou expéditeur non autorisé.
-   **Erreurs de connexion** : vérifiez que l'URL du vaisseau est accessible ; activez `allowPrivateNetwork` pour les vaisseaux locaux.
-   **Erreurs d'authentification** : vérifiez que le code de connexion est à jour (les codes tournent).

## Référence de configuration

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.tlon.enabled` : activer/désactiver le démarrage du canal.
-   `channels.tlon.ship` : nom du vaisseau Urbit du bot (ex. `~sampel-palnet`).
-   `channels.tlon.url` : URL du vaisseau (ex. `https://sampel-palnet.tlon.network`).
-   `channels.tlon.code` : code de connexion du vaisseau.
-   `channels.tlon.allowPrivateNetwork` : autoriser les URL localhost/LAN (contournement SSRF).
-   `channels.tlon.ownerShip` : vaisseau propriétaire pour le système d'approbation (toujours autorisé).
-   `channels.tlon.dmAllowlist` : vaisseaux autorisés à envoyer des MP (vide = aucun).
-   `channels.tlon.autoAcceptDmInvites` : accepter automatiquement les MP des vaisseaux autorisés.
-   `channels.tlon.autoAcceptGroupInvites` : accepter automatiquement toutes les invitations de groupe.
-   `channels.tlon.autoDiscoverChannels` : découvrir automatiquement les canaux de groupe (par défaut : true).
-   `channels.tlon.groupChannels` : nids de canaux épinglés manuellement.
-   `channels.tlon.defaultAuthorizedShips` : vaisseaux autorisés pour tous les canaux.
-   `channels.tlon.authorization.channelRules` : règles d'autorisation par canal.
-   `channels.tlon.showModelSignature` : ajouter le nom du modèle aux messages.

## Notes

-   Les réponses en groupe nécessitent une mention (ex. `~your-bot-ship`) pour répondre.
-   Réponses dans les fils : si le message entrant est dans un fil, OpenClaw répond dans le fil.
-   Texte enrichi : la mise en forme Markdown (gras, italique, code, en-têtes, listes) est convertie au format natif de Tlon.
-   Images : les URL sont téléversées vers le stockage Tlon et intégrées sous forme de blocs d'image.

[Télégramme](./telegram.md)[Twitch](./twitch.md)

---