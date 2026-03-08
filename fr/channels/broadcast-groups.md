

  Configuration

  
# Groupes de diffusion

**Statut :** Expérimental  
**Version :** Ajouté dans 2026.1.9

## Vue d'ensemble

Les groupes de diffusion permettent à plusieurs agents de traiter et de répondre au même message simultanément. Cela vous permet de créer des équipes d'agents spécialisés qui travaillent ensemble dans un seul groupe WhatsApp ou DM — le tout en utilisant un seul numéro de téléphone. Portée actuelle : **WhatsApp uniquement** (canal web). Les groupes de diffusion sont évalués après les listes d'autorisation de canal et les règles d'activation de groupe. Dans les groupes WhatsApp, cela signifie que les diffusions se produisent lorsque OpenClaw répondrait normalement (par exemple : lors d'une mention, selon vos paramètres de groupe).

## Cas d'utilisation

### 1\. Équipes d'agents spécialisés

Déployez plusieurs agents avec des responsabilités atomiques et ciblées :

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Chaque agent traite le même message et fournit sa perspective spécialisée.

### 2\. Support multilingue

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3\. Flux de travail d'assurance qualité

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4\. Automatisation des tâches

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Configuration

### Configuration de base

Ajoutez une section de premier niveau `broadcast` (à côté de `bindings`). Les clés sont les identifiants de pairs WhatsApp :

