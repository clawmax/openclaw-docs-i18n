title: "Guide des nœuds OpenClaw pour périphériques multimédias et hôtes distants"
description: "Apprenez à configurer et gérer les nœuds OpenClaw pour les captures d'écran, la caméra, la localisation, les SMS et l'exécution de commandes à distance. Inclut l'appairage, les commandes CLI et le dépannage."
keywords: ["nœuds openclaw", "appairage de périphérique", "hôte de nœud distant", "instantané de toile", "commandes caméra", "nœuds android", "system.run", "protocole de passerelle"]
---

  Médias et périphériques

  
# Nœuds

Un **nœud** est un périphérique compagnon (macOS/iOS/Android/sans interface) qui se connecte au **WebSocket** de la passerelle (même port que les opérateurs) avec `role: "node"` et expose une surface de commande (par ex. `canvas.*`, `camera.*`, `device.*`, `notifications.*`, `system.*`) via `node.invoke`. Détails du protocole : [Protocole de passerelle](./gateway/protocol.md). Transport hérité : [Protocole Bridge](./gateway/bridge-protocol.md) (TCP JSONL ; obsolète/supprimé pour les nœuds actuels). macOS peut également fonctionner en **mode nœud** : l'application de la barre de menus se connecte au serveur WS de la passerelle et expose ses commandes locales de toile/caméra comme un nœud (ainsi `openclaw nodes …` fonctionne sur ce Mac). Notes :

-   Les nœuds sont des **périphériques**, pas des passerelles. Ils n'exécutent pas le service de passerelle.
-   Les messages Telegram/WhatsApp/etc. arrivent sur la **passerelle**, pas sur les nœuds.
-   Guide de dépannage : [/nodes/troubleshooting](./nodes/troubleshooting.md)

## Appairage + statut

