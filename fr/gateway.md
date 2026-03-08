

  Passerelle

  
# Runbook de la Passerelle

Utilisez cette page pour le démarrage jour-1 et les opérations jour-2 du service Passerelle.

## Démarrage local en 5 minutes

### Étape 1 : Démarrer la Passerelle

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

### Étape 2 : Vérifier l'intégrité du service

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

État sain de base : `Runtime: running` et `RPC probe: ok`.

### Étape 3 : Valider la disponibilité des canaux

```bash
openclaw channels status --probe
```

 

> **ℹ️** Le rechargement de la configuration de la passerelle surveille le chemin du fichier de configuration actif (résolu à partir des valeurs par défaut du profil/état, ou `OPENCLAW_CONFIG_PATH` lorsqu'il est défini). Le mode par défaut est `gateway.reload.mode="hybrid"`.

## Modèle d'exécution

-   Un processus toujours actif pour le routage, le plan de contrôle et les connexions de canaux.
-   Port multiplexé unique pour :
    -   WebSocket contrôle/RPC
    -   APIs HTTP (compatible OpenAI, Réponses, invocation d'outils)
    -   Interface de contrôle et hooks
-   Mode de liaison par défaut : `loopback`.
-   L'authentification est requise par défaut (`gateway.auth.token` / `gateway.auth.password`, ou `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Priorité du port et de la liaison

| Paramètre | Ordre de résolution |
| --- | --- |
| Port de la passerelle | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Mode de liaison | CLI/override → `gateway.bind` → `loopback` |

### Modes de rechargement à chaud

| `gateway.reload.mode` | Comportement |
| --- | --- |
| `off` | Pas de rechargement de configuration |
| `hot` | Applique uniquement les changements sûrs à chaud |
| `restart` | Redémarre sur les changements nécessitant un rechargement |
| `hybrid` (par défaut) | Applique à chaud quand c'est sûr, redémarre quand c'est nécessaire |

## Ensemble de commandes opérateur

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## Accès à distance

Préféré : Tailscale/VPN. Solution de repli : tunnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Ensuite, connectez les clients localement à `ws://127.0.0.1:18789`.

> **⚠️** Si l'authentification de la passerelle est configurée, les clients doivent toujours envoyer l'authentification (`token`/`password`) même via des tunnels SSH.

 Voir : [Passerelle distante](./gateway/remote.md), [Authentification](./gateway/authentication.md), [Tailscale](./gateway/tailscale.md).

## Supervision et cycle de vie du service

Utilisez des exécutions supervisées pour une fiabilité de type production.

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## Passerelles multiples sur un même hôte

La plupart des configurations doivent exécuter **une** seule Passerelle. Utilisez-en plusieurs uniquement pour une isolation/redondance stricte (par exemple un profil de secours). Liste de contrôle par instance :

-   `gateway.port` unique
-   `OPENCLAW_CONFIG_PATH` unique
-   `OPENCLAW_STATE_DIR` unique
-   `agents.defaults.workspace` unique

Exemple :

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Voir : [Passerelles multiples](./gateway/multiple-gateways.md).

### Chemin rapide du profil de développement

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Les valeurs par défaut incluent un état/config isolé et un port de passerelle de base `19001`.

## Référence rapide du protocole (vue opérateur)

-   La première trame du client doit être `connect`.
-   La passerelle renvoie un snapshot `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limites/politique).
-   Requêtes : `req(method, params)` → `res(ok/payload|error)`.
-   Événements courants : `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Les exécutions d'agent sont en deux étapes :

1.  Accusé de réception immédiat (`status:"accepted"`)
2.  Réponse de finalisation (`status:"ok"|"error"`), avec des événements `agent` diffusés entre les deux.

Voir la documentation complète du protocole : [Protocole de la Passerelle](./gateway/protocol.md).

## Vérifications opérationnelles

### Vivacité

-   Ouvrir WS et envoyer `connect`.
-   Attendre une réponse `hello-ok` avec un snapshot.

### Disponibilité

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Récupération d'écart

Les événements ne sont pas rejoués. En cas d'écart de séquence, rafraîchir l'état (`health`, `system-presence`) avant de continuer.

## Signatures courantes de défaillance

| Signature | Problème probable |
| --- | --- |
| `refusing to bind gateway ... without auth` | Liaison non-loopback sans token/mot de passe |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflit de port |
| `Gateway start blocked: set gateway.mode=local` | Configuration en mode distant |
| `unauthorized` pendant la connexion | Incompatibilité d'authentification entre le client et la passerelle |

Pour des échelles de diagnostic complètes, utilisez [Dépannage de la Passerelle](./gateway/troubleshooting.md).

## Garanties de sécurité

-   Les clients du protocole de la passerelle échouent rapidement lorsque la Passerelle est indisponible (pas de repli implicite sur le canal direct).
-   Les premières trames invalides/non-connect sont rejetées et la connexion est fermée.
-   L'arrêt gracieux émet un événement `shutdown` avant la fermeture du socket.

* * *

Liens connexes :

-   [Dépannage](./gateway/troubleshooting.md)
-   [Processus en arrière-plan](./gateway/background-process.md)
-   [Configuration](./gateway/configuration.md)
-   [Intégrité](./gateway/health.md)
-   [Doctor](./gateway/doctor.md)
-   [Authentification](./gateway/authentication.md)

[Configuration](./gateway/configuration.md)