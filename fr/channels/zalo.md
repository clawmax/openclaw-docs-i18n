

  Plateformes de messagerie

  
# Zalo

Statut : expérimental. Les messages privés sont pris en charge ; la gestion des groupes est disponible avec des contrôles explicites de politique de groupe.

## Plugin requis

Zalo est fourni sous forme de plugin et n'est pas inclus dans l'installation de base.

-   Installation via CLI : `openclaw plugins install @openclaw/zalo`
-   Ou sélectionnez **Zalo** pendant l'intégration et confirmez l'invite d'installation
-   Détails : [Plugins](../tools/plugin.md)

## Configuration rapide (débutant)

1.  Installez le plugin Zalo :
    -   À partir d'un dépôt source : `openclaw plugins install ./extensions/zalo`
    -   À partir de npm (si publié) : `openclaw plugins install @openclaw/zalo`
    -   Ou choisissez **Zalo** lors de l'intégration et confirmez l'invite d'installation
2.  Définissez le jeton :
    -   Variable d'environnement : `ZALO_BOT_TOKEN=...`
    -   Ou configuration : `channels.zalo.botToken: "..."`.
3.  Redémarrez la passerelle (ou terminez l'intégration).
4.  L'accès aux messages privés utilise l'appairage par défaut ; approuvez le code d'appairage au premier contact.

Configuration minimale :

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## Qu'est-ce que c'est

Zalo est une application de messagerie axée sur le Vietnam ; son API Bot permet à la Passerelle d'exécuter un bot pour des conversations 1:1. Elle convient bien pour le support ou les notifications où vous souhaitez un routage déterministe vers Zalo.

-   Un canal d'API Bot Zalo détenu par la Passerelle.
-   Routage déterministe : les réponses retournent vers Zalo ; le modèle ne choisit jamais les canaux.
-   Les messages privés partagent la session principale de l'agent.
-   Les groupes sont pris en charge avec des contrôles de politique (`groupPolicy` + `groupAllowFrom`) et adoptent par défaut un comportement de liste autorisée en mode "échec-fermé".

## Configuration (chemin rapide)

### 1) Créer un jeton de bot (Plateforme Bot Zalo)

1.  Rendez-vous sur [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) et connectez-vous.
2.  Créez un nouveau bot et configurez ses paramètres.
3.  Copiez le jeton du bot (format : `12345689:abc-xyz`).

### 2) Configurer le jeton (variable d'env. ou config)

Exemple :

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

Option variable d'environnement : `ZALO_BOT_TOKEN=...` (fonctionne uniquement pour le compte par défaut). Prise en charge multi-comptes : utilisez `channels.zalo.accounts` avec des jetons par compte et un `name` optionnel.

3.  Redémarrez la passerelle. Zalo démarre lorsqu'un jeton est résolu (variable d'environnement ou configuration).
4.  L'accès aux messages privés utilise l'appairage par défaut. Approuvez le code lorsque le bot est contacté pour la première fois.

## Fonctionnement (comportement)

-   Les messages entrants sont normalisés dans l'enveloppe de canal partagée avec des espaces réservés pour les médias.
-   Les réponses sont toujours routées vers la même conversation Zalo.
-   Long-polling par défaut ; mode webhook disponible avec `channels.zalo.webhookUrl`.

## Limites

