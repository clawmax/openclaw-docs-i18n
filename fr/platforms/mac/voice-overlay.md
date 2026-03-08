title: "Intégration de la superposition vocale pour les contributeurs de l'application compagnon macOS"
description: "Apprenez à gérer la superposition vocale lorsque la commande vocale et la pression-pour-parler se chevauchent, y compris la gestion des sessions, le débogage et les étapes de mise en œuvre."
keywords: ["superposition vocale macos", "commande vocale", "pression-pour-parler", "session vocale", "voicewake", "intégration superposition", "application compagnon macos", "coordinateur de session vocale"]
---

  Application compagnon macOS

  
# Superposition vocale

Public : contributeurs de l'application macOS. Objectif : maintenir la superposition vocale prévisible lorsque la commande vocale et la pression-pour-parler se chevauchent.

## Intention actuelle

-   Si la superposition est déjà visible suite à une commande vocale et que l'utilisateur appuie sur la touche de raccourci, la session de la touche de raccourci *adopte* le texte existant au lieu de le réinitialiser. La superposition reste affichée tant que la touche est maintenue. Lorsque l'utilisateur relâche : envoyer s'il y a du texte tronqué, sinon fermer.
-   La commande vocale seule envoie toujours automatiquement lors du silence ; la pression-pour-parler envoie immédiatement au relâchement.

## Implémenté (9 déc. 2025)

-   Les sessions de superposition portent désormais un jeton par capture (commande vocale ou pression-pour-parler). Les mises à jour partielles/finales/envoi/fermeture/niveau sont ignorées lorsque le jeton ne correspond pas, évitant ainsi les rappels obsolètes.
-   La pression-pour-parler adopte tout texte visible de la superposition comme préfixe (ainsi, appuyer sur la touche de raccourci alors que la superposition de commande vocale est active conserve le texte et y ajoute la nouvelle parole). Elle attend jusqu'à 1,5 s pour une transcription finale avant de revenir au texte actuel.
-   La journalisation des sons/superposition est émise au niveau `info` dans les catégories `voicewake.overlay`, `voicewake.ptt` et `voicewake.chime` (début de session, partiel, final, envoi, fermeture, raison du son).

## Prochaines étapes

1.  **VoiceSessionCoordinator (acteur)**
    -   Possède exactement une `VoiceSession` à la fois.
    -   API (basée sur jeton) : `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
    -   Ignore les rappels portant des jetons obsolètes (empêche les anciens reconnaisseurs de rouvrir la superposition).
2.  **VoiceSession (modèle)**
    -   Champs : `token`, `source` (wakeWord|pushToTalk), texte validé/volatil, indicateurs de son, minuteries (envoi automatique, inactivité), `overlayMode` (affichage|édition|envoi), échéance de temporisation.
3.  **Liaison de superposition**
    -   `VoiceSessionPublisher` (`ObservableObject`) reflète la session active dans SwiftUI.
    -   `VoiceWakeOverlayView` s'affiche uniquement via l'éditeur ; il ne modifie jamais directement les singletons globaux.
    -   Les actions utilisateur sur la superposition (`sendNow`, `dismiss`, `edit`) rappellent le coordinateur avec le jeton de session.
4.  **Chemin d'envoi unifié**
    -   Sur `endCapture` : si le texte tronqué est vide → fermer ; sinon `performSend(session:)` (joue le son d'envoi une fois, transmet, ferme).
    -   Pression-pour-parler : aucun délai ; commande vocale : délai optionnel pour l'envoi automatique.
    -   Appliquer une courte temporisation au runtime de commande vocale après la fin de la pression-pour-parler pour que la commande vocale ne se redéclenche pas immédiatement.
5.  **Journalisation**
    -   Le coordinateur émet des logs `.info` dans le sous-système `ai.openclaw`, catégories `voicewake.overlay` et `voicewake.chime`.
    -   Événements clés : `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

## Liste de vérification pour le débogage

-   Diffuser les logs tout en reproduisant une superposition persistante :
    
    Copier
    
    ```bash
    sudo log stream --predicate 'subsystem == "ai.openclaw" AND category CONTAINS "voicewake"' --level info --style compact
    ```
    
-   Vérifier qu'un seul jeton de session est actif ; les rappels obsolètes doivent être ignorés par le coordinateur.
-   S'assurer que le relâchement de la pression-pour-parler appelle toujours `endCapture` avec le jeton actif ; si le texte est vide, s'attendre à une `dismiss` sans son ni envoi.

## Étapes de migration (suggérées)

1.  Ajouter `VoiceSessionCoordinator`, `VoiceSession` et `VoiceSessionPublisher`.
2.  Refactoriser `VoiceWakeRuntime` pour créer/mettre à jour/terminer des sessions au lieu de manipuler directement `VoiceWakeOverlayController`.
3.  Refactoriser `VoicePushToTalk` pour adopter les sessions existantes et appeler `endCapture` au relâchement ; appliquer la temporisation du runtime.
4.  Connecter `VoiceWakeOverlayController` à l'éditeur ; supprimer les appels directs depuis runtime/PTT.
5.  Ajouter des tests d'intégration pour l'adoption de session, la temporisation et la fermeture sans texte.

[Voice Wake](./voicewake.md)[WebChat](./webchat.md)

---