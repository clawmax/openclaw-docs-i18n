title: "Adaptateurs RPC et API OpenClaw pour l'intégration de CLI externes"
description: "Découvrez comment OpenClaw intègre des CLI externes à l'aide d'adaptateurs JSON-RPC, incluant les modèles de démon HTTP et de processus enfant stdio pour des canaux comme Signal et iMessage."
keywords: ["json-rpc", "adaptateur rpc", "signal-cli", "intégration imessage", "passerelle api", "cli externe", "openclaw rpc", "processus stdio"]
---

  RPC et API

  
# Adaptateurs RPC

OpenClaw intègre des CLI externes via JSON-RPC. Deux modèles sont utilisés aujourd'hui.

## Modèle A : Démon HTTP (signal-cli)

-   `signal-cli` s'exécute comme un démon avec JSON-RPC sur HTTP.
-   Le flux d'événements est SSE (`/api/v1/events`).
-   Sonde de santé : `/api/v1/check`.
-   OpenClaw gère le cycle de vie lorsque `channels.signal.autoStart=true`.

Voir [Signal](../channels/signal.md) pour la configuration et les points de terminaison.

## Modèle B : Processus enfant stdio (hérité : imsg)

> **Note :** Pour les nouvelles configurations iMessage, utilisez [BlueBubbles](../channels/bluebubbles.md) à la place.

-   OpenClaw lance `imsg rpc` comme un processus enfant (intégration iMessage héritée).
-   JSON-RPC est délimité par ligne sur stdin/stdout (un objet JSON par ligne).
-   Aucun port TCP, aucun démon requis.

Méthodes principales utilisées :

-   `watch.subscribe` → notifications (`method: "message"`)
-   `watch.unsubscribe`
-   `send`
-   `chats.list` (sonde/diagnostics)

Voir [iMessage](../channels/imessage.md) pour la configuration héritée et l'adressage (préférence pour `chat_id`).

## Directives pour les adaptateurs

-   La passerelle possède le processus (démarrage/arrêt lié au cycle de vie du fournisseur).
-   Gardez les clients RPC résilients : délais d'attente, redémarrage en cas de sortie.
-   Préférez les identifiants stables (par ex., `chat_id`) aux chaînes d'affichage.

[webhooks](../cli/webhooks.md)[Base de données des modèles d'appareils](./device-models.md)

---