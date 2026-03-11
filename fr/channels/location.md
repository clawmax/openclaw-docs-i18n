

  Configuration

  
# Analyse de localisation des canaux

OpenClaw normalise les localisations partagées depuis les canaux de discussion en :

-   un texte lisible par un humain ajouté au corps du message entrant, et
-   des champs structurés dans le contexte de charge utile de la réponse automatique.

Actuellement pris en charge :

-   **Telegram** (épingles de localisation + lieux + localisations en direct)
-   **WhatsApp** (locationMessage + liveLocationMessage)
-   **Matrix** (`m.location` avec `geo_uri`)

## Formatage du texte

Les localisations sont affichées sous forme de lignes conviviales sans crochets :

-   Épingle :
    -   `📍 48.858844, 2.294351 ±12m`
-   Lieu nommé :
    -   `📍 Tour Eiffel — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
-   Partage en direct :
    -   `🛰 Localisation en direct : 48.858844, 2.294351 ±12m`

Si le canal inclut une légende/commentaire, il est ajouté à la ligne suivante :

```
📍 48.858844, 2.294351 ±12m
Rendez-vous ici
```

## Champs de contexte

Lorsqu'une localisation est présente, ces champs sont ajoutés à `ctx` :

-   `LocationLat` (nombre)
-   `LocationLon` (nombre)
-   `LocationAccuracy` (nombre, mètres ; optionnel)
-   `LocationName` (chaîne ; optionnel)
-   `LocationAddress` (chaîne ; optionnel)
-   `LocationSource` (`pin | place | live`)
-   `LocationIsLive` (booléen)

## Notes par canal

-   **Telegram** : les lieux sont mappés sur `LocationName/LocationAddress` ; les localisations en direct utilisent `live_period`.
-   **WhatsApp** : `locationMessage.comment` et `liveLocationMessage.caption` sont ajoutés comme ligne de légende.
-   **Matrix** : `geo_uri` est analysé comme une épingle de localisation ; l'altitude est ignorée et `LocationIsLive` est toujours faux.

[Routage des canaux](./channel-routing.md)[Dépannage des canaux](./troubleshooting.md)