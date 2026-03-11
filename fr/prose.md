

  Extensions

  
# OpenProse

OpenProse est un format de flux de travail portable et markdown-first pour orchestrer des sessions IA. Dans OpenClaw, il est fourni sous forme de plugin qui installe un pack de compétences OpenProse ainsi qu'une commande slash `/prose`. Les programmes vivent dans des fichiers `.prose` et peuvent générer plusieurs sous-agents avec un flux de contrôle explicite. Site officiel : [https://www.prose.md](https://www.prose.md)

## Ce qu'il peut faire

-   Recherche et synthèse multi-agents avec parallélisme explicite.
-   Flux de travail reproductibles et sûrs pour approbation (revue de code, triage d'incidents, pipelines de contenu).
-   Programmes `.prose` réutilisables que vous pouvez exécuter sur les moteurs d'exécution d'agents pris en charge.

## Installer + activer

Les plugins intégrés sont désactivés par défaut. Activez OpenProse :

```bash
openclaw plugins enable open-prose
```

Redémarrez la passerelle après avoir activé le plugin. Pour une installation locale de développement : `openclaw plugins install ./extensions/open-prose` Documentation associée : [Plugins](./tools/plugin.md), [Manifeste du plugin](./plugins/manifest.md), [Compétences](./tools/skills.md).

## Commande slash

OpenProse enregistre `/prose` comme une commande de compétence invocable par l'utilisateur. Elle achemine vers les instructions de la machine virtuelle OpenProse et utilise les outils OpenClaw en arrière-plan. Commandes courantes :

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Exemple : un fichier .prose simple

```bash
# Recherche + synthèse avec deux agents fonctionnant en parallèle.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## Emplacements des fichiers

OpenProse conserve l'état sous `.prose/` dans votre espace de travail :

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Les agents persistants au niveau utilisateur se trouvent dans :

```
~/.prose/agents/
```

## Modes d'état

OpenProse prend en charge plusieurs moteurs d'état :

-   **système de fichiers** (par défaut) : `.prose/runs/...`
-   **in-context** : transitoire, pour les petits programmes
-   **sqlite** (expérimental) : nécessite le binaire `sqlite3`
-   **postgres** (expérimental) : nécessite `psql` et une chaîne de connexion

Notes :

-   sqlite/postgres sont optionnels et expérimentaux.
-   Les identifiants postgres transitent dans les journaux des sous-agents ; utilisez une base de données dédiée avec des privilèges minimum.

## Programmes distants

`/prose run <handle/slug>` se résout en `https://p.prose.md//`. Les URL directes sont récupérées telles quelles. Cela utilise l'outil `web_fetch` (ou `exec` pour POST).

## Correspondance du moteur d'exécution OpenClaw

Les programmes OpenProse correspondent aux primitives OpenClaw :

| Concept OpenProse | Outil OpenClaw |
| --- | --- |
| Démarrer une session / Outil de tâche | `sessions_spawn` |
| Lecture/écriture de fichier | `read` / `write` |
| Récupération web | `web_fetch` |

Si votre liste d'autorisation d'outils bloque ces outils, les programmes OpenProse échoueront. Voir [Configuration des compétences](./tools/skills-config.md).

## Sécurité + approbations

Traitez les fichiers `.prose` comme du code. Examinez-les avant de les exécuter. Utilisez les listes d'autorisation d'outils OpenClaw et les portails d'approbation pour contrôler les effets secondaires. Pour des flux de travail déterministes et soumis à approbation, comparez avec [Lobster](./tools/lobster.md).

[Outils d'agent de plugin](./plugins/agent-tools.md)[Hooks](./automation/hooks.md)

---