

  Application compagnon macOS

  
# WebChat

L'application de barre de menus macOS intègre l'interface WebChat en tant que vue native SwiftUI. Elle se connecte à la Passerelle et utilise par défaut la **session principale** pour l'agent sélectionné (avec un sélecteur de session pour les autres sessions).

-   **Mode local** : se connecte directement au WebSocket de la Passerelle locale.
-   **Mode distant** : transfère le port de contrôle de la Passerelle via SSH et utilise ce tunnel comme plan de données.

## Lancement & débogage

-   Manuel : Menu Lobster → "Ouvrir le chat".
-   Ouverture automatique pour les tests :
    
    Copier
    
    ```bash
    dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
    ```
    
-   Journaux : `./scripts/clawlog.sh` (sous-système `ai.openclaw`, catégorie `WebChatSwiftUI`).

## Comment c'est connecté

-   Plan de données : Méthodes WS de la Passerelle `chat.history`, `chat.send`, `chat.abort`, `chat.inject` et événements `chat`, `agent`, `presence`, `tick`, `health`.
-   Session : utilise par défaut la session principale (`main`, ou `global` lorsque la portée est globale). L'interface peut basculer entre les sessions.
-   L'intégration utilise une session dédiée pour séparer la configuration du premier démarrage.

## Surface de sécurité

-   Le mode distant transfère uniquement le port de contrôle WebSocket de la Passerelle via SSH.

## Limitations connues

-   L'interface est optimisée pour les sessions de chat (ce n'est pas un bac à sable navigateur complet).

[Superposition Vocale](./voice-overlay.md)[Canvas](./canvas.md)

---