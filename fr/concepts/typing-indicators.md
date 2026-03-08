title: "Configuration et modes des indicateurs de saisie OpenClaw"
description: "Apprenez à configurer les indicateurs de saisie dans OpenClaw, y compris les modes instantané, réflexion et message. Contrôlez le démarrage de la saisie et sa cadence de rafraîchissement."
keywords: ["indicateurs de saisie", "configuration openclaw", "paramètres par défaut des agents", "mode de saisie", "intervalle de saisie", "comportement du chat", "boucle du modèle", "delta de raisonnement"]
---

  Détails internes du concept

  
# Indicateurs de saisie

Les indicateurs de saisie sont envoyés au canal de discussion pendant l'exécution d'un run. Utilisez `agents.defaults.typingMode` pour contrôler **quand** la saisie commence et `typingIntervalSeconds` pour contrôler **à quelle fréquence** elle se rafraîchit.

## Valeurs par défaut

Lorsque `agents.defaults.typingMode` n'est **pas défini**, OpenClaw conserve le comportement hérité :

-   **Chats directs** : la saisie commence immédiatement dès que la boucle du modèle démarre.
-   **Chats de groupe avec une mention** : la saisie commence immédiatement.
-   **Chats de groupe sans mention** : la saisie commence uniquement lorsque le texte du message commence à être diffusé.
-   **Runs de type heartbeat** : la saisie est désactivée.

## Modes

Définissez `agents.defaults.typingMode` sur l'une des valeurs suivantes :

-   `never` — aucun indicateur de saisie, jamais.
-   `instant` — commence à saisir **dès que la boucle du modèle commence**, même si le run ne renvoie plus tard que le jeton de réponse silencieux.
-   `thinking` — commence à saisir au **premier delta de raisonnement** (nécessite `reasoningLevel: "stream"` pour le run).
-   `message` — commence à saisir au **premier delta de texte non silencieux** (ignore le jeton silencieux `NO_REPLY`).

Ordre de "précocité du déclenchement" : `never` → `message` → `thinking` → `instant`

## Configuration

```json
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

Vous pouvez remplacer le mode ou la cadence par session :

```json
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```

## Notes

-   Le mode `message` n'affichera pas la saisie pour les réponses uniquement silencieuses (par exemple, le jeton `NO_REPLY` utilisé pour supprimer la sortie).
-   `thinking` ne se déclenche que si le run diffuse du raisonnement (`reasoningLevel: "stream"`). Si le modèle n'émet pas de deltas de raisonnement, la saisie ne démarrera pas.
-   Les heartbeats n'affichent jamais la saisie, quel que soit le mode.
-   `typingIntervalSeconds` contrôle la **cadence de rafraîchissement**, pas l'heure de début. La valeur par défaut est de 6 secondes.

[Mise en forme Markdown](./markdown-formatting.md)[Suivi de l'utilisation](./usage-tracking.md)

---