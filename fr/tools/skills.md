

  Compétences

  
# Compétences

OpenClaw utilise des dossiers de compétences **compatibles [AgentSkills](https://agentskills.io)** pour apprendre à l'agent à utiliser des outils. Chaque compétence est un répertoire contenant un fichier `SKILL.md` avec un frontmatter YAML et des instructions. OpenClaw charge les **compétences intégrées** ainsi que des remplacements locaux optionnels, et les filtre au moment du chargement en fonction de l'environnement, de la configuration et de la présence de binaires.

## Emplacements et priorité

Les compétences sont chargées depuis **trois** emplacements :

1.  **Compétences intégrées** : fournies avec l'installation (package npm ou OpenClaw.app)
2.  **Compétences gérées/locales** : `~/.openclaw/skills`
3.  **Compétences d'espace de travail** : `/skills`

En cas de conflit de nom de compétence, la priorité est : `/skills` (la plus haute) → `~/.openclaw/skills` → compétences intégrées (la plus basse) De plus, vous pouvez configurer des dossiers de compétences supplémentaires (priorité la plus basse) via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json`.

## Compétences par agent vs partagées

Dans les configurations **multi-agents**, chaque agent a son propre espace de travail. Cela signifie :

-   **Compétences par agent** se trouvent dans `/skills` pour cet agent uniquement.
-   **Compétences partagées** se trouvent dans `~/.openclaw/skills` (gérées/locales) et sont visibles par **tous les agents** sur la même machine.
-   **Dossiers partagés** peuvent également être ajoutés via `skills.load.extraDirs` (priorité la plus basse) si vous souhaitez un pack de compétences commun utilisé par plusieurs agents.

Si la même compétence existe à plusieurs endroits, la priorité habituelle s'applique : l'espace de travail l'emporte, puis géré/local, puis intégré.

## Plugins + compétences

Les plugins peuvent fournir leurs propres compétences en listant des répertoires `skills` dans `openclaw.plugin.json` (chemins relatifs à la racine du plugin). Les compétences des plugins se chargent lorsque le plugin est activé et participent aux règles normales de priorité des compétences. Vous pouvez les filtrer via `metadata.openclaw.requires.config` sur l'entrée de configuration du plugin. Voir [Plugins](./plugin.md) pour la découverte/configuration et [Outils](../tools.md) pour la surface d'outils que ces compétences enseignent.

## ClawHub (installation + synchronisation)

ClawHub est le registre public de compétences pour OpenClaw. Parcourez-le à l'adresse [https://clawhub.com](https://clawhub.com). Utilisez-le pour découvrir, installer, mettre à jour et sauvegarder des compétences. Guide complet : [ClawHub](./clawhub.md). Flux courants :

-   Installer une compétence dans votre espace de travail :
    -   `clawhub install <skill-slug>`
-   Mettre à jour toutes les compétences installées :
    -   `clawhub update --all`
-   Synchroniser (analyser + publier les mises à jour) :
    -   `clawhub sync --all`

Par défaut, `clawhub` installe dans `./skills` sous votre répertoire de travail actuel (ou revient à l'espace de travail OpenClaw configuré). OpenClaw le récupère en tant que `/skills` lors de la prochaine session.

## Notes de sécurité

-   Traitez les compétences tierces comme du **code non fiable**. Lisez-les avant de les activer.
-   Préférez les exécutions en bac à sable pour les entrées non fiables et les outils risqués. Voir [Sandboxing](../gateway/sandboxing.md).
-   La découverte de compétences dans l'espace de travail et les dossiers supplémentaires n'accepte que les racines de compétences et les fichiers `SKILL.md` dont le chemin réel résolu reste à l'intérieur de la racine configurée.
-   `skills.entries.*.env` et `skills.entries.*.apiKey` injectent des secrets dans le processus **hôte** pour le tour de cet agent (pas dans le bac à sable). Gardez les secrets hors des invites et des journaux.
-   Pour un modèle de menace plus large et des listes de contrôle, voir [Sécurité](../gateway/security.md).

## Format (AgentSkills + Pi-compatible)

`SKILL.md` doit inclure au minimum :

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

Notes :

-   Nous suivons la spécification AgentSkills pour la mise en page/intention.
-   L'analyseur utilisé par l'agent embarqué ne prend en charge que les clés de frontmatter **sur une seule ligne**.
-   `metadata` doit être un **objet JSON sur une seule ligne**.
-   Utilisez `{baseDir}` dans les instructions pour référencer le chemin du dossier de la compétence.
-   Clés de frontmatter optionnelles :
    -   `homepage` — URL affichée comme "Site Web" dans l'interface utilisateur des compétences macOS (également supportée via `metadata.openclaw.homepage`).
    -   `user-invocable` — `true|false` (par défaut : `true`). Lorsque `true`, la compétence est exposée en tant que commande slash utilisateur.
    -   `disable-model-invocation` — `true|false` (par défaut : `false`). Lorsque `true`, la compétence est exclue de l'invite du modèle (toujours disponible via l'invocation utilisateur).
    -   `command-dispatch` — `tool` (optionnel). Lorsqu'il est défini sur `tool`, la commande slash contourne le modèle et est envoyée directement à un outil.
    -   `command-tool` — nom de l'outil à invoquer lorsque `command-dispatch: tool` est défini.
    -   `command-arg-mode` — `raw` (par défaut). Pour l'envoi à un outil, transmet la chaîne d'arguments brute à l'outil (pas d'analyse par le cœur). L'outil est invoqué avec les paramètres : `{ command: "", commandName: "", skillName: "" }`.

## Filtrage (filtres au moment du chargement)

OpenClaw **filtre les compétences au moment du chargement** en utilisant `metadata` (JSON sur une seule ligne) :

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

Champs sous `metadata.openclaw` :

-   `always: true` — inclut toujours la compétence (ignore les autres filtres).
-   `emoji` — emoji optionnel utilisé par l'interface utilisateur des compétences macOS.
-   `homepage` — URL optionnelle affichée comme "Site Web" dans l'interface utilisateur des compétences macOS.
-   `os` — liste optionnelle de plateformes (`darwin`, `linux`, `win32`). Si définie, la compétence n'est éligible que sur ces systèmes d'exploitation.
-   `requires.bins` — liste ; chaque élément doit exister dans `PATH`.
-   `requires.anyBins` — liste ; au moins un élément doit exister dans `PATH`.
-   `requires.env` — liste ; la variable d'environnement doit exister **ou** être fournie dans la configuration.
-   `requires.config` — liste de chemins `openclaw.json` qui doivent être évalués à vrai.
-   `primaryEnv` — nom de la variable d'environnement associée à `skills.entries..apiKey`.
-   `install` — tableau optionnel de spécifications d'installateur utilisées par l'interface utilisateur des compétences macOS (brew/node/go/uv/download).

Note sur le bac à sable :

-   `requires.bins` est vérifié sur l'**hôte** au moment du chargement de la compétence.
-   Si un agent est en bac à sable, le binaire doit également exister **à l'intérieur du conteneur**. Installez-le via `agents.defaults.sandbox.docker.setupCommand` (ou une image personnalisée). `setupCommand` s'exécute une fois après la création du conteneur. Les installations de paquets nécessitent également une sortie réseau, un système de fichiers racine accessible en écriture et un utilisateur root dans le bac à sable. Exemple : la compétence `summarize` (`skills/summarize/SKILL.md`) a besoin du CLI `summarize` dans le conteneur du bac à sable pour s'exécuter là-bas.

Exemple d'installateur :

```
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

Notes :

-   Si plusieurs installateurs sont listés, la passerelle choisit une **seule** option préférée (brew lorsqu'il est disponible, sinon node).
-   Si tous les installateurs sont `download`, OpenClaw liste chaque entrée pour que vous puissiez voir les artefacts disponibles.
-   Les spécifications d'installateur peuvent inclure `os: ["darwin"|"linux"|"win32"]` pour filtrer les options par plateforme.
-   Les installations Node respectent `skills.install.nodeManager` dans `openclaw.json` (par défaut : npm ; options : npm/pnpm/yarn/bun). Cela n'affecte que les **installations de compétences** ; l'environnement d'exécution de la passerelle doit toujours être Node (Bun n'est pas recommandé pour WhatsApp/Telegram).
-   Installations Go : si `go` est manquant et que `brew` est disponible, la passerelle installe Go via Homebrew d'abord et définit `GOBIN` sur le `bin` de Homebrew lorsque c'est possible.
-   Installations par téléchargement : `url` (requis), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (par défaut : auto lorsque l'archive est détectée), `stripComponents`, `targetDir` (par défaut : `~/.openclaw/tools/`).

Si aucun `metadata.openclaw` n'est présent, la compétence est toujours éligible (sauf si désactivée dans la configuration ou bloquée par `skills.allowBundled` pour les compétences intégrées).

## Remplacements de configuration (~/.openclaw/openclaw.json)

Les compétences intégrées/gérées peuvent être activées/désactivées et recevoir des valeurs d'environnement :

```json
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

Note : si le nom de la compétence contient des traits d'union, mettez la clé entre guillemets (JSON5 autorise les clés entre guillemets). Les clés de configuration correspondent par défaut au **nom de la compétence**. Si une compétence définit `metadata.openclaw.skillKey`, utilisez cette clé sous `skills.entries`. Règles :

-   `enabled: false` désactive la compétence même si elle est intégrée/installée.
-   `env` : injecté **seulement si** la variable n'est pas déjà définie dans le processus.
-   `apiKey` : commodité pour les compétences qui déclarent `metadata.openclaw.primaryEnv`. Prend en charge une chaîne de texte brut ou un objet SecretRef (`{ source, provider, id }`).
-   `config` : sac optionnel pour des champs personnalisés par compétence ; les clés personnalisées doivent se trouver ici.
-   `allowBundled` : liste d'autorisation optionnelle pour les compétences **intégrées** uniquement. Si définie, seules les compétences intégrées dans la liste sont éligibles (les compétences gérées/d'espace de travail ne sont pas affectées).

## Injection d'environnement (par exécution d'agent)

Lorsqu'une exécution d'agent commence, OpenClaw :

1.  Lit les métadonnées de la compétence.
2.  Applique tout `skills.entries..env` ou `skills.entries..apiKey` à `process.env`.
3.  Construit l'invite système avec les compétences **éligibles**.
4.  Restaure l'environnement d'origine après la fin de l'exécution.

Cela est **limité à l'exécution de l'agent**, pas à un environnement shell global.

## Instantané de session (performance)

OpenClaw prend un instantané des compétences éligibles **lorsqu'une session démarre** et réutilise cette liste pour les tours suivants dans la même session. Les changements apportés aux compétences ou à la configuration prennent effet lors de la prochaine nouvelle session. Les compétences peuvent également être rafraîchies en cours de session lorsque le surveillant de compétences est activé ou lorsqu'un nouveau nœud distant éligible apparaît (voir ci-dessous). Considérez cela comme un **rechargement à chaud** : la liste rafraîchie est récupérée au prochain tour de l'agent.

## Nœuds macOS distants (passerelle Linux)

Si la passerelle s'exécute sur Linux mais qu'un **nœud macOS** est connecté **avec `system.run` autorisé** (sécurité des approbations d'exécution non définie sur `deny`), OpenClaw peut traiter les compétences spécifiques à macOS comme éligibles lorsque les binaires requis sont présents sur ce nœud. L'agent doit exécuter ces compétences via l'outil `nodes` (généralement `nodes.run`). Cela repose sur le nœud signalant sa prise en charge des commandes et sur une sonde de binaire via `system.run`. Si le nœud macOS se déconnecte plus tard, les compétences restent visibles ; les invocations peuvent échouer jusqu'à ce que le nœud se reconnecte.

## Surveillant de compétences (rafraîchissement automatique)

Par défaut, OpenClaw surveille les dossiers de compétences et met à jour l'instantané des compétences lorsque les fichiers `SKILL.md` changent. Configurez cela sous `skills.load` :

```json
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Impact sur les tokens (liste des compétences)

Lorsque des compétences sont éligibles, OpenClaw injecte une liste XML compacte des compétences disponibles dans l'invite système (via `formatSkillsForPrompt` dans `pi-coding-agent`). Le coût est déterministe :

-   **Surcharge de base (seulement quand ≥1 compétence) :** 195 caractères.
-   **Par compétence :** 97 caractères + la longueur des valeurs échappées en XML ``, `` et ``.

Formule (caractères) :

```bash
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notes :

-   L'échappement XML transforme `& < > " '` en entités (`&amp;`, `<`, etc.), augmentant la longueur.
-   Les comptes de tokens varient selon le tokeniseur du modèle. Une estimation approximative de type OpenAI est d'environ 4 caractères/token, donc **97 caractères ≈ 24 tokens** par compétence plus vos longueurs de champs réelles.

## Cycle de vie des compétences gérées

OpenClaw fournit un ensemble de base de compétences en tant que **compétences intégrées** dans le cadre de l'installation (package npm ou OpenClaw.app). `~/.openclaw/skills` existe pour les remplacements locaux (par exemple, épingler/corriger une compétence sans changer la copie intégrée). Les compétences d'espace de travail appartiennent à l'utilisateur et remplacent les deux en cas de conflit de nom.

## Référence de configuration

Voir [Configuration des compétences](./skills-config.md) pour le schéma de configuration complet.

## Vous cherchez plus de compétences ?

Parcourez [https://clawhub.com](https://clawhub.com).

* * *

[Commandes Slash](./slash-commands.md)[Configuration des compétences](./skills-config.md)