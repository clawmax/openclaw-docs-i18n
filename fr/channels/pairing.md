

  Configuration

  
# Appairage

L'« appairage » est l'étape d'**approbation explicite du propriétaire** dans OpenClaw. Il est utilisé à deux endroits :

1.  **Appairage DM** (qui est autorisé à parler au bot)
2.  **Appairage de nœud** (quels appareils/nœuds sont autorisés à rejoindre le réseau de la passerelle)

Contexte de sécurité : [Sécurité](../gateway/security.md)

## 1) Appairage DM (accès aux discussions entrantes)

Lorsqu'un canal est configuré avec la politique DM `pairing`, les expéditeurs inconnus reçoivent un code court et leur message **n'est pas traité** jusqu'à votre approbation. Les politiques DM par défaut sont documentées dans : [Sécurité](../gateway/security.md) Codes d'appairage :

-   8 caractères, majuscules, pas de caractères ambigus (`0O1I`).
-   **Expire après 1 heure**. Le bot n'envoie le message d'appairage que lorsqu'une nouvelle demande est créée (environ une fois par heure par expéditeur).
-   Les demandes d'appairage DM en attente sont limitées à **3 par canal** par défaut ; les demandes supplémentaires sont ignorées jusqu'à ce qu'une expire ou soit approuvée.

### Approuver un expéditeur

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canaux pris en charge : `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`, `feishu`.

### Où l'état est stocké

Stocké sous `~/.openclaw/credentials/` :

-   Demandes en attente : `-pairing.json`
-   Stockage de la liste d'autorisation approuvée :
    -   Compte par défaut : `-allowFrom.json`
    -   Compte non par défaut : `--allowFrom.json`

Comportement de portée du compte :

-   Les comptes non par défaut lisent et écrivent uniquement leur fichier de liste d'autorisation à portée limitée.
-   Le compte par défaut utilise le fichier de liste d'autorisation non limité à la portée du canal.

Traitez ces fichiers comme sensibles (ils contrôlent l'accès à votre assistant).

## 2) Appairage d'appareil Node (nœuds iOS/Android/macOS/headless)

Les nœuds se connectent à la Passerelle en tant qu'**appareils** avec `role: node`. La Passerelle crée une demande d'appairage d'appareil qui doit être approuvée.

### Appairer via Telegram (recommandé pour iOS)

Si vous utilisez le plugin `device-pair`, vous pouvez effectuer l'appairage initial d'un appareil entièrement depuis Telegram :

1.  Dans Telegram, envoyez un message à votre bot : `/pair`
2.  Le bot répond avec deux messages : un message d'instructions et un message séparé contenant le **code de configuration** (facile à copier/coller dans Telegram).
3.  Sur votre téléphone, ouvrez l'application OpenClaw iOS → Paramètres → Passerelle.
4.  Collez le code de configuration et connectez-vous.
5.  De retour dans Telegram : `/pair approve`

Le code de configuration est une charge utile JSON encodée en base64 qui contient :

-   `url` : l'URL WebSocket de la Passerelle (`ws://...` ou `wss://...`)
-   `token` : un jeton d'appairage à courte durée de vie

Traitez le code de configuration comme un mot de passe tant qu'il est valide.

### Approuver un appareil nœud

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Stockage de l'état d'appairage de nœud

Stocké sous `~/.openclaw/devices/` :

-   `pending.json` (à courte durée de vie ; les demandes en attente expirent)
-   `paired.json` (appareils appairés + jetons)

### Notes

-   L'API héritée `node.pair.*` (CLI : `openclaw nodes pending/approve`) est un stockage d'appairage distinct appartenant à la passerelle. Les nœuds WS nécessitent toujours l'appairage d'appareil.

## Documentation associée

-   Modèle de sécurité + injection d'invite : [Sécurité](../gateway/security.md)
-   Mise à jour en toute sécurité (exécuter doctor) : [Mise à jour](../install/updating.md)
-   Configurations de canal :
    -   Telegram : [Telegram](./telegram.md)
    -   WhatsApp : [WhatsApp](./whatsapp.md)
    -   Signal : [Signal](./signal.md)
    -   BlueBubbles (iMessage) : [BlueBubbles](./bluebubbles.md)
    -   iMessage (hérité) : [iMessage](./imessage.md)
    -   Discord : [Discord](./discord.md)
    -   Slack : [Slack](./slack.md)

[Zalo Personnel](./zalouser.md)[Messages de groupe](./group-messages.md)