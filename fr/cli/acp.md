title: "Pont CLI OpenClaw ACP pour l'intégration du protocole Agent Client"
description: "Apprenez à utiliser le pont CLI OpenClaw ACP pour connecter des IDE via le protocole Agent Client à une passerelle. Configurez des sessions, déboguez avec le client et configurez l'éditeur Zed."
keywords: ["openclaw acp", "protocole agent client", "pont cli", "websocket de passerelle", "configuration éditeur zed", "gestion de session", "débogage client acp", "intégration ide"]
---

  Commandes CLI

  
# acp

Exécute le pont du [Protocole Agent Client (ACP)](https://agentclientprotocol.com/) qui communique avec une passerelle OpenClaw. Cette commande parle ACP via stdio pour les IDE et transmet les requêtes à la passerelle via WebSocket. Elle maintient les sessions ACP mappées aux clés de session de la passerelle.

## Utilisation

```bash
openclaw acp

# Passerelle distante
openclaw acp --url wss://gateway-host:18789 --token <token>

# Passerelle distante (jeton depuis un fichier)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Attacher à une clé de session existante
openclaw acp --session agent:main:main

# Attacher par étiquette (doit déjà exister)
openclaw acp --session-label "support inbox"

# Réinitialiser la clé de session avant la première requête
openclaw acp --session agent:main:main --reset-session
```

## Client ACP (débogage)

Utilisez le client ACP intégré pour vérifier le fonctionnement du pont sans IDE. Il lance le pont ACP et vous permet de saisir des requêtes de manière interactive.

```bash
openclaw acp client

# Pointer le pont lancé vers une passerelle distante
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Remplacer la commande du serveur (par défaut : openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

Modèle de permissions (mode débogage client) :

-   L'approbation automatique est basée sur une liste blanche et ne s'applique qu'aux identifiants d'outils de base de confiance.
-   L'approbation automatique de `read` est limitée au répertoire de travail actuel (`--cwd` lorsqu'il est défini).
-   Les noms d'outils inconnus/non principaux, les lectures hors périmètre et les outils dangereux nécessitent toujours une approbation explicite via une requête.
-   Le `toolCall.kind` fourni par le serveur est traité comme des métadonnées non fiables (pas une source d'autorisation).

## Comment l'utiliser

Utilisez ACP lorsqu'un IDE (ou autre client) parle le Protocole Agent Client et que vous voulez qu'il pilote une session de passerelle OpenClaw.

1.  Assurez-vous que la passerelle est en cours d'exécution (locale ou distante).
2.  Configurez la cible de la passerelle (configuration ou drapeaux).
3.  Configurez votre IDE pour exécuter `openclaw acp` via stdio.

Exemple de configuration (persistante) :

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Exemple d'exécution directe (sans écriture de configuration) :

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# préféré pour la sécurité des processus locaux
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## Sélection des agents

ACP ne choisit pas directement les agents. Il achemine via la clé de session de la passerelle. Utilisez des clés de session limitées à un agent pour cibler un agent spécifique :

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Chaque session ACP correspond à une seule clé de session de passerelle. Un agent peut avoir plusieurs sessions ; ACP utilise par défaut une session isolée `acp:` sauf si vous remplacez la clé ou l'étiquette.

## Configuration de l'éditeur Zed

Ajoutez un agent ACP personnalisé dans `~/.config/zed/settings.json` (ou utilisez l'interface des paramètres de Zed) :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Pour cibler une passerelle ou un agent spécifique :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Dans Zed, ouvrez le panneau Agent et sélectionnez "OpenClaw ACP" pour démarrer un fil de discussion.

## Mappage des sessions

Par défaut, les sessions ACP obtiennent une clé de session de passerelle isolée avec le préfixe `acp:`. Pour réutiliser une session connue, passez une clé ou une étiquette de session :

-   `--session ` : utiliser une clé de session de passerelle spécifique.
-   `--session-label ` : résoudre une session existante par son étiquette.
-   `--reset-session` : créer un nouvel identifiant de session pour cette clé (même clé, nouveau transcript).

Si votre client ACP prend en charge les métadonnées, vous pouvez les remplacer par session :

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

En savoir plus sur les clés de session sur [/concepts/session](../concepts/session.md).

## Options

-   `--url ` : URL WebSocket de la passerelle (par défaut : gateway.remote.url si configuré).
-   `--token ` : jeton d'authentification de la passerelle.
-   `--token-file ` : lire le jeton d'authentification de la passerelle depuis un fichier.
-   `--password ` : mot de passe d'authentification de la passerelle.
-   `--password-file ` : lire le mot de passe d'authentification de la passerelle depuis un fichier.
-   `--session ` : clé de session par défaut.
-   `--session-label ` : étiquette de session par défaut à résoudre.
-   `--require-existing` : échouer si la clé/étiquette de session n'existe pas.
-   `--reset-session` : réinitialiser la clé de session avant la première utilisation.
-   `--no-prefix-cwd` : ne pas préfixer les requêtes avec le répertoire de travail.
-   `--verbose, -v` : journalisation détaillée vers stderr.

Note de sécurité :

-   `--token` et `--password` peuvent être visibles dans les listes de processus locaux sur certains systèmes.
-   Préférez `--token-file`/`--password-file` ou les variables d'environnement (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).
-   Les processus enfants du backend d'exécution ACP reçoivent `OPENCLAW_SHELL=acp`, qui peut être utilisé pour des règles de shell/profil spécifiques au contexte.
-   `openclaw acp client` définit `OPENCLAW_SHELL=acp-client` sur le processus de pont lancé.

### Options du client acp

-   `--cwd ` : répertoire de travail pour la session ACP.
-   `--server ` : commande du serveur ACP (par défaut : `openclaw`).
-   `--server-args <args...>` : arguments supplémentaires passés au serveur ACP.
-   `--server-verbose` : activer la journalisation détaillée sur le serveur ACP.
-   `--verbose, -v` : journalisation détaillée du client.

[Référence CLI](../cli.md)[agent](./agent.md)