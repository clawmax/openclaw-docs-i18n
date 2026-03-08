

  Médias et appareils

  
# Support des images et médias

Le canal WhatsApp fonctionne via **Baileys Web**. Ce document décrit les règles actuelles de gestion des médias pour l'envoi, la passerelle et les réponses des agents.

## Objectifs

-   Envoyer des médias avec des légendes optionnelles via `openclaw message send --media`.
-   Permettre aux réponses automatiques de la boîte de réception web d'inclure des médias avec du texte.
-   Maintenir des limites par type raisonnables et prévisibles.

## Interface CLI

-   `openclaw message send --media <path-or-url> [--message ]`
    -   `--media` optionnel ; la légende peut être vide pour les envois de médias uniquement.
    -   `--dry-run` affiche la charge utile résolue ; `--json` émet `{ channel, to, messageId, mediaUrl, caption }`.

## Comportement du canal WhatsApp Web

-   Entrée : chemin de fichier local **ou** URL HTTP(S).
-   Flux : charger dans un Buffer, détecter le type de média, et construire la charge utile correcte :
    -   **Images :** redimensionner et recompresser en JPEG (côté max 2048px) ciblant `agents.defaults.mediaMaxMb` (par défaut 5 Mo), plafonné à 6 Mo.
    -   **Audio/Voix/Vidéo :** transmis tel quel jusqu'à 16 Mo ; l'audio est envoyé comme un message vocal (`ptt: true`).
    -   **Documents :** tout le reste, jusqu'à 100 Mo, avec le nom de fichier préservé si disponible.
-   Lecture de type GIF WhatsApp : envoyer un MP4 avec `gifPlayback: true` (CLI : `--gif-playback`) pour que les clients mobiles le lisent en boucle.
-   La détection MIME privilégie les octets magiques, puis les en-têtes, puis l'extension de fichier.
-   La légende provient de `--message` ou `reply.text` ; une légende vide est autorisée.
-   Journalisation : non-verbeux montre `↩️`/`✅` ; verbeux inclut la taille et le chemin/URL source.

## Pipeline de Réponse Automatique

-   `getReplyFromConfig` retourne `{ text?, mediaUrl?, mediaUrls? }`.
-   Lorsqu'un média est présent, l'expéditeur web résout les chemins locaux ou les URL en utilisant le même pipeline que `openclaw message send`.
-   Les entrées multimédias multiples sont envoyées séquentiellement si fournies.

## Médias entrants vers Commandes (Pi)

-   Lorsque les messages web entrants incluent des médias, OpenClaw les télécharge dans un fichier temporaire et expose des variables de modèle :
    -   `{{MediaUrl}}` pseudo-URL pour le média entrant.
    -   `{{MediaPath}}` chemin local temporaire écrit avant l'exécution de la commande.
-   Lorsqu'un bac à sable Docker par session est activé, le média entrant est copié dans l'espace de travail du bac à sable et `MediaPath`/`MediaUrl` sont réécrits en un chemin relatif comme `media/inbound/`.
-   La compréhension des médias (si configurée via `tools.media.*` ou `tools.media.models` partagés) s'exécute avant le modèle et peut insérer des blocs `[Image]`, `[Audio]`, et `[Video]` dans `Body`.
    -   L'audio définit `{{Transcript}}` et utilise la transcription pour l'analyse des commandes afin que les commandes slash fonctionnent toujours.
    -   Les descriptions de vidéo et d'image préservent tout texte de légende pour l'analyse des commandes.
-   Par défaut, seule la première pièce jointe image/audio/vidéo correspondante est traitée ; définissez `tools.media..attachments` pour traiter plusieurs pièces jointes.

## Limites et Erreurs

**Limites d'envoi sortant (envoi WhatsApp web)**

-   Images : plafond d'environ 6 Mo après recompression.
-   Audio/voix/vidéo : plafond de 16 Mo ; documents : plafond de 100 Mo.
-   Média trop volumineux ou illisible → erreur claire dans les journaux et la réponse est ignorée.

**Limites de compréhension des médias (transcription/description)**

-   Image par défaut : 10 Mo (`tools.media.image.maxBytes`).
-   Audio par défaut : 20 Mo (`tools.media.audio.maxBytes`).
-   Vidéo par défaut : 50 Mo (`tools.media.video.maxBytes`).
-   Un média trop volumineux ignore la compréhension, mais les réponses passent toujours avec le corps original.

## Notes pour les Tests

-   Couvrir les flux d'envoi + réponse pour les cas image/audio/document.
-   Valider la recompression pour les images (limite de taille) et le drapeau de message vocal pour l'audio.
-   S'assurer que les réponses multimédias multiples sont envoyées séquentiellement.

[Compréhension des Médias](./media-understanding.md)[Audio et Messages Vocaux](./audio.md)

---