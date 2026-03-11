

  Plateformes de messagerie

  
# Mattermost

Statut : pris en charge via plugin (jeton de bot + événements WebSocket). Les canaux, groupes et messages privés sont pris en charge. Mattermost est une plateforme de messagerie d'équipe auto-hébergeable ; consultez le site officiel [mattermost.com](https://mattermost.com) pour les détails du produit et les téléchargements.

## Plugin requis

Mattermost est fourni sous forme de plugin et n'est pas inclus dans l'installation principale. Installez-le via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/mattermost
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/mattermost
```

Si vous choisissez Mattermost pendant la configuration/le démarrage et qu'un dépôt git est détecté, OpenClaw proposera automatiquement le chemin d'installation local. Détails : [Plugins](../tools/plugin.md)

## Configuration rapide

1.  Installez le plugin Mattermost.
2.  Créez un compte bot Mattermost et copiez le **jeton du bot**.
3.  Copiez l'**URL de base** Mattermost (par exemple, `https://chat.example.com`).
4.  Configurez OpenClaw et démarrez la passerelle.

Configuration minimale :

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Commandes slash natives

Les commandes slash natives sont facultatives. Lorsqu'elles sont activées, OpenClaw enregistre les commandes `oc_*` via l'API Mattermost et reçoit des rappels POST sur le serveur HTTP de la passerelle.

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // À utiliser lorsque Mattermost ne peut pas atteindre directement la passerelle (proxy inverse/URL publique).
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

Notes :

-   `native: "auto"` est désactivé par défaut pour Mattermost. Définissez `native: true` pour l'activer.
-   Si `callbackUrl` est omis, OpenClaw en déduit une à partir de l'hôte/port de la passerelle + `callbackPath`.
-   Pour les configurations multi-comptes, `commands` peut être défini au niveau supérieur ou sous `channels.mattermost.accounts..commands` (les valeurs du compte écrasent les champs de niveau supérieur).
-   Les rappels de commandes sont validés avec des jetons par commande et échouent en mode fermé lorsque les vérifications de jeton échouent.
-   Exigence d'accessibilité : le point de terminaison de rappel doit être accessible depuis le serveur Mattermost.
    -   Ne définissez pas `callbackUrl` sur `localhost` sauf si Mattermost s'exécute sur le même hôte/espace de noms réseau qu'OpenClaw.
    -   Ne définissez pas `callbackUrl` sur votre URL de base Mattermost sauf si cette URL reverse-proxy `/api/channels/mattermost/command` vers OpenClaw.
    -   Une vérification rapide est `curl https://<gateway-host>/api/channels/mattermost/command` ; un GET devrait renvoyer `405 Method Not Allowed` d'OpenClaw, pas `404`.
-   Exigence de liste blanche de sortie Mattermost :
    -   Si votre rappel cible des adresses privées/tailnet/interne, définissez `ServiceSettings.AllowedUntrustedInternalConnections` de Mattermost pour inclure l'hôte/domaine du rappel.
    -   Utilisez des entrées hôte/domaine, pas des URL complètes.
        -   Bon : `gateway.tailnet-name.ts.net`
        -   Mauvais : `https://gateway.tailnet-name.ts.net`

## Variables d'environnement (compte par défaut)

Définissez-les sur l'hôte de la passerelle si vous préférez les variables d'environnement :

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

Les variables d'environnement s'appliquent uniquement au compte **par défaut** (`default`). Les autres comptes doivent utiliser les valeurs de configuration.

## Modes de discussion

Mattermost répond automatiquement aux messages privés. Le comportement dans les canaux est contrôlé par `chatmode` :

-   `oncall` (par défaut) : répond uniquement lorsqu'il est @mentionné dans les canaux.
-   `onmessage` : répond à chaque message de canal.
-   `onchar` : répond lorsqu'un message commence par un préfixe déclencheur.

Exemple de configuration :

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Notes :

-   `onchar` répond toujours aux @mentions explicites.
-   `channels.mattermost.requireMention` est respecté pour les configurations héritées, mais `chatmode` est préféré.

## Contrôle d'accès (messages privés)

