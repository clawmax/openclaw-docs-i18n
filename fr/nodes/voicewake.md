

  Médias et appareils

  
# Voice Wake

OpenClaw traite **les mots d'éveil comme une liste globale unique** appartenant à la **Passerelle**.

-   Il n'y a **pas de mots d'éveil personnalisés par nœud**.
-   **Toute interface utilisateur de nœud/application peut modifier** la liste ; les changements sont persistés par la Passerelle et diffusés à tous.
-   macOS et iOS conservent des boutons locaux **Voice Wake activé/désactivé** (l'expérience utilisateur et les permissions locales diffèrent).
-   Android conserve actuellement Voice Wake désactivé et utilise un flux micro manuel dans l'onglet Voix.

## Stockage (hôte de la Passerelle)

Les mots d'éveil sont stockés sur la machine de la passerelle à l'emplacement :

-   `~/.openclaw/settings/voicewake.json`

Structure :

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## Protocole

### Méthodes

-   `voicewake.get` → `{ triggers: string[] }`
-   `voicewake.set` avec les paramètres `{ triggers: string[] }` → `{ triggers: string[] }`

Notes :

-   Les déclencheurs sont normalisés (espaces supprimés, vides éliminés). Les listes vides reviennent aux valeurs par défaut.
-   Des limites sont appliquées pour la sécurité (plafonds de nombre/longueur).

### Événements

-   `voicewake.changed` avec la charge utile `{ triggers: string[] }`

Qui le reçoit :

-   Tous les clients WebSocket (application macOS, WebChat, etc.)
-   Tous les nœuds connectés (iOS/Android), et également lors de la connexion d'un nœud sous forme de poussée initiale de "l'état actuel".

## Comportement des clients

### Application macOS

-   Utilise la liste globale pour conditionner les déclencheurs de `VoiceWakeRuntime`.
-   La modification des "Mots déclencheurs" dans les paramètres Voice Wake appelle `voicewake.set` puis s'appuie sur la diffusion pour maintenir les autres clients synchronisés.

### Nœud iOS

-   Utilise la liste globale pour la détection des déclencheurs par `VoiceWakeManager`.
-   La modification des Mots d'éveil dans les Paramètres appelle `voicewake.set` (via le WS de la Passerelle) et maintient également la détection locale des mots d'éveil réactive.

### Nœud Android

-   Voice Wake est actuellement désactivé dans l'environnement d'exécution/les Paramètres Android.
-   La voix sur Android utilise une capture micro manuelle dans l'onglet Voix au lieu des déclencheurs par mot d'éveil.

[Mode Discussion](./talk.md)[Commande de localisation](./location-command.md)

---