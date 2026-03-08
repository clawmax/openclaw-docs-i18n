

  Plateformes de messagerie

  
# Matrix

Matrix est un protocole de messagerie ouvert et décentralisé. OpenClaw se connecte en tant qu'**utilisateur** Matrix sur n'importe quel serveur d'accueil (homeserver), vous avez donc besoin d'un compte Matrix pour le bot. Une fois connecté, vous pouvez envoyer des messages directs au bot ou l'inviter dans des salons (les "groupes" Matrix). Beeper est également une option client valide, mais il nécessite que le chiffrement de bout en bout (E2EE) soit activé. Statut : supporté via plugin (@vector-im/matrix-bot-sdk). Messages directs, salons, fils de discussion, médias, réactions, sondages (envoi + début de sondage sous forme de texte), localisation et E2EE (avec support cryptographique).

## Plugin requis

Matrix est fourni sous forme de plugin et n'est pas inclus dans l'installation principale. Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/matrix
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/matrix
```

Si vous choisissez Matrix pendant la configuration/onboarding et qu'un dépôt git est détecté, OpenClaw proposera automatiquement le chemin d'installation local. Détails : [Plugins](../tools/plugin.md)

## Configuration

1.  Installez le plugin Matrix :
    -   Depuis npm : `openclaw plugins install @openclaw/matrix`
    -   Depuis un dépôt local : `openclaw plugins install ./extensions/matrix`
2.  Créez un compte Matrix sur un serveur d'accueil :
    -   Consultez les options d'hébergement sur [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
    -   Ou hébergez-le vous-même.
3.  Obtenez un jeton d'accès pour le compte du bot :
    
    -   Utilisez l'API de connexion Matrix avec `curl` sur votre serveur d'accueil :
    
    Copier
    
    ```bash
    curl --request POST \
      --url https://matrix.example.org/_matrix/client/v3/login \
      --header 'Content-Type: application/json' \
      --data '{
      "type": "m.login.password",
      "identifier": {
        "type": "m.id.user",
        "user": "your-user-name"
      },
      "password": "your-password"
    }'
    ```
    
    -   Remplacez `matrix.example.org` par l'URL de votre serveur d'accueil.
    -   Ou définissez `channels.matrix.userId` + `channels.matrix.password` : OpenClaw appelle le même point de terminaison de connexion, stocke le jeton d'accès dans `~/.openclaw/credentials/matrix/credentials.json`, et le réutilise au prochain démarrage.
4.  Configurez les identifiants :
    -   Variables d'environnement : `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (ou `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
    -   Ou dans la configuration : `channels.matrix.*`
    -   Si les deux sont définis, la configuration a la priorité.
    -   Avec un jeton d'accès : l'identifiant utilisateur est récupéré automatiquement via `/whoami`.
    -   Lorsqu'il est défini, `channels.matrix.userId` doit être l'identifiant Matrix complet (exemple : `@bot:example.org`).
5.  Redémarrez la passerelle (ou terminez l'onboarding).
6.  Démarrez une conversation directe avec le bot ou invitez-le dans un salon depuis n'importe quel client Matrix (Element, Beeper, etc. ; voir [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). Beeper nécessite le chiffrement E2EE, donc définissez `channels.matrix.encryption: true` et vérifiez l'appareil.

Configuration minimale (jeton d'accès, identifiant utilisateur récupéré automatiquement) :

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

Configuration E2EE (chiffrement de bout en bout activé) :

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## Chiffrement (E2EE)

Le chiffrement de bout en bout est **supporté** via le SDK cryptographique Rust. Activez-le avec `channels.matrix.encryption: true` :

-   Si le module cryptographique se charge, les salons chiffrés sont déchiffrés automatiquement.
-   Les médias sortants sont chiffrés lors de l'envoi vers des salons chiffrés.
-   Lors de la première connexion, OpenClaw demande la vérification de l'appareil depuis vos autres sessions.
-   Vérifiez l'appareil dans un autre client Matrix (Element, etc.) pour activer le partage de clés.
-   Si le module cryptographique ne peut pas être chargé, le chiffrement E2EE est désactivé et les salons chiffrés ne seront pas déchiffrés ; OpenClaw enregistre un avertissement.
-   Si vous voyez des erreurs de module cryptographique manquant (par exemple, `@matrix-org/matrix-sdk-crypto-nodejs-*`), autorisez les scripts de construction pour `@matrix-org/matrix-sdk-crypto-nodejs` et exécutez `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` ou récupérez le binaire avec `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

L'état cryptographique est stocké par compte + jeton d'accès dans `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/` (base de données SQLite). L'état de synchronisation se trouve à côté dans `bot-storage.json`. Si le jeton d'accès (appareil) change, un nouveau stockage est créé et le bot doit être re-vérifié pour les salons chiffrés. **Vérification de l'appareil :** Lorsque le chiffrement E2EE est activé, le bot demandera une vérification depuis vos autres sessions au démarrage. Ouvrez Element (ou un autre client) et approuvez la demande de vérification pour établir la confiance. Une fois vérifié, le bot peut déchiffrer les messages dans les salons chiffrés.

## Multi-comptes

Support multi-comptes : utilisez `channels.matrix.accounts` avec des identifiants par compte et un `name` optionnel. Voir [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modèle partagé. Chaque compte s'exécute comme un utilisateur Matrix distinct sur n'importe quel serveur d'accueil. La configuration par compte hérite des paramètres de niveau supérieur `channels.matrix` et peut remplacer n'importe quelle option (politique de messages directs, groupes, chiffrement, etc.).

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "Assistant principal",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "Bot d'alertes",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

Notes :

-   Le démarrage des comptes est sérialisé pour éviter les conditions de concurrence avec les imports de modules simultanés.
-   Les variables d'environnement (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, etc.) s'appliquent uniquement au compte **par défaut**.
-   Les paramètres de base du canal (politique de messages directs, politique de groupe, mention gating, etc.) s'appliquent à tous les comptes sauf s'ils sont remplacés par compte.
-   Utilisez `bindings[].match.accountId` pour router chaque compte vers un agent différent.
-   L'état cryptographique est stocké par compte + jeton d'accès (stockages de clés séparés par compte).

## Modèle de routage

-   Les réponses vont toujours vers Matrix.
-   Les messages directs partagent la session principale de l'agent ; les salons sont mappés à des sessions de groupe.

## Contrôle d'accès (messages directs)

-   Par défaut : `channels.matrix.dm.policy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage.
-   Approuvez via :
    -   `openclaw pairing list matrix`
    -   `openclaw pairing approve matrix `
-   Messages directs publics : `channels.matrix.dm.policy="open"` plus `channels.matrix.dm.allowFrom=["*"]`.
-   `channels.matrix.dm.allowFrom` accepte les identifiants utilisateur Matrix complets (exemple : `@user:server`). L'assistant de configuration résout les noms d'affichage en identifiants utilisateur lorsque la recherche dans l'annuaire trouve un seul correspondance exacte.
-   N'utilisez pas les noms d'affichage ou les parties locales seules (exemple : `"Alice"` ou `"alice"`). Ils sont ambigus et sont ignorés pour la correspondance de la liste d'autorisation. Utilisez les identifiants complets `@user:server`.

## Salons (groupes)

-   Par défaut : `channels.matrix.groupPolicy = "allowlist"` (conditionné par mention). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu'elle n'est pas définie.
-   Note d'exécution : si `channels.matrix` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de salon (même si `channels.defaults.groupPolicy` est défini).
-   Autorisez les salons avec `channels.matrix.groups` (identifiants de salon ou alias ; les noms sont résolus en identifiants lorsque la recherche dans l'annuaire trouve une seule correspondance exacte) :

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

-   `requireMention: false` active la réponse automatique dans ce salon.
-   `groups."*"` peut définir des valeurs par défaut pour le conditionnement par mention dans tous les salons.
-   `groupAllowFrom` restreint les expéditeurs qui peuvent déclencher le bot dans les salons (identifiants utilisateur Matrix complets).
-   Les listes d'autorisation `users` par salon peuvent restreindre davantage les expéditeurs à l'intérieur d'un salon spécifique (utilisez des identifiants utilisateur Matrix complets).
-   L'assistant de configuration demande des listes d'autorisation de salons (identifiants de salon, alias ou noms) et ne résout les noms qu'en cas de correspondance exacte et unique.
-   Au démarrage, OpenClaw résout les noms de salon/utilisateur dans les listes d'autorisation en identifiants et enregistre le mappage ; les entrées non résolues sont ignorées pour la correspondance de la liste d'autorisation.
-   Les invitations sont automatiquement acceptées par défaut ; contrôlez cela avec `channels.matrix.autoJoin` et `channels.matrix.autoJoinAllowlist`.
-   Pour n'autoriser **aucun salon**, définissez `channels.matrix.groupPolicy: "disabled"` (ou gardez une liste d'autorisation vide).
-   Clé héritée : `channels.matrix.rooms` (même forme que `groups`).

## Fils de discussion

-   Le threading des réponses est supporté.
-   `channels.matrix.threadReplies` contrôle si les réponses restent dans les fils de discussion :
    -   `off`, `inbound` (par défaut), `always`
-   `channels.matrix.replyToMode` contrôle les métadonnées de réponse lorsque la réponse n'est pas dans un fil de discussion :
    -   `off` (par défaut), `first`, `all`

## Capacités

| Fonctionnalité | Statut |
| --- | --- |
| Messages directs | ✅ Supporté |
| Salons | ✅ Supporté |
| Fils de discussion | ✅ Supporté |
| Médias | ✅ Supporté |
| Chiffrement E2EE | ✅ Supporté (module cryptographique requis) |
| Réactions | ✅ Supporté (envoi/lecture via outils) |
| Sondages | ✅ Envoi supporté ; les débuts de sondage entrants sont convertis en texte (réponses/fins ignorées) |
| Localisation | ✅ Supporté (URI géo ; altitude ignorée) |
| Commandes natives | ✅ Supporté |

## Dépannage

Exécutez d'abord cette échelle :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis confirmez l'état d'appairage des messages directs si nécessaire :

```bash
openclaw pairing list matrix
```

Échecs courants :

-   Connecté mais les messages de salon ignorés : salon bloqué par `groupPolicy` ou liste d'autorisation de salon.
-   Messages directs ignorés : expéditeur en attente d'approbation lorsque `channels.matrix.dm.policy="pairing"`.
-   Les salons chiffrés échouent : support cryptographique ou paramètres de chiffrement incompatibles.

Pour le flux de triage : [/channels/troubleshooting](./troubleshooting.md).

## Référence de configuration (Matrix)

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.matrix.enabled` : activer/désactiver le démarrage du canal.
-   `channels.matrix.homeserver` : URL du serveur d'accueil.
-   `channels.matrix.userId` : Identifiant utilisateur Matrix (optionnel avec jeton d'accès).
-   `channels.matrix.accessToken` : jeton d'accès.
-   `channels.matrix.password` : mot de passe pour la connexion (jeton stocké).
-   `channels.matrix.deviceName` : nom d'affichage de l'appareil.
-   `channels.matrix.encryption` : activer le chiffrement E2EE (par défaut : false).
-   `channels.matrix.initialSyncLimit` : limite de synchronisation initiale.
-   `channels.matrix.threadReplies` : `off | inbound | always` (par défaut : inbound).
-   `channels.matrix.textChunkLimit` : taille des morceaux de texte sortants (caractères).
-   `channels.matrix.chunkMode` : `length` (par défaut) ou `newline` pour diviser sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   `channels.matrix.dm.policy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
-   `channels.matrix.dm.allowFrom` : liste d'autorisation pour les messages directs (identifiants utilisateur Matrix complets). `open` nécessite `"*"`. L'assistant résout les noms en identifiants lorsque possible.
-   `channels.matrix.groupPolicy` : `allowlist | open | disabled` (par défaut : allowlist).
-   `channels.matrix.groupAllowFrom` : expéditeurs autorisés pour les messages de groupe (identifiants utilisateur Matrix complets).
-   `channels.matrix.allowlistOnly` : forcer les règles de liste d'autorisation pour les messages directs + salons.
-   `channels.matrix.groups` : liste d'autorisation de groupes + carte des paramètres par salon.
-   `channels.matrix.rooms` : liste d'autorisation/config de groupe héritée.
-   `channels.matrix.replyToMode` : mode de réponse pour les fils de discussion/étiquettes.
-   `channels.matrix.mediaMaxMb` : plafond des médias entrants/sortants (Mo).
-   `channels.matrix.autoJoin` : gestion des invitations (`always | allowlist | off`, par défaut : always).
-   `channels.matrix.autoJoinAllowlist` : identifiants/alias de salon autorisés pour l'auto-rejoindre.
-   `channels.matrix.accounts` : configuration multi-comptes indexée par identifiant de compte (chaque compte hérite des paramètres de niveau supérieur).
-   `channels.matrix.actions` : conditionnement des outils par action (réactions/messages/épinglages/infoMembre/infoCanal).

[LINE](./line.md)[Mattermost](./mattermost.md)