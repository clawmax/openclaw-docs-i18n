

  Détails internes des concepts

  
# TypeBox

Dernière mise à jour : 2026-01-10 TypeBox est une bibliothèque de schémas axée sur TypeScript. Nous l'utilisons pour définir le **protocole WebSocket Gateway** (poignée de main, requête/réponse, événements serveur). Ces schémas pilotent la **validation à l'exécution**, l'**exportation JSON Schema** et la **génération de code Swift** pour l'application macOS. Une source unique de vérité ; tout le reste est généré. Si vous voulez le contexte de haut niveau du protocole, commencez par [l'architecture Gateway](./architecture.md).

## Modèle mental (30 secondes)

Chaque message WS Gateway est l'un des trois types de trames :

-   **Requête** : `{ type: "req", id, method, params }`
-   **Réponse** : `{ type: "res", id, ok, payload | error }`
-   **Événement** : `{ type: "event", event, payload, seq?, stateVersion? }`

La première trame **doit** être une requête `connect`. Ensuite, les clients peuvent appeler des méthodes (par ex. `health`, `send`, `chat.send`) et s'abonner à des événements (par ex. `presence`, `tick`, `agent`). Flux de connexion (minimal) :

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Méthodes + événements courants :

| Catégorie | Exemples | Notes |
| --- | --- | --- |
| Noyau | `connect`, `health`, `status` | `connect` doit être la première |
| Messagerie | `send`, `poll`, `agent`, `agent.wait` | les effets de bord nécessitent un `idempotencyKey` |
| Chat | `chat.history`, `chat.send`, `chat.abort`, `chat.inject` | utilisées par WebChat |
| Sessions | `sessions.list`, `sessions.patch`, `sessions.delete` | administration des sessions |
| Nœuds | `node.list`, `node.invoke`, `node.pair.*` | actions Gateway WS + nœud |
| Événements | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push serveur |

La liste faisant autorité se trouve dans `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## Où se trouvent les schémas

-   Source : `src/gateway/protocol/schema.ts`
-   Validateurs à l'exécution (AJV) : `src/gateway/protocol/index.ts`
-   Poignée de main serveur + dispatch des méthodes : `src/gateway/server.ts`
-   Client Node : `src/gateway/client.ts`
-   JSON Schema généré : `dist/protocol.schema.json`
-   Modèles Swift générés : `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## Pipeline actuel

-   `pnpm protocol:gen`
    -   écrit le JSON Schema (draft‑07) dans `dist/protocol.schema.json`
-   `pnpm protocol:gen:swift`
    -   génère les modèles Swift pour la gateway
-   `pnpm protocol:check`
    -   exécute les deux générateurs et vérifie que la sortie est commitée

## Comment les schémas sont utilisés à l'exécution

-   **Côté serveur** : chaque trame entrante est validée avec AJV. La poignée de main n'accepte qu'une requête `connect` dont les paramètres correspondent à `ConnectParams`.
-   **Côté client** : le client JS valide les trames d'événement et de réponse avant de les utiliser.
-   **Surface des méthodes** : la Gateway annonce les `methods` et `events` pris en charge dans `hello-ok`.

## Exemples de trames

Connexion (premier message) :

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Réponse hello-ok :

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Requête + réponse :

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Événement :

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## Client minimal (Node.js)

Flux utile le plus petit : connexion + health.

```typescript
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## Exemple détaillé : ajouter une méthode de bout en bout

Exemple : ajouter une nouvelle requête `system.echo` qui renvoie `{ ok: true, text }`.

1.  **Schéma (source de vérité)**

Ajouter dans `src/gateway/protocol/schema.ts` :

```bash
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Ajouter les deux à `ProtocolSchemas` et exporter les types :

```yaml
SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```bash
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **Validation**

Dans `src/gateway/protocol/index.ts`, exporter un validateur AJV :

```bash
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **Comportement serveur**

Ajouter un gestionnaire dans `src/gateway/server-methods/system.ts` :

```bash
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

L'enregistrer dans `src/gateway/server-methods.ts` (fusionne déjà `systemHandlers`), puis ajouter `"system.echo"` à `METHODS` dans `src/gateway/server.ts`.

4.  **Régénérer**

```bash
pnpm protocol:check
```

5.  **Tests + documentation**

Ajouter un test serveur dans `src/gateway/server.*.test.ts` et noter la méthode dans la documentation.

## Comportement de la génération de code Swift

Le générateur Swift émet :

-   L'énumération `GatewayFrame` avec les cas `req`, `res`, `event` et `unknown`
-   Des structs/énumérations de payload fortement typés
-   Les valeurs `ErrorCode` et `GATEWAY_PROTOCOL_VERSION`

Les types de trames inconnus sont conservés en tant que payloads bruts pour la compatibilité ascendante.

## Gestion de version + compatibilité

-   `PROTOCOL_VERSION` se trouve dans `src/gateway/protocol/schema.ts`.
-   Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette les incompatibilités.
-   Les modèles Swift conservent les types de trames inconnus pour éviter de casser les clients plus anciens.

## Modèles et conventions de schéma

-   La plupart des objets utilisent `additionalProperties: false` pour des payloads stricts.
-   `NonEmptyString` est la valeur par défaut pour les IDs et les noms de méthodes/événements.
-   La `GatewayFrame` de plus haut niveau utilise un **discriminateur** sur `type`.
-   Les méthodes avec effets de bord nécessitent généralement un `idempotencyKey` dans les paramètres (exemple : `send`, `poll`, `agent`, `chat.send`).
-   `agent` accepte des `internalEvents` optionnels pour le contexte d'orchestration généré à l'exécution (par exemple, transfert de fin de tâche sous-agent/cron) ; traitez cela comme une surface d'API interne.

## Schéma JSON en direct

Le JSON Schema généré se trouve dans le dépôt à `dist/protocol.schema.json`. Le fichier brut publié est généralement disponible à l'adresse :

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## Lorsque vous modifiez les schémas

1.  Mettez à jour les schémas TypeBox.
2.  Exécutez `pnpm protocol:check`.
3.  Committez le schéma régénéré + les modèles Swift.

[Date et Heure](../date-time.md)[Formatage Markdown](./markdown-formatting.md)