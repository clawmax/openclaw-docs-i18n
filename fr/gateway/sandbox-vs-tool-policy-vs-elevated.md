

  Sécurité et sandboxing

  
# Sandbox vs Politique d'outils vs Élevé

OpenClaw a trois contrôles liés (mais différents) :

1.  **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) décide **où les outils s'exécutent** (Docker vs hôte).
2.  **Politique d'outils** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) décide **quels outils sont disponibles/autorisés**.
3.  **Élevé** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) est une **trappe d'exécution uniquement (exec-only)** pour s'exécuter sur l'hôte lorsque vous êtes en sandbox.

## Débogage rapide

Utilisez l'inspecteur pour voir ce qu'OpenClaw fait *réellement* :

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Il affiche :

-   le mode/portée/accès au workspace effectif du sandbox
-   si la session est actuellement en sandbox (main vs non-main)
-   la liste d'autorisation/interdiction d'outils effective du sandbox (et si elle provient de l'agent/global/par défaut)
-   les portes (gates) du mode élevé et les chemins des clés de correction

## Sandbox : où les outils s'exécutent

Le sandboxing est contrôlé par `agents.defaults.sandbox.mode` :

-   `"off"` : tout s'exécute sur l'hôte.
-   `"non-main"` : seules les sessions non principales (non-main) sont en sandbox (situation courante de "surprise" pour les groupes/canaux).
-   `"all"` : tout est en sandbox.

Voir [Sandboxing](./sandboxing.md) pour la matrice complète (portée, montages du workspace, images).

### Montages de liaison (vérification rapide de sécurité)

-   `docker.binds` *perce* le système de fichiers du sandbox : tout ce que vous montez est visible à l'intérieur du conteneur avec le mode que vous définissez (`:ro` ou `:rw`).
-   Par défaut, c'est en lecture-écriture si vous omettez le mode ; préférez `:ro` pour les sources/secrets.
-   `scope: "shared"` ignore les montages par agent (seuls les montages globaux s'appliquent).
-   Monter `/var/run/docker.sock` donne effectivement le contrôle de l'hôte au sandbox ; ne faites cela qu'intentionnellement.
-   L'accès au workspace (`workspaceAccess: "ro"`/`"rw"`) est indépendant des modes de montage.

## Politique d'outils : quels outils existent/peuvent être appelés

Deux couches sont importantes :

-   **Profil d'outil** : `tools.profile` et `agents.list[].tools.profile` (liste d'autorisation de base)
-   **Profil d'outil par fournisseur** : `tools.byProvider[provider].profile` et `agents.list[].tools.byProvider[provider].profile`
-   **Politique d'outils globale/par agent** : `tools.allow`/`tools.deny` et `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **Politique d'outils par fournisseur** : `tools.byProvider[provider].allow/deny` et `agents.list[].tools.byProvider[provider].allow/deny`
-   **Politique d'outils du sandbox** (s'applique uniquement en sandbox) : `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` et `agents.list[].tools.sandbox.tools.*`

Règles générales :

-   `deny` l'emporte toujours.
-   Si `allow` n'est pas vide, tout le reste est considéré comme bloqué.
-   La politique d'outils est l'arrêt définitif : `/exec` ne peut pas contourner un outil `exec` interdit.
-   `/exec` ne modifie que les valeurs par défaut de session pour les expéditeurs autorisés ; il n'accorde pas d'accès aux outils. Les clés d'outils par fournisseur acceptent soit `provider` (ex. `google-antigravity`) soit `provider/model` (ex. `openai/gpt-5.2`).

### Groupes d'outils (raccourcis)

Les politiques d'outils (globales, par agent, sandbox) prennent en charge les entrées `group:*` qui se développent en plusieurs outils :

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

Groupes disponibles :

-   `group:runtime` : `exec`, `bash`, `process`
-   `group:fs` : `read`, `write`, `edit`, `apply_patch`
-   `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory` : `memory_search`, `memory_get`
-   `group:ui` : `browser`, `canvas`
-   `group:automation` : `cron`, `gateway`
-   `group:messaging` : `message`
-   `group:nodes` : `nodes`
-   `group:openclaw` : tous les outils intégrés d'OpenClaw (exclut les plugins de fournisseurs)

## Élevé : exécution "sur l'hôte" uniquement pour exec

Le mode élevé **n'accorde pas** d'outils supplémentaires ; il n'affecte que `exec`.

-   Si vous êtes en sandbox, `/elevated on` (ou `exec` avec `elevated: true`) s'exécute sur l'hôte (les approbations peuvent toujours s'appliquer).
-   Utilisez `/elevated full` pour ignorer les approbations d'exec pour la session.
-   Si vous exécutez déjà directement, le mode élevé est effectivement sans effet (toujours contrôlé par des portes).
-   Le mode élevé n'est **pas** limité à une compétence (skill-scoped) et ne **contourne pas** la liste d'autorisation/interdiction d'outils.
-   `/exec` est séparé du mode élevé. Il ajuste uniquement les valeurs par défaut d'exec par session pour les expéditeurs autorisés.

Portes de contrôle (Gates) :

-   Activation : `tools.elevated.enabled` (et optionnellement `agents.list[].tools.elevated.enabled`)
-   Listes d'autorisation d'expéditeurs : `tools.elevated.allowFrom.` (et optionnellement `agents.list[].tools.elevated.allowFrom.`)

Voir [Mode Élevé](../tools/elevated.md).

## Corrections courantes du "sandbox jail"

### "L'outil X est bloqué par la politique d'outils du sandbox"

Clés de correction (choisissez-en une) :

-   Désactivez le sandbox : `agents.defaults.sandbox.mode=off` (ou par agent `agents.list[].sandbox.mode=off`)
-   Autorisez l'outil dans le sandbox :
    -   retirez-le de `tools.sandbox.tools.deny` (ou par agent `agents.list[].tools.sandbox.tools.deny`)
    -   ou ajoutez-le à `tools.sandbox.tools.allow` (ou la liste d'autorisation par agent)

### "Je pensais que c'était main, pourquoi est-ce en sandbox ?"

En mode `"non-main"`, les clés de groupe/canal ne sont *pas* main. Utilisez la clé de session principale (affichée par `sandbox explain`) ou passez en mode `"off"`.

[Sandboxing](./sandboxing.md)[Protocole Gateway](./protocol.md)