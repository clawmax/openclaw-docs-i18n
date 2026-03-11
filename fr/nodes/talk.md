

  Médias et périphériques

  
# Mode Conversation

Le mode Conversation est une boucle de conversation vocale continue :

1.  Écoute de la parole
2.  Envoi de la transcription au modèle (session principale, chat.send)
3.  Attente de la réponse
4.  Lecture vocale via ElevenLabs (lecture en flux)

## Comportement (macOS)

-   **Superposition toujours visible** tant que le mode Conversation est activé.
-   Transitions de phases : **Écoute → Réflexion → Parole**.
-   Sur une **courte pause** (fenêtre de silence), la transcription en cours est envoyée.
-   Les réponses sont **écrites dans le WebChat** (comme une saisie).
-   **Interrompre lors de la parole** (activé par défaut) : si l'utilisateur commence à parler pendant que l'assistant parle, la lecture est arrêtée et l'horodatage de l'interruption est noté pour l'invite suivante.

## Directives vocales dans les réponses

L'assistant peut préfixer sa réponse par une **seule ligne JSON** pour contrôler la voix :

```json
{ "voice": "<voice-id>", "once": true }
```

Règles :

-   Seulement la première ligne non vide.
-   Les clés inconnues sont ignorées.
-   `once: true` s'applique uniquement à la réponse en cours.
-   Sans `once`, la voix devient la nouvelle valeur par défaut pour le mode Conversation.
-   La ligne JSON est supprimée avant la lecture TTS.

Clés prises en charge :

-   `voice` / `voice_id` / `voiceId`
-   `model` / `model_id` / `modelId`
-   `speed`, `rate` (Mots par minute), `stability`, `similarity`, `style`, `speakerBoost`
-   `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
-   `once`

## Configuration (~/.openclaw/openclaw.json)

```json
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

Valeurs par défaut :

-   `interruptOnSpeech`: true
-   `voiceId`: utilise en secours `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (ou la première voix ElevenLabs si la clé API est disponible)
-   `modelId`: par défaut `eleven_v3` si non défini
-   `apiKey`: utilise en secours `ELEVENLABS_API_KEY` (ou le profil du shell de la passerelle si disponible)
-   `outputFormat`: par défaut `pcm_44100` sur macOS/iOS et `pcm_24000` sur Android (définir `mp3_*` pour forcer la diffusion MP3)

## Interface utilisateur macOS

-   Bouton de la barre des menus : **Conversation**
-   Onglet de configuration : groupe **Mode Conversation** (identifiant de voix + interrupteur d'interruption)
-   Superposition :
    -   **Écoute** : nuage pulsant avec le niveau du micro
    -   **Réflexion** : animation d'enfoncement
    -   **Parole** : cercles rayonnants
    -   Cliquer sur le nuage : arrêter la parole
    -   Cliquer sur X : quitter le mode Conversation

## Notes

-   Nécessite les autorisations Reconnaissance vocale + Microphone.
-   Utilise `chat.send` avec la clé de session `main`.
-   La synthèse vocale utilise l'API de flux ElevenLabs avec `ELEVENLABS_API_KEY` et une lecture incrémentale sur macOS/iOS/Android pour une latence réduite.
-   `stability` pour `eleven_v3` est validée à `0.0`, `0.5`, ou `1.0` ; les autres modèles acceptent `0..1`.
-   `latency_tier` est validée à `0..4` si définie.
-   Android prend en charge les formats de sortie `pcm_16000`, `pcm_22050`, `pcm_24000`, et `pcm_44100` pour une lecture en flux AudioTrack à faible latence.

[Capture Caméra](./camera.md)[Réveil Vocal](./voicewake.md)