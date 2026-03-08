title: "Guide de l'application macOS OpenClaw pour le compagnon de barre de menus et la passerelle"
description: "Apprenez à utiliser l'application macOS OpenClaw dans la barre de menus pour gérer les permissions, vous connecter à la passerelle et exposer les capacités macOS comme Canvas et l'enregistrement d'écran aux agents IA."
keywords: ["openclaw macos", "application barre de menus macos", "connexion passerelle", "permissions macos tcc", "approbations system.run", "mode distant ssh", "contrôle launchd", "automatisation canvas"]
---

  Aperçu des plateformes

  
# Application macOS

L'application macOS est le **compagnon de barre de menus** pour OpenClaw. Elle possède les permissions, gère/se connecte à la passerelle localement (launchd ou manuel), et expose les capacités macOS à l'agent en tant que nœud.

## Ce qu'elle fait

-   Affiche des notifications natives et le statut dans la barre de menus.
-   Possède les invites TCC (Notifications, Accessibilité, Enregistrement d'écran, Microphone, Reconnaissance vocale, Automatisation/AppleScript).
-   Exécute ou se connecte à la passerelle (locale ou distante).
-   Expose des outils spécifiques à macOS (Canvas, Caméra, Enregistrement d'écran, `system.run`).
-   Démarre le service hôte de nœud local en mode **distant** (launchd), et l'arrête en mode **local**.
-   Héberge optionnellement **PeekabooBridge** pour l'automatisation d'interface utilisateur.
-   Installe le CLI global (`openclaw`) via npm/pnpm sur demande (bun n'est pas recommandé pour l'exécution de la passerelle).

## Mode local vs distant

-   **Local** (par défaut) : l'application se connecte à une passerelle locale en cours d'exécution si présente ; sinon, elle active le service launchd via `openclaw gateway install`.
-   **Distant** : l'application se connecte à une passerelle via SSH/Tailscale et ne démarre jamais de processus local. L'application démarre le **service hôte de nœud local** pour que la passerelle distante puisse atteindre ce Mac. L'application ne lance pas la passerelle en tant que processus enfant.

## Contrôle Launchd

L'application gère un LaunchAgent par utilisateur étiqueté `ai.openclaw.gateway` (ou `ai.openclaw.` lors de l'utilisation de `--profile`/`OPENCLAW_PROFILE` ; l'ancien `com.openclaw.*` se décharge toujours).

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Remplacez l'étiquette par `ai.openclaw.` lors de l'exécution d'un profil nommé. Si le LaunchAgent n'est pas installé, activez-le depuis l'application ou exécutez `openclaw gateway install`.

## Capacités du nœud (mac)

L'application macOS se présente comme un nœud. Commandes courantes :

-   Canvas : `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
-   Caméra : `camera.snap`, `camera.clip`
-   Écran : `screen.record`
-   Système : `system.run`, `system.notify`

Le nœud rapporte une carte `permissions` pour que les agents puissent décider de ce qui est autorisé. Service de nœud + IPC de l'application :

-   Lorsque le service hôte de nœud sans interface est en cours d'exécution (mode distant), il se connecte à la passerelle WS en tant que nœud.
-   `system.run` s'exécute dans l'application macOS (contexte UI/TCC) via une socket Unix locale ; les invites et la sortie restent dans l'application.

Diagramme (SCI) :

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

## Approbations d'exécution (system.run)

`system.run` est contrôlé par les **Approbations d'exécution** dans l'application macOS (Paramètres → Approbations d'exécution). La sécurité + demande + liste autorisée sont stockées localement sur le Mac dans :

```
~/.openclaw/exec-approvals.json
```

Exemple :

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

Notes :

-   Les entrées `allowlist` sont des motifs glob pour les chemins binaires résolus.
-   Le texte brut de commande shell contenant une syntaxe de contrôle ou d'expansion shell (`&&`, `||`, `;`, `|`, ```, `$`, `<`, `>`, `(`, `)`) est traité comme un échec de liste autorisée et nécessite une approbation explicite (ou l'ajout du binaire shell à la liste autorisée).
-   Choisir "Toujours autoriser" dans l'invite ajoute cette commande à la liste autorisée.
-   Les remplacements d'environnement `system.run` sont filtrés (supprime `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`) puis fusionnés avec l'environnement de l'application.
-   Pour les enveloppes shell (`bash|sh|zsh ... -c/-lc`), les remplacements d'environnement limités à la requête sont réduits à une petite liste autorisée explicite (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   Pour les décisions "toujours autoriser" en mode liste autorisée, les enveloppes de dispatch connues (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) conservent les chemins de l'exécutable interne au lieu des chemins de l'enveloppe. Si le déballage n'est pas sûr, aucune entrée de liste autorisée n'est persistée automatiquement.

## Liens profonds

L'application enregistre le schéma d'URL `openclaw://` pour les actions locales.

### openclaw://agent

Déclenche une requête `agent` de la passerelle.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Paramètres de requête :

-   `message` (obligatoire)
-   `sessionKey` (optionnel)
-   `thinking` (optionnel)
-   `deliver` / `to` / `channel` (optionnel)
-   `timeoutSeconds` (optionnel)
-   `key` (clé optionnelle pour mode sans surveillance)

Sécurité :

-   Sans `key`, l'application demande une confirmation.
-   Sans `key`, l'application impose une limite de message courte pour l'invite de confirmation et ignore `deliver` / `to` / `channel`.
-   Avec une `key` valide, l'exécution est sans surveillance (destinée aux automatisations personnelles).

## Flux d'intégration (typique)

1.  Installez et lancez **OpenClaw.app**.
2.  Complétez la liste de contrôle des permissions (invites TCC).
3.  Assurez-vous que le mode **Local** est actif et que la passerelle est en cours d'exécution.
4.  Installez le CLI si vous voulez un accès terminal.

## Emplacement du répertoire d'état (macOS)

Évitez de placer votre répertoire d'état OpenClaw dans iCloud ou d'autres dossiers synchronisés par le cloud. Les chemins sauvegardés par synchronisation peuvent ajouter de la latence et occasionnellement causer des conflits de verrouillage/synchronisation de fichiers pour les sessions et les identifiants. Préférez un chemin d'état local non synchronisé tel que :

```
OPENCLAW_STATE_DIR=~/.openclaw
```

Si `openclaw doctor` détecte l'état sous :

-   `~/Library/Mobile Documents/com~apple~CloudDocs/...`
-   `~/Library/CloudStorage/...`

il avertira et recommandera de revenir à un chemin local.

## Construction & flux de développement (natif)

-   `cd apps/macos && swift build`
-   `swift run OpenClaw` (ou Xcode)
-   Empaqueter l'application : `scripts/package-mac-app.sh`

## Déboguer la connectivité de la passerelle (CLI macOS)

Utilisez le CLI de débogage pour exercer la même logique de découverte et de poignée de main WebSocket de la passerelle que celle utilisée par l'application macOS, sans lancer l'application.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Options de connexion :

-   `--url <ws://host:port>` : remplacer la configuration
-   `--mode <local|remote>` : résoudre depuis la configuration (par défaut : configuration ou local)
-   `--probe` : forcer une sonde de santé fraîche
-   `--timeout <ms>` : délai d'attente de la requête (par défaut : `15000`)
-   `--json` : sortie structurée pour la comparaison

Options de découverte :

-   `--include-local` : inclure les passerelles qui seraient filtrées comme "locales"
-   `--timeout <ms>` : fenêtre de découverte globale (par défaut : `2000`)
-   `--json` : sortie structurée pour la comparaison

Astuce : comparez avec `openclaw gateway discover --json` pour voir si le pipeline de découverte de l'application macOS (NWBrowser + repli DNS‑SD tailnet) diffère de la découverte basée sur `dns-sd` du CLI Node.

## Plomberie de connexion distante (tunnels SSH)

Lorsque l'application macOS s'exécute en mode **Distant**, elle ouvre un tunnel SSH pour que les composants UI locaux puissent communiquer avec une passerelle distante comme si elle était sur localhost.

### Tunnel de contrôle (port WebSocket de la passerelle)

-   **Objectif :** vérifications de santé, statut, Web Chat, configuration et autres appels du plan de contrôle.
-   **Port local :** le port de la passerelle (par défaut `18789`), toujours stable.
-   **Port distant :** le même port de la passerelle sur l'hôte distant.
-   **Comportement :** pas de port local aléatoire ; l'application réutilise un tunnel sain existant ou le redémarre si nécessaire.
-   **Forme SSH :** `ssh -N -L <local>:127.0.0.1:<remote>` avec les options BatchMode + ExitOnForwardFailure + keepalive.
-   **Rapport d'IP :** le tunnel SSH utilise la boucle locale, donc la passerelle verra l'IP du nœud comme `127.0.0.1`. Utilisez le transport **Direct (ws/wss)** si vous voulez que l'IP réelle du client apparaisse (voir [Accès distant macOS](/platforms/mac/remote)).

Pour les étapes de configuration, voir [Accès distant macOS](/platforms/mac/remote). Pour les détails du protocole, voir [Protocole de la passerelle](/gateway/protocol).

## Documentation associée

-   [Guide d'exploitation de la passerelle](/gateway)
-   [Passerelle (macOS)](/platforms/mac/bundled-gateway)
-   [Permissions macOS](/platforms/mac/permissions)
-   [Canvas](/platforms/mac/canvas)

[Plateformes](/platforms)[Application Linux](/platforms/linux)