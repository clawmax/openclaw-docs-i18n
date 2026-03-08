

  Protocoles et APIs

  
# Protocole de la Passerelle

Le protocole WS de la passerelle est le **plan de contrôle unique + transport des nœuds** pour OpenClaw. Tous les clients (CLI, interface web, application macOS, nœuds iOS/Android, nœuds headless) se connectent via WebSocket et déclarent leur **rôle** + **portée** au moment du handshake.

## Transport

-   WebSocket, trames texte avec des charges utiles JSON.
-   La première trame **doit** être une requête `connect`.

## Handshake (connect)

Passerelle → Client (défi pré-connexion) :

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Client → Passerelle :

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Passerelle → Client :

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Lorsqu'un jeton d'appareil est émis, `hello-ok` inclut également :

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### Exemple de nœud

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## Formatage

-   **Requête** : `{type:"req", id, method, params}`
-   **Réponse** : `{type:"res", id, ok, payload|error}`
-   **Événement** : `{type:"event", event, payload, seq?, stateVersion?}`

Les méthodes ayant des effets de bord nécessitent des **clés d'idempotence** (voir le schéma).

## Rôles + portées

### Rôles

-   `operator` = client du plan de contrôle (CLI/UI/automatisation).
-   `node` = hôte de capacités (caméra/écran/canvas/system.run).

### Portées (opérateur)

Portées courantes :

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

La portée de la méthode n'est que le premier contrôle. Certaines commandes slash atteintes via `chat.send` appliquent des vérifications plus strictes au niveau de la commande en plus. Par exemple, les écritures persistantes `/config set` et `/config unset` nécessitent `operator.admin`.

### Caps/commands/permissions (nœud)

Les nœuds déclarent leurs revendications de capacités au moment de la connexion :

-   `caps` : catégories de capacités de haut niveau.
-   `commands` : liste blanche de commandes pour l'invocation.
-   `permissions` : paramètres granulaires (par ex. `screen.record`, `camera.capture`).

La passerelle traite ces éléments comme des **revendications** et applique des listes blanches côté serveur.

## Présence

-   `system-presence` retourne des entrées indexées par identité d'appareil.
-   Les entrées de présence incluent `deviceId`, `roles` et `scopes` afin que les interfaces utilisateur puissent afficher une seule ligne par appareil même lorsqu'il se connecte à la fois en tant qu'**opérateur** et **nœud**.

### Méthodes d'aide pour les nœuds

-   Les nœuds peuvent appeler `skills.bins` pour récupérer la liste actuelle des exécutables de compétences pour les vérifications d'auto-autorisation.

### Méthodes d'aide pour les opérateurs

-   Les opérateurs peuvent appeler `tools.catalog` (`operator.read`) pour récupérer le catalogue d'outils d'exécution pour un agent. La réponse inclut les outils groupés et les métadonnées de provenance :
    -   `source` : `core` ou `plugin`
    -   `pluginId` : propriétaire du plugin lorsque `source="plugin"`
    -   `optional` : indique si un outil de plugin est optionnel

## Approbations d'exécution

-   Lorsqu'une requête d'exécution nécessite une approbation, la passerelle diffuse `exec.approval.requested`.
-   Les clients opérateurs résolvent en appelant `exec.approval.resolve` (nécessite la portée `operator.approvals`).
-   Pour `host=node`, `exec.approval.request` doit inclure `systemRunPlan` (`argv`/`cwd`/`rawCommand`/métadonnées de session canoniques). Les requêtes sans `systemRunPlan` sont rejetées.

## Gestion de version

-   `PROTOCOL_VERSION` se trouve dans `src/gateway/protocol/schema.ts`.
-   Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette les incompatibilités.
-   Les schémas + modèles sont générés à partir des définitions TypeBox :
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## Authentification

-   Si `OPENCLAW_GATEWAY_TOKEN` (ou `--token`) est défini, `connect.params.auth.token` doit correspondre ou la socket est fermée.
-   Après l'appairage, la passerelle émet un **jeton d'appareil** limité au rôle + portées de la connexion. Il est retourné dans `hello-ok.auth.deviceToken` et doit être conservé par le client pour les connexions futures.
-   Les jetons d'appareil peuvent être renouvelés/révoqués via `device.token.rotate` et `device.token.revoke` (nécessite la portée `operator.pairing`).

## Identité de l'appareil + appairage

-   Les nœuds doivent inclure une identité d'appareil stable (`device.id`) dérivée d'une empreinte de paire de clés.
-   Les passerelles émettent des jetons par appareil + rôle.
-   Les approbations d'appairage sont requises pour les nouveaux identifiants d'appareil, sauf si l'auto-approbation locale est activée.
-   Les connexions **locales** incluent la boucle locale et l'adresse tailnet de l'hôte de la passerelle elle-même (pour que les liaisons tailnet sur le même hôte puissent toujours s'auto-approuver).
-   Tous les clients WS doivent inclure l'identité `device` pendant `connect` (opérateur + nœud). L'interface de contrôle peut l'omettre **uniquement** lorsque `gateway.controlUi.dangerouslyDisableDeviceAuth` est activé pour un usage de secours.
-   Toutes les connexions doivent signer le nonce `connect.challenge` fourni par le serveur.

### Diagnostics de migration de l'authentification d'appareil

Pour les clients hérités qui utilisent encore le comportement de signature pré-défi, `connect` retourne désormais des codes de détail `DEVICE_AUTH_*` sous `error.details.code` avec un `error.details.reason` stable. Échecs de migration courants :

| Message | details.code | details.reason | Signification |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | Le client a omis `device.nonce` (ou a envoyé une valeur vide). |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | Le client a signé avec un nonce obsolète/incorrect. |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | La charge utile de la signature ne correspond pas à la charge utile v2. |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | L'horodatage signé est en dehors de la tolérance autorisée. |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id` ne correspond pas à l'empreinte de la clé publique. |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | Échec du format/de la canonicalisation de la clé publique. |

Cible de migration :

-   Toujours attendre `connect.challenge`.
-   Signer la charge utile v2 qui inclut le nonce du serveur.
-   Envoyer le même nonce dans `connect.params.device.nonce`.
-   La charge utile de signature préférée est `v3`, qui lie `platform` et `deviceFamily` en plus des champs appareil/client/rôle/portées/jeton/nonce.
-   Les signatures héritées `v2` restent acceptées pour la compatibilité, mais l'épinglage des métadonnées de l'appareil appairé contrôle toujours la politique de commande lors de la reconnexion.

## TLS + épinglage

-   TLS est pris en charge pour les connexions WS.
-   Les clients peuvent optionnellement épingler l'empreinte du certificat de la passerelle (voir la configuration `gateway.tls` ainsi que `gateway.remote.tlsFingerprint` ou l'option CLI `--tls-fingerprint`).

## Portée

Ce protocole expose **l'API complète de la passerelle** (statut, canaux, modèles, chat, agent, sessions, nœuds, approbations, etc.). La surface exacte est définie par les schémas TypeBox dans `src/gateway/protocol/schema.ts`.

[Sandbox vs Politique d'outils vs Élevé](./sandbox-vs-tool-policy-vs-elevated.md)[Protocole de Pont](./bridge-protocol.md)