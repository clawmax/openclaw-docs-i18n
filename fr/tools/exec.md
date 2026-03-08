

  Outils intégrés

  
# Outil Exec

Exécutez des commandes shell dans l'espace de travail. Prend en charge l'exécution en premier plan + arrière-plan via `process`. Si `process` est interdit, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`. Les sessions en arrière-plan sont limitées par agent ; `process` ne voit que les sessions du même agent.

## Paramètres

-   `command` (obligatoire)
-   `workdir` (par défaut : répertoire de travail courant)
-   `env` (surcharges clé/valeur)
-   `yieldMs` (par défaut 10000) : passage automatique en arrière-plan après un délai
-   `background` (booléen) : passage immédiat en arrière-plan
-   `timeout` (secondes, par défaut 1800) : arrêt à expiration
-   `pty` (booléen) : exécuter dans un pseudo-terminal quand disponible (CLI TTY uniquement, agents de codage, interfaces terminal)
-   `host` (`sandbox | gateway | node`) : où exécuter
-   `security` (`deny | allowlist | full`) : mode d'application pour `gateway`/`node`
-   `ask` (`off | on-miss | always`) : invites d'approbation pour `gateway`/`node`
-   `node` (chaîne) : identifiant/nom du nœud pour `host=node`
-   `elevated` (booléen) : demande le mode élevé (hôte gateway) ; `security=full` n'est forcé que lorsque `elevated` se résout à `full`

Notes :

-   `host` est par défaut `sandbox`.
-   `elevated` est ignoré lorsque le sandboxing est désactivé (exec s'exécute déjà sur l'hôte).
-   Les approbations `gateway`/`node` sont contrôlées par `~/.openclaw/exec-approvals.json`.
-   `node` nécessite un nœud appairé (application compagnon ou hôte de nœud sans interface).
-   Si plusieurs nœuds sont disponibles, définissez `exec.node` ou `tools.exec.node` pour en sélectionner un.
-   Sur les hôtes non-Windows, exec utilise `SHELL` quand il est défini ; si `SHELL` est `fish`, il préfère `bash` (ou `sh`) depuis `PATH` pour éviter les scripts incompatibles avec fish, puis revient à `SHELL` si aucun des deux n'existe.
-   Sur les hôtes Windows, exec préfère la découverte de PowerShell 7 (`pwsh`) (Program Files, ProgramW6432, puis PATH), puis revient à Windows PowerShell 5.1.
-   L'exécution hôte (`gateway`/`node`) rejette `env.PATH` et les surcharges de chargeur (`LD_*`/`DYLD_*`) pour empêcher le détournement de binaires ou l'injection de code.
-   OpenClaw définit `OPENCLAW_SHELL=exec` dans l'environnement de la commande lancée (y compris l'exécution PTY et sandbox) afin que les règles shell/profile puissent détecter le contexte de l'outil exec.
-   Important : le sandboxing est **désactivé par défaut**. Si le sandboxing est désactivé et que `host=sandbox` est explicitement configuré/demandé, exec échoue maintenant en mode fermé au lieu de s'exécuter silencieusement sur l'hôte gateway. Activez le sandboxing ou utilisez `host=gateway` avec approbations.
-   Les vérifications préalables des scripts (pour les erreurs de syntaxe shell Python/Node courantes) n'inspectent que les fichiers à l'intérieur de la limite effective de `workdir`. Si un chemin de script se résout en dehors de `workdir`, la vérification préalable est ignorée pour ce fichier.

## Configuration

-   `tools.exec.notifyOnExit` (par défaut : true) : quand true, les sessions exec en arrière-plan mettent en file d'attente un événement système et demandent un battement de cœur à la sortie.
-   `tools.exec.approvalRunningNoticeMs` (par défaut : 10000) : émet un seul avis "running" quand une exécution exec soumise à approbation dure plus longtemps que ce délai (0 désactive).
-   `tools.exec.host` (par défaut : `sandbox`)
-   `tools.exec.security` (par défaut : `deny` pour sandbox, `allowlist` pour gateway + node quand non défini)
-   `tools.exec.ask` (par défaut : `on-miss`)
-   `tools.exec.node` (par défaut : non défini)
-   `tools.exec.pathPrepend` : liste de répertoires à préfixer à `PATH` pour les exécutions exec (gateway + sandbox uniquement).
-   `tools.exec.safeBins` : binaires sûrs en entrée standard uniquement qui peuvent s'exécuter sans entrées d'autorisation explicites. Pour les détails du comportement, voir [Binaires sûrs](./exec-approvals.md#safe-bins-stdin-only).
-   `tools.exec.safeBinTrustedDirs` : répertoires explicites supplémentaires approuvés pour les vérifications de chemin des `safeBins`. Les entrées `PATH` ne sont jamais automatiquement approuvées. Les valeurs par défaut intégrées sont `/bin` et `/usr/bin`.
-   `tools.exec.safeBinProfiles` : politique d'argv personnalisée facultative par binaire sûr (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

Exemple :

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### Gestion de PATH

-   `host=gateway` : fusionne votre `PATH` de shell de connexion dans l'environnement exec. Les surcharges `env.PATH` sont rejetées pour l'exécution hôte. Le démon lui-même s'exécute toujours avec un `PATH` minimal :
    -   macOS : `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
    -   Linux : `/usr/local/bin`, `/usr/bin`, `/bin`
