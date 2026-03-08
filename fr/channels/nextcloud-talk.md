

  Plateformes de messagerie

  
# Nextcloud Talk

Statut : pris en charge via plugin (bot webhook). Les messages directs, les salles, les réactions et les messages en markdown sont pris en charge.

## Plugin requis

Nextcloud Talk est fourni sous forme de plugin et n'est pas inclus dans l'installation principale. Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Installation locale (lors d'une exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Si vous choisissez Nextcloud Talk pendant la configuration/onboarding et qu'un dépôt git est détecté, OpenClaw proposera automatiquement le chemin d'installation local. Détails : [Plugins](../tools/plugin.md)

## Configuration rapide (débutant)

1.  Installez le plugin Nextcloud Talk.
2.  Sur votre serveur Nextcloud, créez un bot :
    
    Copier
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  Activez le bot dans les paramètres de la salle cible.
4.  Configurez OpenClaw :
    -   Config : `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   Ou variable d'env : `NEXTCLOUD_TALK_BOT_SECRET` (compte par défaut uniquement)
5.  Redémarrez la passerelle (ou terminez l'onboarding).

Configuration minimale :

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## Notes

-   Les bots ne peuvent pas initier de messages directs. L'utilisateur doit d'abord envoyer un message au bot.
-   L'URL du webhook doit être accessible par la Passerelle ; définissez `webhookPublicUrl` si vous êtes derrière un proxy.
-   Les téléversements de médias ne sont pas pris en charge par l'API bot ; les médias sont envoyés sous forme d'URL.
-   La charge utile du webhook ne distingue pas les messages directs des salles ; définissez `apiUser` + `apiPassword` pour activer les vérifications de type de salle (sinon, les messages directs sont traités comme des salles).

## Contrôle d'accès (messages directs)

-   Par défaut : `channels.nextcloud-talk.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage.
-   Approuvez via :
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   Messages directs publics : `channels.nextcloud-talk.dmPolicy="open"` plus `channels.nextcloud-talk.allowFrom=["*"]`.
-   `allowFrom` correspond uniquement aux identifiants utilisateur Nextcloud ; les noms d'affichage sont ignorés.

## Salles (groupes)

-   Par défaut : `channels.nextcloud-talk.groupPolicy = "allowlist"` (accès par mention).
-   Ajoutez des salles à la liste autorisée avec `channels.nextcloud-talk.rooms` :

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   Pour n'autoriser aucune salle, gardez la liste autorisée vide ou définissez `channels.nextcloud-talk.groupPolicy="disabled"`.

## Capacités

| Fonctionnalité | Statut |
| --- | --- |
| Messages directs | Pris en charge |
| Salles | Pris en charge |
| Fils de discussion | Non pris en charge |
| Médias | URL uniquement |
| Réactions | Pris en charge |
| Commandes natives | Non pris en charge |

## Référence de configuration (Nextcloud Talk)

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.nextcloud-talk.enabled` : activer/désactiver le démarrage du canal.
-   `channels.nextcloud-talk.baseUrl` : URL de l'instance Nextcloud.
-   `channels.nextcloud-talk.botSecret` : secret partagé du bot.
-   `channels.nextcloud-talk.botSecretFile` : chemin du fichier secret.
-   `channels.nextcloud-talk.apiUser` : utilisateur API pour les vérifications de salle (détection des messages directs).
-   `channels.nextcloud-talk.apiPassword` : mot de passe API/app pour les vérifications de salle.
-   `channels.nextcloud-talk.apiPasswordFile` : chemin du fichier de mot de passe API.
-   `channels.nextcloud-talk.webhookPort` : port d'écoute du webhook (par défaut : 8788).
-   `channels.nextcloud-talk.webhookHost` : hôte du webhook (par défaut : 0.0.0.0).
-   `channels.nextcloud-talk.webhookPath` : chemin du webhook (par défaut : /nextcloud-talk-webhook).
-   `channels.nextcloud-talk.webhookPublicUrl` : URL du webhook accessible publiquement.
-   `channels.nextcloud-talk.dmPolicy` : `pairing | allowlist | open | disabled`.
-   `channels.nextcloud-talk.allowFrom` : liste autorisée pour les messages directs (identifiants utilisateur). `open` nécessite `"*"`.
-   `channels.nextcloud-talk.groupPolicy` : `allowlist | open | disabled`.
-   `channels.nextcloud-talk.groupAllowFrom` : liste autorisée pour les groupes (identifiants utilisateur).
-   `channels.nextcloud-talk.rooms` : paramètres et liste autorisée par salle.
-   `channels.nextcloud-talk.historyLimit` : limite d'historique des groupes (0 désactive).
-   `channels.nextcloud-talk.dmHistoryLimit` : limite d'historique des messages directs (0 désactive).
-   `channels.nextcloud-talk.dms` : paramètres par message direct (historyLimit).
-   `channels.nextcloud-talk.textChunkLimit` : taille des morceaux de texte sortants (caractères).
-   `channels.nextcloud-talk.chunkMode` : `length` (par défaut) ou `newline` pour diviser sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   `channels.nextcloud-talk.blockStreaming` : désactiver le streaming par bloc pour ce canal.
-   `channels.nextcloud-talk.blockStreamingCoalesce` : réglage de la fusion du streaming par bloc.
-   `channels.nextcloud-talk.mediaMaxMb` : limite de taille des médias entrants (Mo).

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)