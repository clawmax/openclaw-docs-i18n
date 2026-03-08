

  Application compagnon macOS

  
# Voice Wake

## Modes

-   **Mode mot d'éveil** (par défaut) : le module de reconnaissance vocale toujours actif attend les mots déclencheurs (`swabbleTriggerWords`). En cas de correspondance, il démarre la capture, affiche la superposition avec le texte partiel et l'envoie automatiquement après un silence.
-   **Appui pour parler (maintenir Option droite)** : maintenez la touche Option droite pour capturer immédiatement — aucun déclencheur requis. La superposition apparaît pendant le maintien ; le relâchement finalise et transfère après un court délai pour permettre de modifier le texte.

## Comportement d'exécution (mot d'éveil)

-   Le module de reconnaissance vocale réside dans `VoiceWakeRuntime`.
-   Le déclencheur ne s'active que lorsqu'il y a une **pause significative** entre le mot d'éveil et le mot suivant (~0,55s d'écart). La superposition/sonnerie peut démarrer pendant la pause, avant même le début de la commande.
-   Fenêtres de silence : 2,0s lorsque la parole est en cours, 5,0s si seul le déclencheur a été entendu.
-   Arrêt forcé : 120s pour éviter les sessions incontrôlées.
-   Anti-rebond entre les sessions : 350ms.
-   La superposition est pilotée via `VoiceWakeOverlayController` avec un code couleur validé/volatil.
-   Après l'envoi, le module de reconnaissance redémarre proprement pour écouter le prochain déclencheur.

## Invariants du cycle de vie

-   Si Voice Wake est activé et que les autorisations sont accordées, le module de reconnaissance par mot d'éveil doit être à l'écoute (sauf pendant une capture explicite en mode appui pour parler).
-   La visibilité de la superposition (y compris la fermeture manuelle via le bouton X) ne doit jamais empêcher le module de reconnaissance de reprendre.

## Mode de défaillance de superposition persistante (précédent)

Auparavant, si la superposition restait bloquée visible et que vous la fermiez manuellement, Voice Wake pouvait sembler "mort" car la tentative de redémarrage du runtime pouvait être bloquée par la visibilité de la superposition et aucun redémarrage ultérieur n'était planifié. Renforcement :

-   Le redémarrage du runtime d'éveil n'est plus bloqué par la visibilité de la superposition.
-   La fin de la fermeture de la superposition déclenche un `VoiceWakeRuntime.refresh(...)` via `VoiceSessionCoordinator`, donc la fermeture manuelle avec X reprend toujours l'écoute.

## Spécificités de l'appui pour parler

-   La détection de raccourci utilise un moniteur global `.flagsChanged` pour l'**Option droite** (`keyCode 61` + `.option`). Nous observons uniquement les événements (pas d'interception).
-   Le pipeline de capture réside dans `VoicePushToTalk` : démarre immédiatement la reconnaissance vocale, diffuse les résultats partiels vers la superposition et appelle `VoiceWakeForwarder` lors du relâchement.
-   Lorsque l'appui pour parler démarre, nous mettons en pause le runtime de mot d'éveil pour éviter des conflits de capture audio ; il redémarre automatiquement après le relâchement.
-   Autorisations : nécessite Microphone + Reconnaissance vocale ; pour voir les événements, l'approbation Accessibilité/Surveillance des entrées est requise.
-   Claviers externes : certains peuvent ne pas exposer l'Option droite comme prévu — proposez un raccourci de secours si les utilisateurs signalent des ratés.

## Paramètres visibles par l'utilisateur

-   **Bascule Voice Wake** : active le runtime de mot d'éveil.
-   **Maintenir Cmd+Fn pour parler** : active le moniteur d'appui pour parler. Désactivé sur macOS < 26.
-   Sélecteurs de langue et de micro, indicateur de niveau en direct, tableau des mots déclencheurs, testeur (local uniquement ; ne transfère pas).
-   Le sélecteur de micro conserve la dernière sélection si un périphérique est déconnecté, affiche une indication de déconnexion et bascule temporairement sur la valeur par défaut du système jusqu'à son retour.
-   **Sons** : sonneries à la détection du déclencheur et à l'envoi ; par défaut, le son système macOS "Glass". Vous pouvez choisir n'importe quel fichier chargeable par `NSSound` (par ex. MP3/WAV/AIFF) pour chaque événement ou choisir **Aucun son**.

## Comportement de transfert

-   Lorsque Voice Wake est activé, les transcriptions sont transférées vers la passerelle/agent actif (le même mode local vs distant utilisé par le reste de l'application mac).
-   Les réponses sont délivrées au **dernier fournisseur principal utilisé** (WhatsApp/Telegram/Discord/WebChat). Si la livraison échoue, l'erreur est enregistrée et l'exécution reste visible via WebChat/les journaux de session.

## Charge utile de transfert

-   `VoiceWakeForwarder.prefixedTranscript(_:)` ajoute un préfixe d'indication machine avant l'envoi. Partagé entre les chemins mot d'éveil et appui pour parler.

## Vérification rapide

-   Activez l'appui pour parler, maintenez Cmd+Fn, parlez, relâchez : la superposition doit afficher les résultats partiels puis envoyer.
-   Pendant le maintien, les oreilles de la barre des menus doivent rester agrandies (utilise `triggerVoiceEars(ttl:nil)`) ; elles redescendent après le relâchement.

[Barre des menus](./menu-bar.md)[Superposition vocale](./voice-overlay.md)

---