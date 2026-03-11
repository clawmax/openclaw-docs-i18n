

  Application compagnon macOS

  
# Icône de la barre de menus

Auteur : steipete · Mis à jour : 2025-12-06 · Périmètre : application macOS (`apps/macos`)

-   **Inactif :** Animation normale de l'icône (clignotement, mouvement occasionnel).
-   **En pause :** L'élément de statut utilise `appearsDisabled` ; pas de mouvement.
-   **Déclenchement vocal (grandes oreilles) :** Le détecteur de réveil vocal appelle `AppState.triggerVoiceEars(ttl: nil)` lorsque le mot d'activation est entendu, maintenant `earBoostActive=true` pendant la capture de l'énoncé. Les oreilles augmentent d'échelle (1,9x), obtiennent des trous d'oreille circulaires pour la lisibilité, puis redescendent via `stopVoiceEars()` après 1s de silence. Seulement déclenché depuis le pipeline vocal intégré à l'application.
-   **En cours d'exécution (agent actif) :** `AppState.isWorking=true` entraîne un micro-mouvement de « remuement de queue/pattes » : mouvement plus rapide des pattes et léger décalage pendant le travail. Actuellement basculé autour des exécutions de l'agent WebChat ; ajoutez la même bascule autour d'autres tâches longues lorsque vous les connectez.

Points de connexion

-   Réveil vocal : l'exécution/le testeur appelle `AppState.triggerVoiceEars(ttl: nil)` lors du déclenchement et `stopVoiceEars()` après 1s de silence pour correspondre à la fenêtre de capture.
-   Activité de l'agent : définissez `AppStateStore.shared.setWorking(true/false)` autour des périodes de travail (déjà fait dans l'appel de l'agent WebChat). Gardez les périodes courtes et réinitialisez dans des blocs `defer` pour éviter les animations bloquées.

Formes et tailles

-   Icône de base dessinée dans `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
-   L'échelle des oreilles est par défaut à `1.0` ; le boost vocal définit `earScale=1.9` et active `earHoles=true` sans changer le cadre global (image modèle 18×18 pt rendue dans un stockage de support Retina 36×36 px).
-   Le remuement utilise un mouvement des pattes jusqu'à ~1,0 avec une petite secousse horizontale ; il s'ajoute à tout mouvement d'inactivité existant.

Notes comportementales

-   Pas de bascule CLI/broker externe pour les oreilles/l'activité ; gardez cela interne aux signaux propres de l'application pour éviter un battement accidentel.
-   Gardez les TTL courts (&lt;10s) pour que l'icône revienne rapidement à l'état de base si un travail se bloque.

[Contrôles de santé](./health.md)[Journalisation macOS](./logging.md)