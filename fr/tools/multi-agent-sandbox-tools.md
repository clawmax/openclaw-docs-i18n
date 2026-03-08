

  Coordination des agents

  
# Bac à Sable Multi-Agent & Outils

## Vue d'ensemble

Chaque agent dans une configuration multi-agent peut désormais avoir sa propre :

-   **Configuration de bac à sable** (`agents.list[].sandbox` remplace `agents.defaults.sandbox`)
-   **Restrictions d'outils** (`tools.allow` / `tools.deny`, plus `agents.list[].tools`)

Cela vous permet d'exécuter plusieurs agents avec différents profils de sécurité :

-   Assistant personnel avec accès complet
-   Agents familiaux/professionnels avec des outils restreints
-   Agents publics dans des bacs à sable

`setupCommand` appartient à `sandbox.docker` (global ou par agent) et s'exécute une fois lors de la création du conteneur. L'authentification est par agent : chaque agent lit depuis son propre magasin d'authentification `agentDir` à l'emplacement :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants ne sont **pas** partagés entre les agents. Ne réutilisez jamais `agentDir` entre les agents. Si vous souhaitez partager des identifiants, copiez `auth-profiles.json` dans le `agentDir` de l'autre agent. Pour comprendre le comportement du bac à sable lors de l'exécution, consultez [Bac à Sable](../gateway/sandboxing.md). Pour déboguer "pourquoi est-ce bloqué ?", consultez [Bac à Sable vs Politique d'Outils vs Élevé](../gateway/sandbox-vs-tool-policy-vs-elevated.md) et `openclaw sandbox explain`.

* * *

## Exemples de Configuration

### Exemple 1 : Agent Personnel + Familial Restreint

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Assistant Personnel",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Bot Familial",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Résultat :**

-   Agent `main` : S'exécute sur l'hôte, accès complet aux outils
-   Agent `family` : S'exécute dans Docker (un conteneur par agent), seulement l'outil `read`

* * *

### Exemple 2 : Agent Professionnel avec Bac à Sable Partagé

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### Exemple 2b : Profil de codage global + agent uniquement messagerie

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Résultat :**

-   les agents par défaut obtiennent les outils de codage
-   l'agent `support` est uniquement messagerie (+ outil Slack)

* * *

### Exemple 3 : Différents Modes de Bac à Sable par Agent

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // Valeur par défaut globale
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // Remplacement : main jamais en bac à sable
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // Remplacement : public toujours en bac à sable
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## Priorité de Configuration

Lorsque des configurations globales (`agents.defaults.*`) et spécifiques à un agent (`agents.list[].*`) existent :

### Configuration du Bac à Sable

Les paramètres spécifiques à l'agent remplacent les paramètres globaux :

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Notes :**

-   `agents.list[].sandbox.{docker,browser,prune}.*` remplace `agents.defaults.sandbox.{docker,browser,prune}.*` pour cet agent (ignoré lorsque la portée du bac à sable est résolue à `"shared"`).

### Restrictions d'Outils

L'ordre de filtrage est :

1.  **Profil d'outil** (`tools.profile` ou `agents.list[].tools.profile`)
2.  **Profil d'outil du fournisseur** (`tools.byProvider[provider].profile` ou `agents.list[].tools.byProvider[provider].profile`)
3.  **Politique d'outils globale** (`tools.allow` / `tools.deny`)
4.  **Politique d'outils du fournisseur** (`tools.byProvider[provider].allow/deny`)
5.  **Politique d'outils spécifique à l'agent** (`agents.list[].tools.allow/deny`)
6.  **Politique du fournisseur de l'agent** (`agents.list[].tools.byProvider[provider].allow/deny`)
7.  **Politique d'outils du bac à sable** (`tools.sandbox.tools` ou `agents.list[].tools.sandbox.tools`)
8.  **Politique d'outils du sous-agent** (`tools.subagents.tools`, si applicable)

Chaque niveau peut restreindre davantage les outils, mais ne peut pas réaccorder des outils refusés par des niveaux antérieurs. Si `agents.list[].tools.sandbox.tools` est défini, il remplace `tools.sandbox.tools` pour cet agent. Si `agents.list[].tools.profile` est défini, il remplace `tools.profile` pour cet agent. Les clés d'outils du fournisseur acceptent soit `provider` (par ex. `google-antigravity`) soit `provider/model` (par ex. `openai/gpt-5.2`).

### Groupes d'outils (raccourcis)

Les politiques d'outils (globales, agent, bac à sable) prennent en charge les entrées `group:*` qui se développent en plusieurs outils concrets :

-   `group:runtime` : `exec`, `bash`, `process`
-   `group:fs` : `read`, `write`, `edit`, `apply_patch`
-   `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory` : `memory_search`, `memory_get`
-   `group:ui` : `browser`, `canvas`
-   `group:automation` : `cron`, `gateway`
-   `group:messaging` : `message`
-   `group:nodes` : `nodes`
-   `group:openclaw` : tous les outils intégrés d'OpenClaw (exclut les plugins fournisseurs)

### Mode Élevé

`tools.elevated` est la ligne de base globale (liste d'autorisation basée sur l'expéditeur). `agents.list[].tools.elevated` peut restreindre davantage l'accès élevé pour des agents spécifiques (les deux doivent autoriser). Modèles d'atténuation :

-   Refuser `exec` pour les agents non fiables (`agents.list[].tools.deny: ["exec"]`)
-   Éviter de mettre sur liste d'autorisation des expéditeurs qui routent vers des agents restreints
-   Désactiver l'accès élevé globalement (`tools.elevated.enabled: false`) si vous ne voulez que l'exécution en bac à sable
-   Désactiver l'accès élevé par agent (`agents.list[].tools.elevated.enabled: false`) pour les profils sensibles

* * *

## Migration depuis un Agent Unique

**Avant (agent unique) :**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Après (multi-agent avec différents profils) :**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Les anciennes configurations `agent.*` sont migrées par `openclaw doctor` ; préférez `agents.defaults` + `agents.list` à l'avenir.

* * *

## Exemples de Restriction d'Outils

### Agent en Lecture Seule

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### Agent d'Exécution Sûre (pas de modifications de fichiers)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### Agent Uniquement Communication

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## Piège Courant : "non-main"

`agents.defaults.sandbox.mode: "non-main"` est basé sur `session.mainKey` (par défaut `"main"`), et non sur l'id de l'agent. Les sessions de groupe/canal obtiennent toujours leurs propres clés, elles sont donc traitées comme non-main et seront placées en bac à sable. Si vous voulez qu'un agent ne soit jamais en bac à sable, définissez `agents.list[].sandbox.mode: "off"`.

* * *

## Test

Après avoir configuré le bac à sable et les outils multi-agents :

1.  **Vérifier la résolution des agents :**
    
    Copier
    
    ```bash
    openclaw agents list --bindings
    ```
    
2.  **Vérifier les conteneurs de bac à sable :**
    
    Copier
    
    ```bash
    docker ps --filter "name=openclaw-sbx-"
    ```
    
3.  **Tester les restrictions d'outils :**
    -   Envoyer un message nécessitant des outils restreints
    -   Vérifier que l'agent ne peut pas utiliser les outils refusés
4.  **Surveiller les journaux :**
    
    Copier
    
    ```bash
    tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
    ```
    

* * *

## Dépannage

### Agent non placé en bac à sable malgré mode: "all"

-   Vérifiez s'il existe un `agents.defaults.sandbox.mode` global qui le remplace
-   La configuration spécifique à l'agent a la priorité, donc définissez `agents.list[].sandbox.mode: "all"`

### Outils toujours disponibles malgré la liste de refus

-   Vérifiez l'ordre de filtrage des outils : global → agent → bac à sable → sous-agent
-   Chaque niveau ne peut que restreindre davantage, pas réaccorder
-   Vérifiez avec les journaux : `[tools] filtering tools for agent:${agentId}`

### Conteneur non isolé par agent

-   Définissez `scope: "agent"` dans la configuration de bac à sable spécifique à l'agent
-   La valeur par défaut est `"session"` qui crée un conteneur par session

* * *

## Voir Aussi

-   [Routage Multi-Agent](../concepts/multi-agent.md)
-   [Configuration du Bac à Sable](../gateway/configuration.md#agentsdefaults-sandbox)
-   [Gestion des Sessions](../concepts/session.md)

[Agents ACP](./acp-agents.md)[Création de Compétences](./creating-skills.md)