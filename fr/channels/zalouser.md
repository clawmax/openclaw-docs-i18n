

  Plateformes de messagerie

  
# Zalo Personnel

Statut : expérimental. Cette intégration automatise un **compte Zalo personnel** via `zca-js` natif à l'intérieur d'OpenClaw.

> **Avertissement :** Il s'agit d'une intégration non officielle et peut entraîner une suspension/ban du compte. Utilisez à vos propres risques.

## Plugin requis

Zalo Personnel est fourni sous forme de plugin et n'est pas inclus dans l'installation de base.

-   Installation via CLI : `openclaw plugins install @openclaw/zalouser`
-   Ou depuis un dépôt source : `openclaw plugins install ./extensions/zalouser`
-   Détails : [Plugins](../tools/plugin.md)

Aucun binaire CLI externe `zca`/`openzca` n'est requis.

## Configuration rapide (débutant)

1.  Installez le plugin (voir ci-dessus).
2.  Connectez-vous (QR, sur la machine de la Gateway) :
    -   `openclaw channels login --channel zalouser`
    -   Scannez le code QR avec l'application mobile Zalo.
3.  Activez le canal :

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4.  Redémarrez la Gateway (ou terminez l'intégration).
5.  L'accès en MP est par défaut en mode appairage ; approuvez le code d'appairage au premier contact.

## Ce que c'est

-   Fonctionne entièrement en processus via `zca-js`.
-   Utilise des écouteurs d'événements natifs pour recevoir les messages entrants.
-   Envoie les réponses directement via l'API JS (texte/média/lien).
-   Conçu pour les cas d'usage de "compte personnel" où l'API Bot Zalo n'est pas disponible.

## Nommage

L'identifiant du canal est `zalouser` pour indiquer explicitement qu'il automatise un **compte utilisateur Zalo personnel** (non officiel). Nous réservons `zalo` pour une future intégration potentielle de l'API officielle Zalo.

## Trouver les identifiants (annuaire)

Utilisez le CLI d'annuaire pour découvrir les pairs/groupes et leurs identifiants :

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## Limites

-   Le texte sortant est découpé en morceaux d'environ 2000 caractères (limites du client Zalo).
-   Le streaming est bloqué par défaut.

## Contrôle d'accès (MP)

`channels.zalouser.dmPolicy` prend en charge : `pairing | allowlist | open | disabled` (par défaut : `pairing`). `channels.zalouser.allowFrom` accepte des identifiants ou noms d'utilisateur. Pendant l'intégration, les noms sont résolus en identifiants en utilisant la recherche de contacts en processus du plugin. Approuvez via :

-   `openclaw pairing list zalouser`
-   `openclaw pairing approve zalouser `

## Accès aux groupes (optionnel)

-   Par défaut : `channels.zalouser.groupPolicy = "open"` (groupes autorisés). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu'elle n'est pas définie.
-   Restreindre à une liste autorisée avec :
    -   `channels.zalouser.groupPolicy = "allowlist"`
    -   `channels.zalouser.groups` (les clés sont des identifiants ou noms de groupe)
-   Bloquer tous les groupes : `channels.zalouser.groupPolicy = "disabled"`.
-   L'assistant de configuration peut demander les listes autorisées de groupes.
-   Au démarrage, OpenClaw résout les noms de groupes/utilisateurs dans les listes autorisées en identifiants et enregistre le mappage ; les entrées non résolues sont conservées telles qu'elles sont saisies.

Exemple :

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### Gestion des mentions dans les groupes

-   `channels.zalouser.groups..requireMention` contrôle si les réponses dans un groupe nécessitent une mention.
-   Ordre de résolution : identifiant/nom exact du groupe -> slug de groupe normalisé -> `*` -> valeur par défaut (`true`).
-   Cela s'applique à la fois aux groupes en liste autorisée et au mode groupe ouvert.

Exemple :

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## Multi-compte

Les comptes sont mappés aux profils `zalouser` dans l'état d'OpenClaw. Exemple :

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## Indicateur de saisie, réactions et accusés de réception

-   OpenClaw envoie un événement de saisie avant d'envoyer une réponse (au mieux).
-   L'action de réaction aux messages `react` est prise en charge pour `zalouser` dans les actions de canal.
    -   Utilisez `remove: true` pour supprimer un émoji de réaction spécifique d'un message.
    -   Sémantique des réactions : [Réactions](../tools/reactions.md)
-   Pour les messages entrants qui incluent des métadonnées d'événement, OpenClaw envoie des accusés de réception de livraison et de lecture (au mieux).

## Dépannage

**La connexion ne persiste pas :**

-   `openclaw channels status --probe`
-   Reconnectez-vous : `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**Le nom dans la liste autorisée/du groupe n'a pas été résolu :**

-   Utilisez des identifiants numériques dans `allowFrom`/`groups`, ou les noms exacts d'amis/groupes.

**Mis à niveau depuis une ancienne configuration basée sur CLI :**

-   Supprimez toute ancienne hypothèse de processus externe `zca`.
-   Le canal s'exécute désormais entièrement dans OpenClaw sans binaires CLI externes.

[Zalo](./zalo.md)[Appairage](./pairing.md)