**Les nœuds WS utilisent l'appairage de périphérique.** Les nœuds présentent une identité de périphérique lors de `connect` ; la passerelle crée une demande d'appairage de périphérique pour `role: node`. Approuvez via la CLI (ou l'interface) des périphériques. CLI rapide :

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notes :

-   `nodes status` marque un nœud comme **appairé** lorsque le rôle d'appairage de son périphérique inclut `node`.
-   `node.pair.*` (CLI : `openclaw nodes pending/approve/reject`) est un magasin d'appairage de nœud distinct appartenant à la passerelle ; il **ne** contrôle **pas** la poignée de main `connect` WS.

## Hôte de nœud distant (system.run)

Utilisez un **hôte de nœud** lorsque votre passerelle s'exécute sur une machine et que vous souhaitez que les commandes s'exécutent sur une autre. Le modèle communique toujours avec la **passerelle** ; la passerelle transmet les appels `exec` à l'**hôte de nœud** lorsque `host=node` est sélectionné.

### Ce qui s'exécute où

-   **Hôte de la passerelle** : reçoit les messages, exécute le modèle, achemine les appels d'outils.
-   **Hôte du nœud** : exécute `system.run`/`system.which` sur la machine du nœud.
-   **Approbations** : appliquées sur l'hôte du nœud via `~/.openclaw/exec-approvals.json`.

### Démarrer un hôte de nœud (premier plan)

Sur la machine du nœud :

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Passerelle distante via tunnel SSH (liaison loopback)

Si la passerelle se lie à loopback (`gateway.bind=loopback`, par défaut en mode local), les hôtes de nœuds distants ne peuvent pas se connecter directement. Créez un tunnel SSH et pointez l'hôte du nœud vers l'extrémité locale du tunnel. Exemple (hôte du nœud -> hôte de la passerelle) :

```bash
# Terminal A (garder en cours) : redirige local 18790 -> passerelle 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Terminal B : exportez le jeton de passerelle et connectez-vous via le tunnel
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

Notes :

-   Le jeton est `gateway.auth.token` de la configuration de la passerelle (`~/.openclaw/openclaw.json` sur l'hôte de la passerelle).
-   `openclaw node run` lit `OPENCLAW_GATEWAY_TOKEN` pour l'authentification.

### Démarrer un hôte de nœud (service)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### Appairer + nommer

Sur l'hôte de la passerelle :

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

Options de nommage :

-   `--display-name` sur `openclaw node run` / `openclaw node install` (persiste dans `~/.openclaw/node.json` sur le nœud).
-   `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (surcharge de la passerelle).

### Liste autorisée des commandes

Les approbations d'exécution sont **par hôte de nœud**. Ajoutez des entrées à la liste autorisée depuis la passerelle :

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Les approbations résident sur l'hôte du nœud dans `~/.openclaw/exec-approvals.json`.

### Pointer exec vers le nœud

Configurer les valeurs par défaut (configuration de la passerelle) :

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Ou par session :

```bash
/exec host=node security=allowlist node=<id-or-name>
```

Une fois configuré, tout appel `exec` avec `host=node` s'exécute sur l'hôte du nœud (sous réserve de la liste autorisée/approbations du nœud). Liens connexes :

-   [CLI de l'hôte de nœud](./cli/node.md)
-   [Outil Exec](./tools/exec.md)
-   [Approbations Exec](./tools/exec-approvals.md)

## Appel de commandes

Bas niveau (RPC brut) :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Des aides de plus haut niveau existent pour les flux de travail courants "donner à l'agent une pièce jointe MÉDIA".

## Captures d'écran (instantanés de toile)

Si le nœud affiche la Toile (WebView), `canvas.snapshot` renvoie `{ format, base64 }`. Aide CLI (écrit dans un fichier temporaire et affiche `MEDIA:`) :

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### Contrôles de la toile

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notes :

-   `canvas present` accepte des URL ou des chemins de fichiers locaux (`--target`), plus éventuellement `--x/--y/--width/--height` pour le positionnement.
-   `canvas eval` accepte du JS en ligne (`--js`) ou un argument positionnel.

### A2UI (Toile)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Notes :

-   Seul A2UI v0.8 JSONL est pris en charge (v0.9/createSurface est rejeté).

## Photos + vidéos (caméra du nœud)

Photos (`jpg`) :

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # par défaut : les deux orientations (2 lignes MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Séquences vidéo (`mp4`) :

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notes :

-   Le nœud doit être **au premier plan** pour `canvas.*` et `camera.*` (les appels en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`).
-   La durée des séquences est limitée (actuellement `<= 60s`) pour éviter des charges utiles base64 trop volumineuses.
-   Android demandera les autorisations `CAMERA`/`RECORD_AUDIO` lorsque possible ; les autorisations refusées échouent avec `*_PERMISSION_REQUIRED`.

## Enregistrements d'écran (nœuds)

Les nœuds exposent `screen.record` (mp4). Exemple :

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notes :

-   `screen.record` nécessite que l'application du nœud soit au premier plan.
-   Android affichera l'invite système d'enregistrement d'écran avant l'enregistrement.
-   Les enregistrements d'écran sont limités à `<= 60s`.
-   `--no-audio` désactive la capture du microphone (pris en charge sur iOS/Android ; macOS utilise la capture audio système).
-   Utilisez `--screen ` pour sélectionner un écran lorsque plusieurs écrans sont disponibles.

## Localisation (nœuds)

Les nœuds exposent `location.get` lorsque la Localisation est activée dans les paramètres. Aide CLI :

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Notes :

-   La localisation est **désactivée par défaut**.
-   "Toujours" nécessite une autorisation système ; la récupération en arrière-plan est au mieux.
-   La réponse inclut lat/lon, précision (mètres) et horodatage.

## SMS (nœuds Android)

Les nœuds Android peuvent exposer `sms.send` lorsque l'utilisateur accorde l'autorisation **SMS** et que l'appareil prend en charge la téléphonie. Appel bas niveau :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notes :

-   L'invite d'autorisation doit être acceptée sur l'appareil Android avant que la capacité ne soit annoncée.
-   Les appareils uniquement Wi-Fi sans téléphonie n'annonceront pas `sms.send`.

## Commandes de périphérique Android et données personnelles

Les nœuds Android peuvent annoncer des familles de commandes supplémentaires lorsque les capacités correspondantes sont activées. Familles disponibles :

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

Exemples d'appels :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

Notes :

-   Les commandes de mouvement sont conditionnées par les capteurs disponibles.
-   `app.update` est conditionné par les autorisations et la politique du runtime du nœud.

## Commandes système (hôte de nœud / nœud mac)

Le nœud macOS expose `system.run`, `system.notify` et `system.execApprovals.get/set`. L'hôte de nœud sans interface expose `system.run`, `system.which` et `system.execApprovals.get/set`. Exemples :

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notes :

-   `system.run` renvoie la sortie standard/erreur standard/code de sortie dans la charge utile.
-   `system.notify` respecte l'état de l'autorisation de notification dans l'application macOS.
-   Les métadonnées `platform` / `deviceFamily` de nœud non reconnues utilisent une liste autorisée conservatrice par défaut qui exclut `system.run` et `system.which`. Si vous avez intentionnellement besoin de ces commandes pour une plateforme inconnue, ajoutez-les explicitement via `gateway.nodes.allowCommands`.
-   `system.run` prend en charge `--cwd`, `--env KEY=VAL`, `--command-timeout` et `--needs-screen-recording`.
-   Pour les enveloppes shell (`bash|sh|zsh ... -c/-lc`), les valeurs `--env` limitées à la portée de la requête sont réduites à une liste autorisée explicite (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   Pour les décisions d'autorisation toujours dans le mode liste autorisée, les enveloppes de distribution connues (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) conservent les chemins d'exécutable internes au lieu des chemins d'enveloppe. Si le déballage n'est pas sûr, aucune entrée de liste autorisée n'est persistée automatiquement.
-   Sur les hôtes de nœuds Windows en mode liste autorisée, les exécutions via enveloppe shell `cmd.exe /c` nécessitent une approbation (l'entrée dans la liste autorisée seule n'autorise pas automatiquement la forme enveloppée).
-   `system.notify` prend en charge `--priority <passive|active|timeSensitive>` et `--delivery <system|overlay|auto>`.
-   Les hôtes de nœuds ignorent les remplacements de `PATH` et suppriment les clés de démarrage/shell dangereuses (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). Si vous avez besoin d'entrées PATH supplémentaires, configurez l'environnement du service de l'hôte de nœud (ou installez les outils dans des emplacements standard) au lieu de passer `PATH` via `--env`.
-   Sur le mode nœud macOS, `system.run` est conditionné par les approbations d'exécution dans l'application macOS (Paramètres → Approbations d'exécution). Demander/liste autorisée/complet se comportent de la même manière que l'hôte de nœud sans interface ; les invites refusées renvoient `SYSTEM_RUN_DENIED`.
-   Sur l'hôte de nœud sans interface, `system.run` est conditionné par les approbations d'exécution (`~/.openclaw/exec-approvals.json`).

## Liaison de nœud Exec

Lorsque plusieurs nœuds sont disponibles, vous pouvez lier exec à un nœud spécifique. Cela définit le nœud par défaut pour `exec host=node` (et peut être remplacé par agent). Valeur par défaut globale :

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Remplacement par agent :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Définir pour autoriser n'importe quel nœud :

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## Carte des autorisations

Les nœuds peuvent inclure une carte `permissions` dans `node.list` / `node.describe`, indexée par nom d'autorisation (par ex. `screenRecording`, `accessibility`) avec des valeurs booléennes (`true` = accordée).

## Hôte de nœud sans interface (multi-plateforme)

OpenClaw peut exécuter un **hôte de nœud sans interface** (sans UI) qui se connecte au WebSocket de la passerelle et expose `system.run` / `system.which`. Ceci est utile sur Linux/Windows ou pour exécuter un nœud minimal à côté d'un serveur. Démarrez-le :

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notes :

-   L'appairage est toujours requis (la passerelle affichera une invite d'appairage de périphérique).
-   L'hôte de nœud stocke son identifiant de nœud, son jeton, son nom d'affichage et ses informations de connexion à la passerelle dans `~/.openclaw/node.json`.
-   Les approbations d'exécution sont appliquées localement via `~/.openclaw/exec-approvals.json` (voir [Approbations Exec](./tools/exec-approvals.md)).
-   Sur macOS, l'hôte de nœud sans interface exécute `system.run` localement par défaut. Définissez `OPENCLAW_NODE_EXEC_HOST=app` pour acheminer `system.run` via l'hôte d'exécution de l'application compagnon ; ajoutez `OPENCLAW_NODE_EXEC_FALLBACK=0` pour exiger l'hôte de l'application et échouer en mode fermé s'il n'est pas disponible.
-   Ajoutez `--tls` / `--tls-fingerprint` lorsque le WS de la passerelle utilise TLS.

## Mode nœud Mac

-   L'application de la barre de menus macOS se connecte au serveur WS de la passerelle en tant que nœud (ainsi `openclaw nodes …` fonctionne sur ce Mac).
-   En mode distant, l'application ouvre un tunnel SSH pour le port de la passerelle et se connecte à `localhost`.

[Surveillance de l'authentification](./automation/auth-monitoring.md)[Dépannage des nœuds](./nodes/troubleshooting.md)