-   Le texte sortant est découpé en segments de 2000 caractères (limite de l'API Zalo).
-   Les téléchargements/chargements de médias sont limités par `channels.zalo.mediaMaxMb` (5 par défaut).
-   Le streaming est bloqué par défaut en raison de la limite de 2000 caractères qui le rend moins utile.

## Contrôle d'accès (messages privés)

### Accès aux messages privés

-   Par défaut : `channels.zalo.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'à approbation (les codes expirent après 1 heure).
-   Approuvez via :
    -   `openclaw pairing list zalo`
    -   `openclaw pairing approve zalo `
-   L'appairage est l'échange de jetons par défaut. Détails : [Appairage](./pairing.md)
-   `channels.zalo.allowFrom` accepte des identifiants utilisateur numériques (aucune recherche par nom d'utilisateur disponible).

## Contrôle d'accès (Groupes)

-   `channels.zalo.groupPolicy` contrôle la gestion des messages entrants des groupes : `open | allowlist | disabled`.
-   Le comportement par défaut est en mode "échec-fermé" : `allowlist`.
-   `channels.zalo.groupAllowFrom` restreint les identifiants d'expéditeurs pouvant déclencher le bot dans les groupes.
-   Si `groupAllowFrom` n'est pas défini, Zalo utilise `allowFrom` pour les vérifications d'expéditeur.
-   `groupPolicy: "disabled"` bloque tous les messages de groupe.
-   `groupPolicy: "open"` autorise tout membre du groupe (sous condition de mention).
-   Note d'exécution : si `channels.zalo` est entièrement absent, l'exécution utilise tout de même `groupPolicy="allowlist"` par sécurité.

## Long-polling vs webhook

-   Par défaut : long-polling (aucune URL publique requise).
-   Mode webhook : définissez `channels.zalo.webhookUrl` et `channels.zalo.webhookSecret`.
    -   Le secret du webhook doit comporter entre 8 et 256 caractères.
    -   L'URL du webhook doit utiliser HTTPS.
    -   Zalo envoie des événements avec l'en-tête `X-Bot-Api-Secret-Token` pour vérification.
    -   Le serveur HTTP de la passerelle gère les requêtes webhook à `channels.zalo.webhookPath` (par défaut le chemin de l'URL webhook).
    -   Les requêtes doivent utiliser `Content-Type: application/json` (ou les types média `+json`).
    -   Les événements en double (`event_name + message_id`) sont ignorés pendant une courte fenêtre de rejeu.
    -   Le trafic en rafale est limité par chemin/source et peut renvoyer HTTP 429.

**Note :** getUpdates (polling) et webhook sont mutuellement exclusifs selon la documentation de l'API Zalo.

## Types de messages pris en charge

-   **Messages texte** : Prise en charge complète avec découpage en segments de 2000 caractères.
-   **Messages image** : Téléchargement et traitement des images entrantes ; envoi d'images via `sendPhoto`.
-   **Autocollants** : Journalisés mais non entièrement traités (pas de réponse de l'agent).
-   **Types non pris en charge** : Journalisés (par exemple, messages d'utilisateurs protégés).

## Capacités

| Fonctionnalité | Statut |
| --- | --- |
| Messages privés | ✅ Pris en charge |
| Groupes | ⚠️ Pris en charge avec contrôles de politique (liste autorisée par défaut) |
| Médias (images) | ✅ Pris en charge |
| Réactions | ❌ Non pris en charge |
| Fils de discussion | ❌ Non pris en charge |
| Sondages | ❌ Non pris en charge |
| Commandes natives | ❌ Non pris en charge |
| Streaming | ⚠️ Bloqué (limite de 2000 caractères) |

## Cibles de livraison (CLI/cron)

-   Utilisez un identifiant de chat comme cible.
-   Exemple : `openclaw message send --channel zalo --target 123456789 --message "hi"`.

## Dépannage

**Le bot ne répond pas :**

-   Vérifiez que le jeton est valide : `openclaw channels status --probe`
-   Vérifiez que l'expéditeur est approuvé (appairage ou allowFrom)
-   Consultez les journaux de la passerelle : `openclaw logs --follow`

**Le webhook ne reçoit pas d'événements :**

-   Assurez-vous que l'URL du webhook utilise HTTPS
-   Vérifiez que le jeton secret comporte entre 8 et 256 caractères
-   Confirmez que le point de terminaison HTTP de la passerelle est accessible sur le chemin configuré
-   Vérifiez que le polling getUpdates n'est pas en cours d'exécution (ils sont mutuellement exclusifs)

## Référence de configuration (Zalo)

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.zalo.enabled` : activer/désactiver le démarrage du canal.
-   `channels.zalo.botToken` : jeton du bot provenant de la Plateforme Bot Zalo.
-   `channels.zalo.tokenFile` : lire le jeton depuis un chemin de fichier.
-   `channels.zalo.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
-   `channels.zalo.allowFrom` : liste autorisée pour les messages privés (identifiants utilisateur). `open` nécessite `"*"`. L'assistant demandera des identifiants numériques.
-   `channels.zalo.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
-   `channels.zalo.groupAllowFrom` : liste autorisée des expéditeurs de groupe (identifiants utilisateur). Utilise `allowFrom` si non défini.
-   `channels.zalo.mediaMaxMb` : limite de médias entrants/sortants (Mo, par défaut 5).
-   `channels.zalo.webhookUrl` : activer le mode webhook (HTTPS requis).
-   `channels.zalo.webhookSecret` : secret du webhook (8-256 caractères).
-   `channels.zalo.webhookPath` : chemin du webhook sur le serveur HTTP de la passerelle.
-   `channels.zalo.proxy` : URL du proxy pour les requêtes API.

Options multi-comptes :

-   `channels.zalo.accounts..botToken` : jeton par compte.
-   `channels.zalo.accounts..tokenFile` : fichier de jeton par compte.
-   `channels.zalo.accounts..name` : nom d'affichage.
-   `channels.zalo.accounts..enabled` : activer/désactiver le compte.
-   `channels.zalo.accounts..dmPolicy` : politique de messages privés par compte.
-   `channels.zalo.accounts..allowFrom` : liste autorisée par compte.
-   `channels.zalo.accounts..groupPolicy` : politique de groupe par compte.
-   `channels.zalo.accounts..groupAllowFrom` : liste autorisée des expéditeurs de groupe par compte.
-   `channels.zalo.accounts..webhookUrl` : URL webhook par compte.
-   `channels.zalo.accounts..webhookSecret` : secret webhook par compte.
-   `channels.zalo.accounts..webhookPath` : chemin webhook par compte.
-   `channels.zalo.accounts..proxy` : URL du proxy par compte.

[WhatsApp](./whatsapp.md)[Zalo Personal](./zalouser.md)

---