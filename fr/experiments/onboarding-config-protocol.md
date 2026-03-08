title: "Protocole d'Intégration et de Configuration pour les Expériences (CLI, macOS et Interface Web)"
description: "Découvrez le protocole partagé d'intégration et de configuration pour les expériences OpenClaw AI sur CLI, application macOS et interface Web, incluant le flux de l'assistant et les points de terminaison RPC."
keywords: ["protocole d'intégration", "schéma de configuration", "moteur d'assistant", "passerelle rpc", "indications ui", "expériences openclaw", "gestion de configuration", "schéma json"]
---

  Expériences

  
# Protocole d'Intégration et de Configuration

Objectif : surfaces partagées d'intégration et de configuration pour la CLI, l'application macOS et l'interface Web.

## Composants

-   Moteur d'assistant (session partagée + invites + état d'intégration).
-   L'intégration CLI utilise le même flux d'assistant que les clients UI.
-   La passerelle RPC expose les points de terminaison de l'assistant et du schéma de configuration.
-   L'intégration macOS utilise le modèle d'étapes de l'assistant.
-   L'interface Web rend les formulaires de configuration à partir du JSON Schema + des indications UI.

## Passerelle RPC

-   `wizard.start` paramètres : `{ mode?: "local"|"remote", workspace?: string }`
-   `wizard.next` paramètres : `{ sessionId, answer?: { stepId, value? } }`
-   `wizard.cancel` paramètres : `{ sessionId }`
-   `wizard.status` paramètres : `{ sessionId }`
-   `config.schema` paramètres : `{}`
-   `config.schema.lookup` paramètres : `{ path }`
    -   `path` accepte les segments de configuration standard ainsi que les identifiants de plugin délimités par des barres obliques, par exemple `plugins.entries.pack/one.config`.

Réponses (format)

-   Assistant : `{ sessionId, done, step?, status?, error? }`
-   Schéma de configuration : `{ schema, uiHints, version, generatedAt }`
-   Recherche de schéma de configuration : `{ path, schema, hint?, hintPath?, children[] }`

## Indications UI

-   `uiHints` indexé par chemin ; métadonnées optionnelles (label/aide/groupe/ordre/avancé/sensible/placeholder).
-   Les champs sensibles s'affichent comme des champs de mot de passe ; pas de couche de rédaction.
-   Les nœuds de schéma non pris en charge reviennent à l'éditeur JSON brut.

## Notes

-   Ce document est l'unique référence pour suivre les refontes du protocole pour l'intégration/configuration.

[Intégration de la passerelle Kilo](../design/kilo-gateway-integration.md)[Agents liés au fil ACP](./plans/acp-thread-bound-agents.md)

---