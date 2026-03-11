

  Outils intégrés

  
# Lobster

Lobster est un shell de workflow qui permet à OpenClaw d'exécuter des séquences d'outils multi-étapes comme une opération unique et déterministe avec des points de contrôle d'approbation explicites.

## Accroche

Votre assistant peut construire les outils qui le gèrent lui-même. Demandez un workflow, et 30 minutes plus tard vous avez un CLI plus des pipelines qui s'exécutent en un seul appel. Lobster est la pièce manquante : des pipelines déterministes, des approbations explicites et un état reprise.

## Pourquoi

Aujourd'hui, les workflows complexes nécessitent de nombreux appels d'outils aller-retour. Chaque appel coûte des tokens, et le LLM doit orchestrer chaque étape. Lobster déplace cette orchestration dans un runtime typé :

-   **Un appel au lieu de plusieurs** : OpenClaw exécute un seul appel d'outil Lobster et obtient un résultat structuré.
-   **Approbations intégrées** : Les effets de bord (envoyer un email, poster un commentaire) interrompent le workflow jusqu'à approbation explicite.
-   **Reprise** : Les workflows interrompus retournent un token ; approuvez et reprenez sans tout ré-exécuter.

## Pourquoi un DSL au lieu de programmes simples ?

Lobster est intentionnellement petit. Le but n'est pas "un nouveau langage", c'est une spécification de pipeline prévisible et adaptée à l'IA avec des approbations et des tokens de reprise de première classe.

-   **Approuver/reprendre est intégré** : Un programme normal peut demander à un humain, mais il ne peut pas *mettre en pause et reprendre* avec un token durable sans que vous inventiez vous-même ce runtime.
-   **Déterminisme + auditabilité** : Les pipelines sont des données, donc ils sont faciles à journaliser, comparer, rejouer et revoir.
-   **Surface contrainte pour l'IA** : Une grammaire minuscule + le piping JSON réduit les chemins de code "créatifs" et rend la validation réaliste.
-   **Politique de sécurité intégrée** : Les timeouts, limites de sortie, vérifications de sandbox et listes d'autorisation sont appliqués par le runtime, pas par chaque script.
-   **Toujours programmable** : Chaque étape peut appeler n'importe quel CLI ou script. Si vous voulez du JS/TS, générez des fichiers `.lobster` à partir du code.

## Comment ça marche

OpenClaw lance le CLI local `lobster` en **mode outil** et analyse une enveloppe JSON depuis stdout. Si le pipeline est mis en pause pour approbation, l'outil retourne un `resumeToken` pour continuer plus tard.

## Modèle : petit CLI + pipes JSON + approbations

