

  Outils intégrés

  
# Réactions

Sémantique des réactions partagée entre les canaux :

-   `emoji` est requis pour ajouter une réaction.
-   `emoji=""` supprime la ou les réactions du bot lorsque c'est pris en charge.
-   `remove: true` supprime l'émoji spécifié lorsque c'est pris en charge (nécessite `emoji`).

Notes par canal :

-   **Discord/Slack** : un `emoji` vide supprime toutes les réactions du bot sur le message ; `remove: true` supprime uniquement cet émoji.
-   **Google Chat** : un `emoji` vide supprime les réactions de l'application sur le message ; `remove: true` supprime uniquement cet émoji.
-   **Telegram** : un `emoji` vide supprime les réactions du bot ; `remove: true` supprime également les réactions mais nécessite toujours un `emoji` non vide pour la validation de l'outil.
-   **WhatsApp** : un `emoji` vide supprime la réaction du bot ; `remove: true` correspond à un émoji vide (nécessite toujours `emoji`).
-   **Zalo Personnel (`zalouser`)** : nécessite un `emoji` non vide ; `remove: true` supprime cette réaction par émoji spécifique.
-   **Signal** : les notifications de réaction entrantes émettent des événements système lorsque `channels.signal.reactionNotifications` est activé.

[Détection de boucle d'outils](./loop-detection.md)[Niveaux de réflexion](./thinking.md)

---