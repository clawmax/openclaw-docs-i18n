title: "Capture par caméra pour les nœuds iOS Android macOS dans OpenClaw"
description: "Apprenez à capturer des photos et des vidéos en utilisant les nœuds caméra dans OpenClaw. Configurez les paramètres, utilisez les commandes CLI et intégrez la capture caméra dans les flux de travail des agents."
keywords: ["capture caméra", "nœuds openclaw", "caméra ios", "caméra android", "caméra macos", "node.invoke", "assistant cli", "capture média"]
---

  Médias et appareils

  
# Capture par caméra

OpenClaw prend en charge la **capture par caméra** pour les flux de travail des agents :

-   **Nœud iOS** (appairé via Gateway) : capture une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.
-   **Nœud Android** (appairé via Gateway) : capture une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.
-   **Application macOS** (nœud via Gateway) : capture une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.

Tous les accès à la caméra sont contrôlés par des **paramètres gérés par l'utilisateur**.

## Nœud iOS

### Paramètre utilisateur (activé par défaut)

-   Onglet Paramètres iOS → **Caméra** → **Autoriser la caméra** (`camera.enabled`)
    -   Par défaut : **activé** (l'absence de clé est traitée comme activée).
    -   Lorsque désactivé : les commandes `camera.*` renvoient `CAMERA_DISABLED`.

### Commandes (via node.invoke de Gateway)

-   `camera.list`
    -   Charge utile de réponse :
        -   `devices` : tableau de `{ id, name, position, deviceType }`
-   `camera.snap`
    -   Paramètres :
        -   `facing` : `front|back` (par défaut : `front`)
        -   `maxWidth` : nombre (optionnel ; par défaut `1600` sur le nœud iOS)
        -   `quality` : `0..1` (optionnel ; par défaut `0.9`)
        -   `format` : actuellement `jpg`
        -   `delayMs` : nombre (optionnel ; par défaut `0`)
        -   `deviceId` : chaîne (optionnel ; provenant de `camera.list`)
    -   Charge utile de réponse :
        -   `format: "jpg"`
        -   `base64: "<...>"`
        -   `width`, `height`
    -   Garde-fou de charge utile : les photos sont recompressées pour maintenir la charge utile base64 sous 5 Mo.
-   `camera.clip`
    -   Paramètres :
        -   `facing` : `front|back` (par défaut : `front`)
        -   `durationMs` : nombre (par défaut `3000`, limité à un maximum de `60000`)
        -   `includeAudio` : booléen (par défaut `true`)
        -   `format` : actuellement `mp4`
        -   `deviceId` : chaîne (optionnel ; provenant de `camera.list`)
    -   Charge utile de réponse :
        -   `format: "mp4"`
        -   `base64: "<...>"`
        -   `durationMs`
        -   `hasAudio`

### Exigence de premier plan

Comme `canvas.*`, le nœud iOS n'autorise les commandes `camera.*` qu'en **premier plan**. Les invocations en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`.

### Assistant CLI (fichiers temporaires + MEDIA)

Le moyen le plus simple d'obtenir des pièces jointes est via l'assistant CLI, qui écrit les médias décodés dans un fichier temporaire et affiche `MEDIA:`. Exemples :

```bash
openclaw nodes camera snap --node <id>               # par défaut : avant + arrière (2 lignes MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Notes :

-   `nodes camera snap` utilise par défaut **les deux** orientations pour donner à l'agent les deux vues.
-   Les fichiers de sortie sont temporaires (dans le répertoire temporaire du système d'exploitation) sauf si vous créez votre propre wrapper.

## Nœud Android

### Paramètre utilisateur Android (activé par défaut)

-   Fenêtre Paramètres Android → **Caméra** → **Autoriser la caméra** (`camera.enabled`)
    -   Par défaut : **activé** (l'absence de clé est traitée comme activée).
    -   Lorsque désactivé : les commandes `camera.*` renvoient `CAMERA_DISABLED`.

### Permissions

-   Android nécessite des permissions d'exécution :
    -   `CAMERA` pour `camera.snap` et `camera.clip`.
    -   `RECORD_AUDIO` pour `camera.clip` lorsque `includeAudio=true`.

Si les permissions sont manquantes, l'application demandera quand cela est possible ; si elles sont refusées, les requêtes `camera.*` échouent avec une erreur `*_PERMISSION_REQUIRED`.

### Exigence de premier plan Android

Comme `canvas.*`, le nœud Android n'autorise les commandes `camera.*` qu'en **premier plan**. Les invocations en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`.

### Commandes Android (via node.invoke de Gateway)

-   `camera.list`
    -   Charge utile de réponse :
        -   `devices` : tableau de `{ id, name, position, deviceType }`

### Garde-fou de charge utile

Les photos sont recompressées pour maintenir la charge utile base64 sous 5 Mo.

## Application macOS

### Paramètre utilisateur (désactivé par défaut)

L'application compagnon macOS expose une case à cocher :

-   **Paramètres → Général → Autoriser la caméra** (`openclaw.cameraEnabled`)
    -   Par défaut : **désactivé**
    -   Lorsque désactivé : les requêtes caméra renvoient "Caméra désactivée par l'utilisateur".

### Assistant CLI (node invoke)

Utilisez le CLI principal `openclaw` pour invoquer les commandes caméra sur le nœud macOS. Exemples :

```bash
openclaw nodes camera list --node <id>            # lister les identifiants de caméra
openclaw nodes camera snap --node <id>            # affiche MEDIA:<chemin>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # affiche MEDIA:<chemin>
openclaw nodes camera clip --node <id> --duration-ms 3000      # affiche MEDIA:<chemin> (option héritée)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notes :

-   `openclaw nodes camera snap` utilise par défaut `maxWidth=1600` sauf indication contraire.
-   Sur macOS, `camera.snap` attend `delayMs` (par défaut 2000ms) après la stabilisation de la mise en route/de l'exposition avant de capturer.
-   Les charges utiles des photos sont recompressées pour maintenir la base64 sous 5 Mo.

## Sécurité + limites pratiques

-   L'accès à la caméra et au microphone déclenche les invites de permission habituelles du système d'exploitation (et nécessite des chaînes d'utilisation dans Info.plist).
-   Les clips vidéo sont limités (actuellement `<= 60s`) pour éviter des charges utiles de nœud trop volumineuses (surcharge base64 + limites de messages).

## Vidéo d'écran macOS (niveau système)

Pour la vidéo d'*écran* (pas de la caméra), utilisez l'application compagnon macOS :

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # affiche MEDIA:<chemin>
```

Notes :

-   Nécessite la permission **Enregistrement d'écran** macOS (TCC).

[Audio et notes vocales](./audio.md)[Mode conversation](./talk.md)