-   chats de groupe : JID de groupe (ex. `120363403215116621@g.us`)
-   DMs : numéro de téléphone E.164 (ex. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Résultat :** Lorsqu'OpenClaw répondrait dans ce chat, il exécutera les trois agents.

### Stratégie de traitement

Contrôlez comment les agents traitent les messages :

#### Parallèle (Par défaut)

Tous les agents traitent simultanément :

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Séquentiel

Les agents traitent dans l'ordre (un agent attend que le précédent ait terminé) :

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Exemple complet

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Fonctionnement

### Flux des messages

1.  **Message entrant** arrive dans un groupe WhatsApp
2.  **Vérification de diffusion** : Le système vérifie si l'ID du pair est dans `broadcast`
3.  **S'il est dans la liste de diffusion** :
    -   Tous les agents listés traitent le message
    -   Chaque agent a sa propre clé de session et son contexte isolé
    -   Les agents traitent en parallèle (par défaut) ou séquentiellement
4.  **S'il n'est pas dans la liste de diffusion** :
    -   Le routage normal s'applique (première liaison correspondante)

Note : les groupes de diffusion ne contournent pas les listes d'autorisation de canal ni les règles d'activation de groupe (mentions/commandes/etc). Ils modifient uniquement *quels agents s'exécutent* lorsqu'un message est éligible au traitement.

### Isolation des sessions

Chaque agent dans un groupe de diffusion maintient des éléments complètement séparés :

-   **Clés de session** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
-   **Historique de conversation** (l'agent ne voit pas les messages des autres agents)
-   **Espace de travail** (sandboxes séparés si configurés)
-   **Accès aux outils** (listes d'autorisation/interdiction différentes)
-   **Mémoire/contexte** (IDENTITY.md, SOUL.md, etc. séparés)
-   **Tampon de contexte de groupe** (messages récents du groupe utilisés pour le contexte) est partagé par pair, donc tous les agents de diffusion voient le même contexte lorsqu'ils sont déclenchés

Cela permet à chaque agent d'avoir :

-   Différentes personnalités
-   Différents accès aux outils (ex. lecture seule vs lecture-écriture)
-   Différents modèles (ex. opus vs sonnet)
-   Différentes compétences installées

### Exemple : Sessions isolées

Dans le groupe `120363403215116621@g.us` avec les agents `["alfred", "baerbel"]` : **Contexte d'Alfred :**

```yaml
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contexte de Bärbel :**

```yaml
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Bonnes pratiques

### 1\. Gardez les agents ciblés

Concevez chaque agent avec une seule responsabilité claire :

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Bon :** Chaque agent a un seul travail  
❌ **Mauvais :** Un agent générique "aide-développement"

### 2\. Utilisez des noms descriptifs

Rendez clair ce que fait chaque agent :

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3\. Configurez des accès aux outils différents

Donnez aux agents uniquement les outils dont ils ont besoin :

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Lecture seule
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Lecture-écriture
    }
  }
}
```

### 4\. Surveillez les performances

Avec de nombreux agents, considérez :

-   Utiliser `"strategy": "parallel"` (par défaut) pour la vitesse
-   Limiter les groupes de diffusion à 5-10 agents
-   Utiliser des modèles plus rapides pour les agents plus simples

### 5\. Gérez les échecs avec élégance

Les agents échouent indépendamment. L'erreur d'un agent ne bloque pas les autres :

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Compatibilité

### Fournisseurs

Les groupes de diffusion fonctionnent actuellement avec :

-   ✅ WhatsApp (implémenté)
-   🚧 Telegram (prévu)
-   🚧 Discord (prévu)
-   🚧 Slack (prévu)

### Routage

Les groupes de diffusion fonctionnent aux côtés du routage existant :

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

-   `GROUP_A` : Seul Alfred répond (routage normal)
-   `GROUP_B` : agent1 ET agent2 répondent (diffusion)

**Priorité :** `broadcast` a la priorité sur `bindings`.

## Dépannage

### Les agents ne répondent pas

**Vérifiez :**

1.  Les ID d'agents existent dans `agents.list`
2.  Le format de l'ID de pair est correct (ex. `120363403215116621@g.us`)
3.  Les agents ne sont pas dans les listes d'interdiction

**Débogage :**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Un seul agent répond

**Cause :** L'ID de pair pourrait être dans `bindings` mais pas dans `broadcast`. **Solution :** Ajoutez à la configuration de diffusion ou supprimez des liaisons.

### Problèmes de performance

**Si lent avec de nombreux agents :**

-   Réduisez le nombre d'agents par groupe
-   Utilisez des modèles plus légers (sonnet au lieu d'opus)
-   Vérifiez le temps de démarrage du sandbox

## Exemples

### Exemple 1 : Équipe de revue de code

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**L'utilisateur envoie :** Extrait de code  
**Réponses :**

-   code-formatter : "Indentation corrigée et indications de type ajoutées"
-   security-scanner : "⚠️ Vulnérabilité d'injection SQL à la ligne 12"
-   test-coverage : "La couverture est de 45%, tests manquants pour les cas d'erreur"
-   docs-checker : "Docstring manquant pour la fonction `process_data`"

### Exemple 2 : Support multilingue

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Référence API

### Schéma de configuration

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Champs

-   `strategy` (optionnel) : Comment traiter les agents
    -   `"parallel"` (par défaut) : Tous les agents traitent simultanément
    -   `"sequential"` : Les agents traitent dans l'ordre du tableau
-   `[peerId]` : JID de groupe WhatsApp, numéro E.164, ou autre ID de pair
    -   Valeur : Tableau des ID d'agents qui doivent traiter les messages

## Limitations

1.  **Agents max :** Pas de limite stricte, mais 10+ agents peuvent être lents
2.  **Contexte partagé :** Les agents ne voient pas les réponses des autres (par conception)
3.  **Ordre des messages :** Les réponses parallèles peuvent arriver dans n'importe quel ordre
4.  **Limites de débit :** Tous les agents comptent dans les limites de débit WhatsApp

## Améliorations futures

Fonctionnalités prévues :

-   [ ]  Mode contexte partagé (les agents voient les réponses des autres)
-   [ ]  Coordination des agents (les agents peuvent se signaler)
-   [ ]  Sélection dynamique des agents (choisir les agents en fonction du contenu du message)
-   [ ]  Priorités des agents (certains agents répondent avant d'autres)

## Voir aussi

-   [Configuration multi-agent](../tools/multi-agent-sandbox-tools.md)
-   [Configuration du routage](./channel-routing.md)
-   [Gestion des sessions](../concepts/session.md)

[Groupes](./groups.md)[Routage des canaux](./channel-routing.md)