-   `host=sandbox` : exécute `sh -lc` (shell de connexion) à l'intérieur du conteneur, donc `/etc/profile` peut réinitialiser `PATH`. OpenClaw préfixe `env.PATH` après le chargement du profil via une variable d'environnement interne (pas d'interpolation shell) ; `tools.exec.pathPrepend` s'applique ici aussi.
-   `host=node` : seules les surcharges d'environnement non bloquées que vous transmettez sont envoyées au nœud. Les surcharges `env.PATH` sont rejetées pour l'exécution hôte et ignorées par les hôtes de nœud. Si vous avez besoin d'entrées PATH supplémentaires sur un nœud, configurez l'environnement du service hôte du nœud (systemd/launchd) ou installez les outils dans des emplacements standard.

Liaison de nœud par agent (utilisez l'index de la liste d'agents dans la configuration) :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Interface de contrôle : l'onglet Nodes inclut un petit panneau "Liaison de nœud Exec" pour les mêmes paramètres.

## Surcharges de session (/exec)

Utilisez `/exec` pour définir des valeurs par défaut **par session** pour `host`, `security`, `ask` et `node`. Envoyez `/exec` sans arguments pour afficher les valeurs actuelles. Exemple :

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Modèle d'autorisation

`/exec` n'est honoré que pour les **expéditeurs autorisés** (listes d'autorisation de canal/appariement plus `commands.useAccessGroups`). Il met à jour **uniquement l'état de la session** et n'écrit pas de configuration. Pour désactiver complètement exec, refusez-le via la politique d'outil (`tools.deny: ["exec"]` ou par agent). Les approbations hôtes s'appliquent toujours sauf si vous définissez explicitement `security=full` et `ask=off`.

## Approbations Exec (application compagnon / hôte de nœud)

Les agents sandboxés peuvent nécessiter une approbation par demande avant qu'`exec` ne s'exécute sur la passerelle ou l'hôte de nœud. Voir [Approbations Exec](./exec-approvals.md) pour la politique, la liste d'autorisation et le flux d'interface. Lorsque des approbations sont requises, l'outil exec retourne immédiatement avec `status: "approval-pending"` et un identifiant d'approbation. Une fois approuvée (ou refusée / expirée), la Gateway émet des événements système (`Exec finished` / `Exec denied`). Si la commande est toujours en cours d'exécution après `tools.exec.approvalRunningNoticeMs`, un seul avis `Exec running` est émis.

## Liste d'autorisation + binaires sûrs

L'application manuelle de la liste d'autorisation correspond **uniquement aux chemins de binaires résolus** (pas de correspondance sur le nom de base). Quand `security=allowlist`, les commandes shell sont automatiquement autorisées seulement si chaque segment du pipeline est autorisé ou est un binaire sûr. L'enchaînement (`;`, `&&`, `||`) et les redirections sont rejetés en mode liste d'autorisation à moins que chaque segment de premier niveau ne satisfasse la liste d'autorisation (y compris les binaires sûrs). Les redirections restent non prises en charge. `autoAllowSkills` est un chemin de commodité distinct dans les approbations exec. Ce n'est pas la même chose que les entrées manuelles de liste d'autorisation de chemin. Pour une confiance explicite stricte, gardez `autoAllowSkills` désactivé. Utilisez les deux contrôles pour des tâches différentes :

-   `tools.exec.safeBins` : petits filtres de flux en entrée standard uniquement.
-   `tools.exec.safeBinTrustedDirs` : répertoires de confiance explicites supplémentaires pour les chemins d'exécutables des binaires sûrs.
-   `tools.exec.safeBinProfiles` : politique d'argv explicite pour les binaires sûrs personnalisés.
-   liste d'autorisation : confiance explicite pour les chemins d'exécutables.

Ne traitez pas `safeBins` comme une liste d'autorisation générique, et n'ajoutez pas de binaires d'interpréteur/exécution (par exemple `python3`, `node`, `ruby`, `bash`). Si vous en avez besoin, utilisez des entrées explicites dans la liste d'autorisation et gardez les invites d'approbation activées. `openclaw security audit` avertit lorsque des entrées `safeBins` d'interpréteur/exécution manquent de profils explicites, et `openclaw doctor --fix` peut générer des entrées personnalisées manquantes dans `safeBinProfiles`. Pour les détails complets de la politique et des exemples, voir [Approbations Exec](./exec-approvals.md#safe-bins-stdin-only) et [Binaires sûrs versus liste d'autorisation](./exec-approvals.md#safe-bins-versus-allowlist).

## Exemples

Premier plan :

```json
{ "tool": "exec", "command": "ls -la" }
```

Arrière-plan + interrogation :

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Envoyer des touches (style tmux) :

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Soumettre (envoyer CR uniquement) :

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Coller (entre crochets par défaut) :

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (expérimental)

`apply_patch` est un sous-outil de `exec` pour des modifications structurées multi-fichiers. Activez-le explicitement :

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Notes :

-   Disponible uniquement pour les modèles OpenAI/OpenAI Codex.
-   La politique d'outil s'applique toujours ; `allow: ["exec"]` autorise implicitement `apply_patch`.
-   La configuration se trouve sous `tools.exec.applyPatch`.
-   `tools.exec.applyPatch.workspaceOnly` est par défaut `true` (contenu dans l'espace de travail). Définissez-le à `false` uniquement si vous souhaitez intentionnellement que `apply_patch` écrive/supprime en dehors du répertoire de l'espace de travail.

[Mode Élevé](./elevated.md)[Approbations Exec](./exec-approvals.md)