Construisez de petites commandes qui parlent JSON, puis enchaînez-les en un seul appel Lobster. (Noms de commandes d'exemple ci-dessous — remplacez par les vôtres.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Si le pipeline demande une approbation, reprenez avec le token :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

L'IA déclenche le workflow ; Lobster exécute les étapes. Les portes d'approbation rendent les effets de bord explicites et auditable. Exemple : mapper des éléments d'entrée en appels d'outils :

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## Étapes LLM uniquement JSON (llm-task)

Pour les workflows qui nécessitent une **étape LLM structurée**, activez l'outil plugin optionnel `llm-task` et appelez-le depuis Lobster. Cela garde le workflow déterministe tout en vous permettant de classer/résumer/rédiger avec un modèle. Activez l'outil :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Utilisez-le dans un pipeline :

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Voir [LLM Task](./llm-task.md) pour les détails et options de configuration.

## Fichiers de workflow (.lobster)

Lobster peut exécuter des fichiers de workflow YAML/JSON avec les champs `name`, `args`, `steps`, `env`, `condition`, et `approval`. Dans les appels d'outil OpenClaw, définissez `pipeline` sur le chemin du fichier.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notes :

-   `stdin: $step.stdout` et `stdin: $step.json` passent la sortie d'une étape précédente.
-   `condition` (ou `when`) peut conditionner des étapes sur `$step.approved`.

## Installer Lobster

Installez le CLI Lobster sur **le même hôte** qui exécute la Gateway OpenClaw (voir le [dépôt Lobster](https://github.com/openclaw/lobster)), et assurez-vous que `lobster` est dans le `PATH`.

## Activer l'outil

Lobster est un outil plugin **optionnel** (non activé par défaut). Recommandé (additif, sûr) :

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Ou par agent :

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Évitez d'utiliser `tools.allow: ["lobster"]` sauf si vous souhaitez exécuter en mode liste d'autorisation restrictive. Note : les listes d'autorisation sont opt-in pour les plugins optionnels. Si votre liste d'autorisation ne nomme que des outils plugins (comme `lobster`), OpenClaw garde les outils de base activés. Pour restreindre les outils de base, incluez les outils ou groupes de base que vous voulez dans la liste d'autorisation également.

## Exemple : Tri d'emails

Sans Lobster :

```
Utilisateur : "Vérifie mes emails et rédige des réponses"
→ openclaw appelle gmail.list
→ LLM résume
→ Utilisateur : "rédige des réponses pour #2 et #5"
→ LLM rédige
→ Utilisateur : "envoie #2"
→ openclaw appelle gmail.send
(répéter quotidiennement, pas de mémoire de ce qui a été trié)
```

Avec Lobster :

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Retourne une enveloppe JSON (tronquée) :

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

L'utilisateur approuve → reprise :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un workflow. Déterministe. Sûr.

## Paramètres de l'outil

### run

Exécute un pipeline en mode outil.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Exécute un fichier de workflow avec des arguments :

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

Continue un workflow interrompu après approbation.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### Entrées optionnelles

-   `cwd` : Répertoire de travail relatif pour le pipeline (doit rester dans le répertoire de travail du processus actuel).
-   `timeoutMs` : Tue le sous-processus s'il dépasse cette durée (par défaut : 20000).
-   `maxStdoutBytes` : Tue le sous-processus si stdout dépasse cette taille (par défaut : 512000).
-   `argsJson` : Chaîne JSON passée à `lobster run --args-json` (fichiers de workflow uniquement).

## Enveloppe de sortie

Lobster retourne une enveloppe JSON avec l'un des trois statuts :

-   `ok` → terminé avec succès
-   `needs_approval` → en pause ; `requiresApproval.resumeToken` est requis pour reprendre
-   `cancelled` → explicitement refusé ou annulé

L'outil expose l'enveloppe à la fois dans `content` (JSON joli) et `details` (objet brut).

## Approbations

Si `requiresApproval` est présent, inspectez l'invite et décidez :

-   `approve: true` → reprendre et continuer les effets de bord
-   `approve: false` → annuler et finaliser le workflow

Utilisez `approve --preview-from-stdin --limit N` pour attacher un aperçu JSON aux demandes d'approbation sans colle jq/heredoc personnalisée. Les tokens de reprise sont maintenant compacts : Lobster stocke l'état de reprise du workflow sous son répertoire d'état et renvoie une petite clé token.

## OpenProse

OpenProse fonctionne bien avec Lobster : utilisez `/prose` pour orchestrer une préparation multi-agent, puis exécutez un pipeline Lobster pour des approbations déterministes. Si un programme Prose a besoin de Lobster, autorisez l'outil `lobster` pour les sous-agents via `tools.subagents.tools`. Voir [OpenProse](../prose.md).

## Sécurité

-   **Sous-processus local uniquement** — pas d'appels réseau depuis le plugin lui-même.
-   **Pas de secrets** — Lobster ne gère pas OAuth ; il appelle les outils OpenClaw qui le font.
-   **Conscient du sandbox** — désactivé lorsque le contexte de l'outil est en sandbox.
-   **Durci** — nom d'exécutable fixe (`lobster`) dans `PATH` ; timeouts et limites de sortie appliqués.

## Dépannage

-   **`lobster subprocess timed out`** → augmentez `timeoutMs`, ou divisez un long pipeline.
-   **`lobster output exceeded maxStdoutBytes`** → augmentez `maxStdoutBytes` ou réduisez la taille de sortie.
-   **`lobster returned invalid JSON`** → assurez-vous que le pipeline s'exécute en mode outil et n'imprime que du JSON.
-   **`lobster failed (code …)`** → exécutez le même pipeline dans un terminal pour inspecter stderr.

## En savoir plus

-   [Plugins](./plugin.md)
-   [Création d'outils plugin](../plugins/agent-tools.md)

## Étude de cas : workflows communautaires

Un exemple public : un CLI "second cerveau" + pipelines Lobster qui gèrent trois coffres Markdown (personnel, partenaire, partagé). Le CLI émet du JSON pour les statistiques, listes de boîte de réception et scans obsolètes ; Lobster enchaîne ces commandes en workflows comme `weekly-review`, `inbox-triage`, `memory-consolidation`, et `shared-task-sync`, chacun avec des portes d'approbation. L'IA gère le jugement (catégorisation) quand disponible et revient à des règles déterministes sinon.

-   Fil : [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
-   Dépôt : [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM Task](./llm-task.md)[Détection de boucle d'outils](./loop-detection.md)