-   Par défaut : `channels.mattermost.dmPolicy = "pairing"` (les expéditeurs inconnus reçoivent un code d'appairage).
-   Approuvez via :
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   Messages privés publics : `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.

## Canaux (groupes)

-   Par défaut : `channels.mattermost.groupPolicy = "allowlist"` (protégé par mention).
-   Liste blanche des expéditeurs avec `channels.mattermost.groupAllowFrom` (les ID utilisateur sont recommandés).
-   La correspondance `@nomutilisateur` est mutable et n'est activée que lorsque `channels.mattermost.dangerouslyAllowNameMatching: true`.
-   Canaux ouverts : `channels.mattermost.groupPolicy="open"` (protégé par mention).
-   Note d'exécution : si `channels.mattermost` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

## Cibles pour l'envoi sortant

Utilisez ces formats de cible avec `openclaw message send` ou les cron/webhooks :

-   `channel:` pour un canal
-   `user:` pour un message privé
-   `@nomutilisateur` pour un message privé (résolu via l'API Mattermost)

Les ID nus sont traités comme des canaux.

## Réactions (outil de message)

-   Utilisez `message action=react` avec `channel=mattermost`.
-   `messageId` est l'id de publication Mattermost.
-   `emoji` accepte des noms comme `thumbsup` ou `:+1:` (les deux-points sont facultatifs).
-   Définissez `remove=true` (booléen) pour supprimer une réaction.
-   Les événements d'ajout/suppression de réaction sont transmis comme événements système à la session de l'agent routé.

Exemples :

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Configuration :

-   `channels.mattermost.actions.reactions` : active/désactive les actions de réaction (vrai par défaut).
-   Remplacement par compte : `channels.mattermost.accounts..actions.reactions`.

## Boutons interactifs (outil de message)

Envoyez des messages avec des boutons cliquables. Lorsqu'un utilisateur clique sur un bouton, l'agent reçoit la sélection et peut répondre. Activez les boutons en ajoutant `inlineButtons` aux capacités du canal :

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

Utilisez `message action=send` avec un paramètre `buttons`. Les boutons sont un tableau 2D (lignes de boutons) :

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Oui","callback_data":"yes"},{"text":"Non","callback_data":"no"}]]
```

Champs des boutons :

-   `text` (obligatoire) : libellé d'affichage.
-   `callback_data` (obligatoire) : valeur renvoyée au clic (utilisée comme ID d'action).
-   `style` (facultatif) : `"default"`, `"primary"`, ou `"danger"`.

Lorsqu'un utilisateur clique sur un bouton :

1.  Tous les boutons sont remplacés par une ligne de confirmation (par exemple, ”✓ **Oui** sélectionné par @utilisateur”).
2.  L'agent reçoit la sélection comme un message entrant et répond.

Notes :

-   Les rappels de boutons utilisent une vérification HMAC-SHA256 (automatique, aucune configuration nécessaire).
-   Mattermost supprime les données de rappel de ses réponses API (fonctionnalité de sécurité), donc tous les boutons sont supprimés au clic — une suppression partielle n'est pas possible.
-   Les ID d'action contenant des traits d'union ou des tirets bas sont automatiquement nettoyés (limitation de routage Mattermost).

Configuration :

-   `channels.mattermost.capabilities` : tableau de chaînes de capacités. Ajoutez `"inlineButtons"` pour activer la description de l'outil de boutons dans l'invite système de l'agent.
-   `channels.mattermost.interactions.callbackBaseUrl` : URL de base externe facultative pour les rappels de boutons (par exemple `https://gateway.example.com`). Utilisez-la lorsque Mattermost ne peut pas atteindre la passerelle directement à son hôte de liaison.
-   Dans les configurations multi-comptes, vous pouvez également définir le même champ sous `channels.mattermost.accounts..interactions.callbackBaseUrl`.
-   Si `interactions.callbackBaseUrl` est omis, OpenClaw déduit l'URL de rappel de `gateway.customBindHost` + `gateway.port`, puis revient à `http://localhost:`.
-   Règle d'accessibilité : l'URL de rappel des boutons doit être accessible depuis le serveur Mattermost. `localhost` ne fonctionne que lorsque Mattermost et OpenClaw s'exécutent sur le même hôte/espace de noms réseau.
-   Si votre cible de rappel est privée/tailnet/interne, ajoutez son hôte/domaine à `ServiceSettings.AllowedUntrustedInternalConnections` de Mattermost.

### Intégration API directe (scripts externes)

Les scripts externes et les webhooks peuvent publier des boutons directement via l'API REST Mattermost au lieu de passer par l'outil `message` de l'agent. Utilisez `buildButtonAttachments()` de l'extension si possible ; si vous publiez du JSON brut, suivez ces règles : **Structure de la charge utile :**

```json
{
  channel_id: "<channelId>",
  message: "Choisissez une option :",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // alphanumérique uniquement — voir ci-dessous
            type: "button", // requis, sinon les clics sont ignorés silencieusement
            name: "Approuver", // libellé d'affichage
            style: "primary", // facultatif : "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // doit correspondre à l'id du bouton (pour la recherche de nom)
                action: "approve",
                // ... tout champ personnalisé ...
                _token: "<hmac>", // voir section HMAC ci-dessous
              },
            },
          },
        ],
      },
    ],
  },
}
```

**Règles critiques :**

1.  Les pièces jointes vont dans `props.attachments`, pas dans `attachments` de niveau supérieur (ignorées silencieusement).
2.  Chaque action nécessite `type: "button"` — sans cela, les clics sont ignorés silencieusement.
3.  Chaque action nécessite un champ `id` — Mattermost ignore les actions sans ID.
4.  L'`id` de l'action doit être **uniquement alphanumérique** (`[a-zA-Z0-9]`). Les traits d'union et les tirets bas cassent le routage côté serveur de Mattermost (renvoie 404). Supprimez-les avant utilisation.
5.  `context.action_id` doit correspondre à l'`id` du bouton pour que le message de confirmation affiche le nom du bouton (par exemple, "Approuver") au lieu d'un ID brut.
6.  `context.action_id` est requis — le gestionnaire d'interaction renvoie 400 sans lui.

**Génération du jeton HMAC :** La passerelle vérifie les clics de boutons avec HMAC-SHA256. Les scripts externes doivent générer des jetons qui correspondent à la logique de vérification de la passerelle :

1.  Dérivez le secret du jeton du bot : `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  Construisez l'objet context avec tous les champs **sauf** `_token`.
3.  Sérialisez avec **clés triées** et **sans espaces** (la passerelle utilise `JSON.stringify` avec clés triées, ce qui produit une sortie compacte).
4.  Signez : `HMAC-SHA256(key=secret, data=serializedContext)`
5.  Ajoutez le condensé hexadécimal résultant comme `_token` dans le contexte.

Exemple Python :

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

Pièges courants HMAC :

-   `json.dumps` de Python ajoute des espaces par défaut (`{"key": "val"}`). Utilisez `separators=(",", ":")` pour correspondre à la sortie compacte de JavaScript (`{"key":"val"}`).
-   Signez toujours **tous** les champs du contexte (moins `_token`). La passerelle supprime `_token` puis signe tout le reste. Signer un sous-ensemble provoque un échec silencieux de la vérification.
-   Utilisez `sort_keys=True` — la passerelle trie les clés avant de signer, et Mattermost peut réorganiser les champs du contexte lors du stockage de la charge utile.
-   Dérivez le secret du jeton du bot (déterministe), pas d'octets aléatoires. Le secret doit être le même entre le processus qui crée les boutons et la passerelle qui vérifie.

## Adaptateur d'annuaire

Le plugin Mattermost inclut un adaptateur d'annuaire qui résout les noms de canaux et d'utilisateurs via l'API Mattermost. Cela permet d'utiliser les cibles `#nom-canal` et `@nomutilisateur` dans `openclaw message send` et les livraisons cron/webhook. Aucune configuration n'est nécessaire — l'adaptateur utilise le jeton du bot de la configuration du compte.

## Multi-comptes

Mattermost prend en charge plusieurs comptes sous `channels.mattermost.accounts` :

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Principal", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alertes", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Dépannage

-   Pas de réponses dans les canaux : assurez-vous que le bot est dans le canal et mentionnez-le (oncall), utilisez un préfixe déclencheur (onchar), ou définissez `chatmode: "onmessage"`.
-   Erreurs d'authentification : vérifiez le jeton du bot, l'URL de base et si le compte est activé.
-   Problèmes multi-comptes : les variables d'environnement s'appliquent uniquement au compte `default`.
-   Les boutons apparaissent comme des boîtes blanches : l'agent peut envoyer des données de bouton mal formées. Vérifiez que chaque bouton a les champs `text` et `callback_data`.
-   Les boutons s'affichent mais les clics ne font rien : vérifiez que `AllowedUntrustedInternalConnections` dans la configuration du serveur Mattermost inclut `127.0.0.1 localhost`, et que `EnablePostActionIntegration` est `true` dans ServiceSettings.
-   Les boutons renvoient 404 au clic : l'`id` du bouton contient probablement des traits d'union ou des tirets bas. Le routeur d'actions de Mattermost échoue sur les ID non alphanumériques. Utilisez uniquement `[a-zA-Z0-9]`.
-   Les journaux de la passerelle indiquent `invalid _token` : discordance HMAC. Vérifiez que vous signez tous les champs du contexte (pas un sous-ensemble), utilisez des clés triées et du JSON compact (sans espaces). Voir la section HMAC ci-dessus.
-   Les journaux de la passerelle indiquent `missing _token in context` : le champ `_token` n'est pas dans le contexte du bouton. Assurez-vous qu'il est inclus lors de la construction de la charge utile d'intégration.
-   La confirmation affiche un ID brut au lieu du nom du bouton : `context.action_id` ne correspond pas à l'`id` du bouton. Définissez les deux à la même valeur nettoyée.
-   L'agent ne connaît pas les boutons : ajoutez `capabilities: ["inlineButtons"]` à la configuration du canal Mattermost.

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)