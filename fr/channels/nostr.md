

  Plateformes de messagerie

  
# Nostr

**Statut :** Plugin optionnel (désactivé par défaut). Nostr est un protocole décentralisé pour les réseaux sociaux. Ce canal permet à OpenClaw de recevoir et de répondre aux messages directs chiffrés (DM) via NIP-04.

## Installation (à la demande)

### Intégration (recommandé)

-   L'assistant d'intégration (`openclaw onboard`) et `openclaw channels add` listent les plugins de canal optionnels.
-   La sélection de Nostr vous invite à installer le plugin à la demande.

Installation par défaut :

-   **Canal de développement + dépôt git disponible :** utilise le chemin local du plugin.
-   **Stable/Bêta :** télécharge depuis npm.

Vous pouvez toujours remplacer le choix dans l'invite.

### Installation manuelle

```bash
openclaw plugins install @openclaw/nostr
```

Utiliser un dépôt local (workflows de développement) :

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Redémarrez la passerelle après l'installation ou l'activation des plugins.

## Configuration rapide

1.  Générez une paire de clés Nostr (si nécessaire) :

```bash
# En utilisant nak
nak key generate
```

2.  Ajoutez à la configuration :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3.  Exportez la clé :

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4.  Redémarrez la passerelle.

## Référence de configuration

| Clé | Type | Par défaut | Description |
| --- | --- | --- | --- |
| `privateKey` | string | requis | Clé privée au format `nsec` ou hexadécimal |
| `relays` | string\[\] | `['wss://relay.damus.io', 'wss://nos.lol']` | URLs des relais (WebSocket) |
| `dmPolicy` | string | `pairing` | Politique d'accès aux DM |
| `allowFrom` | string\[\] | `[]` | Clés publiques des expéditeurs autorisés |
| `enabled` | boolean | `true` | Activer/désactiver le canal |
| `name` | string | \- | Nom d'affichage |
| `profile` | object | \- | Métadonnées de profil NIP-01 |

## Métadonnées du profil

Les données du profil sont publiées en tant qu'événement NIP-01 `kind:0`. Vous pouvez les gérer depuis l'interface de contrôle (Canaux -> Nostr -> Profil) ou les définir directement dans la configuration. Exemple :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Assistant personnel bot DM",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notes :

-   Les URLs du profil doivent utiliser `https://`.
-   L'importation depuis les relais fusionne les champs et préserve les remplacements locaux.

## Contrôle d'accès

### Politiques de DM

-   **pairing** (par défaut) : les expéditeurs inconnus reçoivent un code d'appairage.
-   **allowlist** : seules les clés publiques dans `allowFrom` peuvent envoyer des DM.
-   **open** : DM entrants publics (nécessite `allowFrom: ["*"]`).
-   **disabled** : ignorer les DM entrants.

### Exemple de liste d'autorisation

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## Formats de clés

Formats acceptés :

-   **Clé privée :** `nsec...` ou hexadécimal de 64 caractères
-   **Clés publiques (`allowFrom`) :** `npub...` ou hexadécimal

## Relais

Par défaut : `relay.damus.io` et `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

Conseils :

-   Utilisez 2-3 relais pour la redondance.
-   Évitez trop de relais (latence, duplication).
-   Les relais payants peuvent améliorer la fiabilité.
-   Les relais locaux conviennent pour les tests (`ws://localhost:7777`).

## Support des protocoles

| NIP | Statut | Description |
| --- | --- | --- |
| NIP-01 | Supporté | Format d'événement de base + métadonnées de profil |
| NIP-04 | Supporté | DM chiffrés (`kind:4`) |
| NIP-17 | Prévu | DM emballés-cadeaux |
| NIP-44 | Prévu | Chiffrement versionné |

## Tests

### Relais local

```bash
# Démarrer strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### Test manuel

1.  Notez la clé publique du bot (npub) dans les journaux.
2.  Ouvrez un client Nostr (Damus, Amethyst, etc.).
3.  Envoyez un DM à la clé publique du bot.
4.  Vérifiez la réponse.

## Dépannage

### Pas de réception de messages

-   Vérifiez que la clé privée est valide.
-   Assurez-vous que les URLs des relais sont accessibles et utilisent `wss://` (ou `ws://` pour local).
-   Confirmez que `enabled` n'est pas `false`.
-   Vérifiez les journaux de la passerelle pour les erreurs de connexion aux relais.

### Pas d'envoi de réponses

-   Vérifiez que le relais accepte les écritures.
-   Vérifiez la connectivité sortante.
-   Surveillez les limites de débit des relais.

### Réponses en double

-   Attendu lors de l'utilisation de plusieurs relais.
-   Les messages sont dédupliqués par ID d'événement ; seule la première livraison déclenche une réponse.

## Sécurité

-   Ne commettez jamais de clés privées.
-   Utilisez des variables d'environnement pour les clés.
-   Envisagez `allowlist` pour les bots en production.

## Limitations (MVP)

-   Messages directs uniquement (pas de discussions de groupe).
-   Pas de pièces jointes multimédias.
-   NIP-04 uniquement (NIP-17 gift-wrap prévu).

[Nextcloud Talk](./nextcloud-talk.md)[Signal](./signal.md)