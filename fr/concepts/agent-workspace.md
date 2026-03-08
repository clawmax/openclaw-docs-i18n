

  Fondamentaux

  
# Espace de travail de l'agent

L'espace de travail est la maison de l'agent. C'est le seul répertoire de travail utilisé pour les outils de fichier et pour le contexte de l'espace de travail. Gardez-le privé et traitez-le comme une mémoire. Il est séparé de `~/.openclaw/`, qui stocke la configuration, les identifiants et les sessions. **Important :** l'espace de travail est le **cwd par défaut**, pas un bac à sable strict. Les outils résolvent les chemins relatifs par rapport à l'espace de travail, mais les chemins absolus peuvent toujours atteindre d'autres endroits sur l'hôte à moins que le sandboxing ne soit activé. Si vous avez besoin d'isolation, utilisez [`agents.defaults.sandbox`](../gateway/sandboxing.md) (et/ou la configuration de sandbox par agent). Lorsque le sandboxing est activé et que `workspaceAccess` n'est pas `"rw"`, les outils fonctionnent à l'intérieur d'un espace de travail sandbox sous `~/.openclaw/sandboxes`, et non dans votre espace de travail hôte.

## Emplacement par défaut

-   Par défaut : `~/.openclaw/workspace`
-   Si `OPENCLAW_PROFILE` est défini et n'est pas `"default"`, l'emplacement par défaut devient `~/.openclaw/workspace-`.
-   Surchargez-le dans `~/.openclaw/openclaw.json` :

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`, `openclaw configure`, ou `openclaw setup` créera l'espace de travail et y placera les fichiers d'amorçage s'ils sont manquants. Les copies d'amorçage du sandbox n'acceptent que les fichiers réguliers dans l'espace de travail ; les alias de lien symbolique/lien physique qui pointent en dehors de l'espace de travail source sont ignorés. Si vous gérez déjà vous-même les fichiers de l'espace de travail, vous pouvez désactiver la création des fichiers d'amorçage :

```json
{ agent: { skipBootstrap: true } }
```

## Dossiers d'espace de travail supplémentaires

Les installations plus anciennes peuvent avoir créé `~/openclaw`. Garder plusieurs répertoires d'espace de travail peut entraîner des confusions d'authentification ou des dérives d'état, car un seul espace de travail est actif à la fois. **Recommandation :** conservez un seul espace de travail actif. Si vous n'utilisez plus les dossiers supplémentaires, archivez-les ou déplacez-les vers la Corbeille (par exemple `trash ~/openclaw`). Si vous conservez intentionnellement plusieurs espaces de travail, assurez-vous que `agents.defaults.workspace` pointe vers celui qui est actif. `openclaw doctor` avertit lorsqu'il détecte des répertoires d'espace de travail supplémentaires.

## Carte des fichiers de l'espace de travail (signification de chaque fichier)

Voici les fichiers standard qu'OpenClaw attend à l'intérieur de l'espace de travail :

-   `AGENTS.md`
    -   Instructions de fonctionnement pour l'agent et comment il doit utiliser la mémoire.
    -   Chargé au début de chaque session.
    -   Bon endroit pour les règles, les priorités et les détails sur "comment se comporter".
-   `SOUL.md`
    -   Persona, ton et limites.
    -   Chargé à chaque session.
-   `USER.md`
    -   Qui est l'utilisateur et comment s'adresser à lui.
    -   Chargé à chaque session.
-   `IDENTITY.md`
    -   Le nom, l'ambiance et l'emoji de l'agent.
    -   Créé/mis à jour pendant le rituel d'amorçage.
-   `TOOLS.md`
    -   Notes sur vos outils locaux et conventions.
    -   Ne contrôle pas la disponibilité des outils ; ce n'est qu'un guide.
-   `HEARTBEAT.md`
    -   Petite liste de contrôle optionnelle pour les exécutions de heartbeat.
    -   Gardez-la courte pour éviter la consommation de tokens.
-   `BOOT.md`
    -   Liste de contrôle de démarrage optionnelle exécutée au redémarrage de la passerelle lorsque les hooks internes sont activés.
    -   Gardez-la courte ; utilisez l'outil de message pour les envois sortants.
-   `BOOTSTRAP.md`
    -   Rituel de première exécution unique.
    -   Créé uniquement pour un espace de travail entièrement nouveau.
    -   Supprimez-le après la fin du rituel.
-   `memory/YYYY-MM-DD.md`
    -   Journal de mémoire quotidien (un fichier par jour).
    -   Recommandé de lire celui d'aujourd'hui + celui d'hier au démarrage de la session.
-   `MEMORY.md` (optionnel)
    -   Mémoire à long terme organisée.
    -   À charger uniquement dans la session principale, privée (pas dans les contextes partagés/groupes).

Voir [Mémoire](./memory.md) pour le flux de travail et la purge automatique de la mémoire.

-   `skills/` (optionnel)
    -   Compétences spécifiques à l'espace de travail.
    -   Remplace les compétences gérées/groupées en cas de conflit de noms.
-   `canvas/` (optionnel)
    -   Fichiers d'interface utilisateur Canvas pour les affichages de nœuds (par exemple `canvas/index.html`).

Si un fichier d'amorçage est manquant, OpenClaw injecte un marqueur "fichier manquant" dans la session et continue. Les gros fichiers d'amorçage sont tronqués lors de l'injection ; ajustez les limites avec `agents.defaults.bootstrapMaxChars` (par défaut : 20000) et `agents.defaults.bootstrapTotalMaxChars` (par défaut : 150000). `openclaw setup` peut recréer les fichiers par défaut manquants sans écraser les fichiers existants.

## Ce qui n'est PAS dans l'espace de travail

Ces éléments se trouvent sous `~/.openclaw/` et ne doivent PAS être commités dans le dépôt de l'espace de travail :

-   `~/.openclaw/openclaw.json` (configuration)
-   `~/.openclaw/credentials/` (jetons OAuth, clés API)
-   `~/.openclaw/agents//sessions/` (transcriptions de session + métadonnées)
-   `~/.openclaw/skills/` (compétences gérées)

