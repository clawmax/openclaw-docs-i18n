

  Plateformes de messagerie

  
# LINE

LINE se connecte à OpenClaw via la LINE Messaging API. Le plugin fonctionne comme un récepteur de webhook sur la passerelle et utilise votre jeton d'accès au canal + le secret du canal pour l'authentification. Statut : pris en charge via plugin. Les messages directs, les discussions de groupe, les médias, les localisations, les messages Flex, les messages modèles et les réponses rapides sont pris en charge. Les réactions et les fils de discussion ne sont pas pris en charge.

## Plugin requis

Installez le plugin LINE :

```bash
openclaw plugins install @openclaw/line
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/line
```

## Configuration

1.  Créez un compte LINE Developers et ouvrez la Console : [https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  Créez (ou choisissez) un Fournisseur et ajoutez un canal **Messaging API**.
3.  Copiez le **Channel access token** et le **Channel secret** depuis les paramètres du canal.
4.  Activez **Use webhook** dans les paramètres de la Messaging API.
5.  Définissez l'URL du webhook sur le point de terminaison de votre passerelle (HTTPS requis) :

```
https://gateway-host/line/webhook
```

La passerelle répond à la vérification du webhook de LINE (GET) et aux événements entrants (POST). Si vous avez besoin d'un chemin personnalisé, définissez `channels.line.webhookPath` ou `channels.line.accounts..webhookPath` et mettez à jour l'URL en conséquence. Note de sécurité :

-   La vérification de signature LINE dépend du corps (HMAC sur le corps brut), donc OpenClaw applique des limites strictes de pré-authentification du corps et un délai d'attente avant la vérification.

## Configurer

Configuration minimale :

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Variables d'environnement (compte par défaut uniquement) :

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

Fichiers de jeton/secret :

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

Comptes multiples :

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## Contrôle d'accès

Les messages directs utilisent par défaut l'appairage. Les expéditeurs inconnus reçoivent un code d'appairage et leurs messages sont ignorés jusqu'à approbation.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Listes d'autorisation et politiques :

-   `channels.line.dmPolicy` : `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom` : identifiants utilisateur LINE autorisés pour les messages directs
-   `channels.line.groupPolicy` : `allowlist | open | disabled`
-   `channels.line.groupAllowFrom` : identifiants utilisateur LINE autorisés pour les groupes
-   Remplacements par groupe : `channels.line.groups..allowFrom`
-   Note d'exécution : si `channels.line` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

Les identifiants LINE sont sensibles à la casse. Les identifiants valides ressemblent à :

-   Utilisateur : `U` + 32 caractères hexadécimaux
-   Groupe : `C` + 32 caractères hexadécimaux
-   Salon : `R` + 32 caractères hexadécimaux

## Comportement des messages

-   Le texte est découpé par blocs de 5000 caractères.
-   Le formatage Markdown est supprimé ; les blocs de code et les tableaux sont convertis en cartes Flex lorsque possible.
-   Les réponses en flux sont mises en mémoire tampon ; LINE reçoit des blocs complets avec une animation de chargement pendant que l'agent travaille.
-   Les téléchargements de médias sont limités par `channels.line.mediaMaxMb` (par défaut 10).

## Données de canal (messages enrichis)

Utilisez `channelData.line` pour envoyer des réponses rapides, des localisations, des cartes Flex ou des messages modèles.

```json
{
  text: "Voici",
  channelData: {
    line: {
      quickReplies: ["Statut", "Aide"],
      location: {
        title: "Bureau",
        address: "123 Rue Principale",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Carte de statut",
        contents: {
          /* Payload Flex */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Procéder ?",
        confirmLabel: "Oui",
        confirmData: "yes",
        cancelLabel: "Non",
        cancelData: "no",
      },
    },
  },
}
```

Le plugin LINE fournit également une commande `/card` pour des préréglages de messages Flex :

```bash
/card info "Bienvenue" "Merci de nous avoir rejoint !"
```

## Dépannage

-   **Échec de la vérification du webhook :** assurez-vous que l'URL du webhook est en HTTPS et que le `channelSecret` correspond à celui de la console LINE.
-   **Aucun événement entrant :** confirmez que le chemin du webhook correspond à `channels.line.webhookPath` et que la passerelle est accessible depuis LINE.
-   **Erreurs de téléchargement de médias :** augmentez `channels.line.mediaMaxMb` si le média dépasse la limite par défaut.

[IRC](./irc.md)[Matrix](./matrix.md)