Si vous devez migrer des sessions ou la configuration, copiez-les séparément et gardez-les hors du contrôle de version.

## Sauvegarde Git (recommandée, privée)

Traitez l'espace de travail comme une mémoire privée. Placez-le dans un dépôt git **privé** pour qu'il soit sauvegardé et récupérable. Exécutez ces étapes sur la machine où la passerelle fonctionne (c'est là que se trouve l'espace de travail).

### 1) Initialiser le dépôt

Si git est installé, les espaces de travail entièrement nouveaux sont initialisés automatiquement. Si cet espace de travail n'est pas déjà un dépôt, exécutez :

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) Ajouter un dépôt distant privé (options adaptées aux débutants)

Option A : Interface web GitHub

1.  Créez un nouveau dépôt **privé** sur GitHub.
2.  Ne l'initialisez pas avec un README (évite les conflits de fusion).
3.  Copiez l'URL HTTPS du dépôt distant.
4.  Ajoutez le dépôt distant et poussez :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Option B : GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C : Interface web GitLab

1.  Créez un nouveau dépôt **privé** sur GitLab.
2.  Ne l'initialisez pas avec un README (évite les conflits de fusion).
3.  Copiez l'URL HTTPS du dépôt distant.
4.  Ajoutez le dépôt distant et poussez :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) Mises à jour continues

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## Ne committez pas de secrets

Même dans un dépôt privé, évitez de stocker des secrets dans l'espace de travail :

-   Clés API, jetons OAuth, mots de passe ou identifiants privés.
-   Tout ce qui se trouve sous `~/.openclaw/`.
-   Les dumps bruts de conversations ou les pièces jointes sensibles.

Si vous devez stocker des références sensibles, utilisez des espaces réservés et gardez le vrai secret ailleurs (gestionnaire de mots de passe, variables d'environnement, ou `~/.openclaw/`). `.gitignore` de départ suggéré :

```
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## Déplacer l'espace de travail vers une nouvelle machine

1.  Clonez le dépôt vers le chemin souhaité (par défaut `~/.openclaw/workspace`).
2.  Définissez `agents.defaults.workspace` sur ce chemin dans `~/.openclaw/openclaw.json`.
3.  Exécutez `openclaw setup --workspace ` pour placer les fichiers manquants.
4.  Si vous avez besoin des sessions, copiez `~/.openclaw/agents//sessions/` de l'ancienne machine séparément.

## Notes avancées

-   Le routage multi-agent peut utiliser différents espaces de travail par agent. Voir [Routage des canaux](../channels/channel-routing.md) pour la configuration du routage.
-   Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent utiliser des espaces de travail sandbox par session sous `agents.defaults.sandbox.workspaceRoot`.

[Contexte](./context.md)[OAuth](./oauth.md)