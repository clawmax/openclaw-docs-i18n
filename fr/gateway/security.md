

  Sécurité et sandboxing

  
# Sécurité

> \[!WARNING\] **Modèle de confiance d'assistant personnel :** ce guide suppose une limite d'opérateur de confiance unique par passerelle (modèle utilisateur unique/assistant personnel). OpenClaw n'est **pas** une limite de sécurité multi-locataire hostile pour plusieurs utilisateurs adversariaux partageant un agent/passerelle. Si vous avez besoin d'une opération à confiance mixte ou avec des utilisateurs adversariaux, séparez les limites de confiance (passerelle + identifiants séparés, idéalement utilisateurs/hôtes OS séparés).

## Définir le périmètre d'abord : modèle de sécurité d'assistant personnel

Les conseils de sécurité d'OpenClaw supposent un déploiement en **assistant personnel** : une limite d'opérateur de confiance, potentiellement plusieurs agents.

-   Posture de sécurité prise en charge : un utilisateur/limite de confiance par passerelle (préférer un utilisateur/hôte/VPS OS par limite).
-   Ce n'est pas une limite de sécurité prise en charge : une passerelle/agent partagé utilisé par des utilisateurs mutuellement non fiables ou adversariaux.
-   Si une isolation d'utilisateurs adversariaux est requise, séparez par limite de confiance (passerelle + identifiants séparés, et idéalement utilisateurs/hôtes OS séparés).
-   Si plusieurs utilisateurs non fiables peuvent envoyer des messages à un agent avec outils activés, considérez qu'ils partagent la même autorité d'outil déléguée pour cet agent.

Cette page explique le durcissement **dans ce modèle**. Elle ne prétend pas à une isolation multi-locataire hostile sur une passerelle partagée.

## Vérification rapide : audit de sécurité openclaw

Voir aussi : [Vérification formelle (Modèles de sécurité)](../security/formal-verification.md) Exécutez ceci régulièrement (surtout après avoir modifié la configuration ou exposé des surfaces réseau) :

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

Il signale les erreurs courantes (Exposition de l'authentification de la passerelle, exposition du contrôle du navigateur, listes d'autorisation élevées, permissions du système de fichiers). OpenClaw est à la fois un produit et une expérience : vous connectez le comportement de modèles de pointe à de véritables surfaces de messagerie et de vrais outils. **Il n'existe pas de configuration "parfaitement sécurisée".** L'objectif est d'être délibéré concernant :

-   qui peut parler à votre bot
-   où le bot est autorisé à agir
-   ce que le bot peut toucher

Commencez avec le plus petit accès qui fonctionne encore, puis élargissez-le au fur et à mesure que vous gagnez en confiance.

## Hypothèse de déploiement (important)

OpenClaw suppose que l'hôte et la limite de configuration sont de confiance :

-   Si quelqu'un peut modifier l'état/la configuration de l'hôte de la passerelle (`~/.openclaw`, y compris `openclaw.json`), considérez-le comme un opérateur de confiance.
-   Exécuter une passerelle pour plusieurs opérateurs mutuellement non fiables/adversariaux est **une configuration non recommandée**.
-   Pour les équipes à confiance mixte, séparez les limites de confiance avec des passerelles distinctes (ou au minimum des utilisateurs/hôtes OS séparés).
-   OpenClaw peut exécuter plusieurs instances de passerelle sur une même machine, mais les opérations recommandées favorisent une séparation claire des limites de confiance.
-   Par défaut recommandé : un utilisateur par machine/hôte (ou VPS), une passerelle pour cet utilisateur, et un ou plusieurs agents dans cette passerelle.
-   Si plusieurs utilisateurs veulent OpenClaw, utilisez un VPS/hôte par utilisateur.

### Conséquence pratique (limite de confiance de l'opérateur)

À l'intérieur d'une instance de passerelle, l'accès d'opérateur authentifié est un rôle de plan de contrôle de confiance, pas un rôle de locataire par utilisateur.

-   Les opérateurs avec un accès en lecture/plan de contrôle peuvent inspecter les métadonnées/l'historique des sessions de la passerelle par conception.
-   Les identifiants de session (`sessionKey`, IDs de session, labels) sont des sélecteurs de routage, pas des jetons d'autorisation.
-   Exemple : s'attendre à une isolation par opérateur pour des méthodes comme `sessions.list`, `sessions.preview`, ou `chat.history` est en dehors de ce modèle.
-   Si vous avez besoin d'une isolation d'utilisateurs adversariaux, exécutez des passerelles séparées par limite de confiance.
-   Plusieurs passerelles sur une même machine sont techniquement possibles, mais ce n'est pas la ligne de base recommandée pour l'isolation multi-utilisateur.

## Modèle d'assistant personnel (pas un bus multi-locataire)

OpenClaw est conçu comme un modèle de sécurité d'assistant personnel : une limite d'opérateur de confiance, potentiellement plusieurs agents.

-   Si plusieurs personnes peuvent envoyer des messages à un agent avec outils activés, chacune d'elles peut orienter le même ensemble de permissions.
-   L'isolation de session/mémoire par utilisateur aide à la confidentialité, mais ne transforme pas un agent partagé en une autorisation d'hôte par utilisateur.
-   Si les utilisateurs peuvent être adversariaux les uns envers les autres, exécutez des passerelles séparées (ou des utilisateurs/hôtes OS séparés) par limite de confiance.

### Espace de travail Slack partagé : risque réel

Si "tout le monde dans Slack peut envoyer un message au bot", le risque principal est l'autorité d'outil déléguée :

-   tout expéditeur autorisé peut induire des appels d'outils (`exec`, navigateur, outils réseau/fichiers) dans le cadre de la politique de l'agent ;
-   l'injection de prompt/contenu d'un expéditeur peut provoquer des actions affectant l'état partagé, les appareils ou les sorties ;
-   si un agent partagé a des identifiants/fichiers sensibles, tout expéditeur autorisé peut potentiellement provoquer une exfiltration via l'utilisation d'outils.

Utilisez des agents/passerelles séparés avec des outils minimaux pour les flux de travail d'équipe ; gardez les agents de données personnelles privés.

### Agent partagé en entreprise : modèle acceptable

Ceci est acceptable lorsque tous ceux qui utilisent cet agent sont dans la même limite de confiance (par exemple une équipe d'entreprise) et que l'agent est strictement limité au domaine professionnel.

-   exécutez-le sur une machine/VM/conteneur dédié ;
-   utilisez un utilisateur OS dédié + un navigateur/profil/comptes dédiés pour ce runtime ;
-   ne connectez pas ce runtime à des comptes Apple/Google personnels ou à des profils de navigateur/gestionnaire de mots de passe personnels.

Si vous mélangez des identités personnelles et professionnelles sur le même runtime, vous effacez la séparation et augmentez le risque d'exposition de données personnelles.

## Concept de confiance de la passerelle et du nœud

Considérez la passerelle et le nœud comme un domaine de confiance d'opérateur unique, avec des rôles différents :

-   La **passerelle** est le plan de contrôle et la surface de politique (`gateway.auth`, politique d'outils, routage).
-   Le **nœud** est la surface d'exécution distante jumelée à cette passerelle (commandes, actions sur appareil, capacités locales à l'hôte).
-   Un appelant authentifié auprès de la passerelle est de confiance au niveau de la passerelle. Après le jumelage, les actions du nœud sont des actions d'opérateur de confiance sur ce nœud.
-   `sessionKey` est une sélection de routage/contexte, pas une authentification par utilisateur.
-   Les approbations d'exécution (liste d'autorisation + demande) sont des garde-fous pour l'intention de l'opérateur, pas une isolation multi-locataire hostile.

Si vous avez besoin d'une isolation d'utilisateurs hostiles, séparez les limites de confiance par utilisateur/hôte OS et exécutez des passerelles séparées.

## Matrice des limites de confiance

Utilisez ceci comme modèle rapide pour évaluer les risques :

| Limite ou contrôle | Ce que cela signifie | Mauvaise interprétation courante |
| --- | --- | --- |
| `gateway.auth` (jeton/mot de passe/auth d'appareil) | Authentifie les appelants auprès des API de la passerelle | "Nécessite des signatures par message sur chaque trame pour être sécurisé" |
| `sessionKey` | Clé de routage pour la sélection de contexte/session | "La clé de session est une limite d'authentification utilisateur" |
| Garde-fous de prompt/contenu | Réduisent le risque d'abus du modèle | "L'injection de prompt seule prouve un contournement d'authentification" |
| `canvas.eval` / évaluation du navigateur | Capacité d'opérateur intentionnelle lorsqu'elle est activée | "Toute primitive d'évaluation JS est automatiquement une vulnérabilité dans ce modèle de confiance" |
| Shell TUI local `!` | Exécution locale déclenchée explicitement par l'opérateur | "La commande de shell local de commodité est une injection distante" |
| Jumelage de nœud et commandes de nœud | Exécution distante au niveau opérateur sur les appareils jumelés | "Le contrôle d'appareil distant doit être traité par défaut comme un accès utilisateur non fiable" |

## Non vulnérabilités par conception

Ces modèles sont couramment signalés et sont généralement clôturés sans action à moins qu'un contournement réel de limite soit démontré :

-   Chaînes d'injection de prompt uniquement sans contournement de politique/auth/sandbox.
-   Affirmations qui supposent une opération multi-locataire hostile sur un hôte/config partagé.
-   Affirmations qui classent un accès en lecture normal de l'opérateur (par exemple `sessions.list`/`sessions.preview`/`chat.history`) comme IDOR dans une configuration de passerelle partagée.
-   Constatations de déploiement localhost uniquement (par exemple HSTS sur une passerelle en boucle locale uniquement).
-   Constatations de signature de webhook entrant Discord pour des chemins entrants qui n'existent pas dans ce dépôt.
-   Constatations de "manque d'autorisation par utilisateur" qui traitent `sessionKey` comme un jeton d'authentification.

## Liste de contrôle préalable pour les chercheurs

Avant d'ouvrir un GHSA, vérifiez tous ces points :

1.  La reproduction fonctionne toujours sur la dernière branche `main` ou la dernière version.
2.  Le rapport inclut le chemin de code exact (`fichier`, fonction, plage de lignes) et la version/commit testé.
3.  L'impact franchit une limite de confiance documentée (pas seulement une injection de prompt).
4.  L'affirmation n'est pas listée dans [Hors périmètre](https://github.com/openclaw/openclaw/blob/main/SECURITY.md#out-of-scope).
5.  Les avis existants ont été vérifiés pour les doublons (réutilisez le GHSA canonique le cas échéant).
6.  Les hypothèses de déploiement sont explicites (boucle locale/local vs exposé, opérateurs de confiance vs non fiables).

## Ligne de base durcie en 60 secondes

Utilisez cette ligne de base d'abord, puis réactivez sélectivement les outils par agent de confiance :

```json
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "replace-with-long-random-token" },
  },
  session: {
    dmScope: "per-channel-peer",
  },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn", "sessions_send"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false },
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

Cela garde la passerelle en local uniquement, isole les messages directs et désactive les outils de plan de contrôle/runtime par défaut.

## Règle rapide pour les boîtes de réception partagées

Si plus d'une personne peut envoyer un message direct à votre bot :

-   Définissez `session.dmScope: "per-channel-peer"` (ou `"per-account-channel-peer"` pour les canaux multi-comptes).
-   Gardez `dmPolicy: "pairing"` ou des listes d'autorisation strictes.
-   Ne combinez jamais les messages directs partagés avec un large accès aux outils.
-   Cela durcit les boîtes de réception coopératives/partagées, mais n'est pas conçu comme une isolation de co-locataires hostiles lorsque les utilisateurs partagent un accès en écriture à l'hôte/config.

### Ce que l'audit vérifie (niveau élevé)

-   **Accès entrant** (politiques de messages directs, politiques de groupe, listes d'autorisation) : des inconnus peuvent-ils déclencher le bot ?
-   **Rayon d'impact des outils** (outils élevés + salles ouvertes) : une injection de prompt peut-elle se transformer en actions shell/fichier/réseau ?
-   **Exposition réseau** (liaison/auth de la passerelle, Tailscale Serve/Funnel, jetons d'authentification faibles/courts).
-   **Exposition du contrôle du navigateur** (nœuds distants, ports de relais, points de terminaison CDP distants).
-   **Hygiène du disque local** (permissions, liens symboliques, inclusions de configuration, chemins de "dossier synchronisé").
-   **Plugins** (les extensions existent sans liste d'autorisation explicite).
-   **Dérive/mauvaise configuration de la politique** (paramètres Docker de sandbox configurés mais mode sandbox désactivé ; modèles `gateway.nodes.denyCommands` inefficaces car la correspondance est exactement sur le nom de commande uniquement (par exemple `system.run`) et n'inspecte pas le texte du shell ; entrées `gateway.nodes.allowCommands` dangereuses ; `tools.profile="minimal"` global remplacé par des profils par agent ; outils d'extension de plugin accessibles sous une politique d'outils permissive).
-   **Dérive des attentes du runtime** (par exemple `tools.exec.host="sandbox"` alors que le mode sandbox est désactivé, ce qui s'exécute directement sur l'hôte de la passerelle).
-   **Hygiène des modèles** (avertit lorsque les modèles configurés semblent obsolètes ; pas un blocage dur).

Si vous exécutez `--deep`, OpenClaw tente également une sonde en direct de la passerelle avec les moyens du bord.

## Carte de stockage des identifiants

Utilisez ceci lors de l'audit d'accès ou pour décider quoi sauvegarder :

-   **WhatsApp** : `~/.openclaw/credentials/whatsapp//creds.json`
-   **Jeton de bot Telegram** : config/env ou `channels.telegram.tokenFile`
-   **Jeton de bot Discord** : config/env ou SecretRef (fournisseurs env/fichier/exec)
-   **Jetons Slack** : config/env (`channels.slack.*`)
-   **Listes d'autorisation de jumelage** :
    -   `~/.openclaw/credentials/-allowFrom.json` (compte par défaut)
    -   `~/.openclaw/credentials/--allowFrom.json` (comptes non par défaut)
-   **Profils d'authentification de modèle** : `~/.openclaw/agents//agent/auth-profiles.json`
-   **Charge utile de secrets basée sur fichier (optionnelle)** : `~/.openclaw/secrets.json`
-   **Import OAuth hérité** : `~/.openclaw/credentials/oauth.json`

## Liste de contrôle d'audit de sécurité

Lorsque l'audit affiche des constatations, traitez-les par ordre de priorité :

1.  **Tout ce qui est "ouvert" + outils activés** : verrouillez d'abord les messages directs/groupes (jumelage/listes d'autorisation), puis resserrez la politique d'outils/sandboxing.
2.  **Exposition réseau publique** (liaison LAN, Funnel, auth manquante) : corrigez immédiatement.
3.  **Exposition distante du contrôle du navigateur** : traitez-le comme un accès opérateur (tailnet uniquement, jumelez les nœuds délibérément, évitez l'exposition publique).
4.  **Permissions** : assurez-vous que l'état/config/identifiants/auth ne sont pas lisibles par le groupe/le monde.
5.  **Plugins/extensions** : chargez uniquement ceux que vous faites explicitement confiance.
6.  **Choix du modèle** : préférez des modèles modernes et renforcés par instruction pour tout bot avec des outils.

## Glossaire de l'audit de sécurité

Valeurs `checkId` à fort signal que vous verrez très probablement dans les déploiements réels (non exhaustif) :

| `checkId` | Sévérité | Pourquoi c'est important | Clé/chemin de correction principal | Correction auto |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | critique | D'autres utilisateurs/processus peuvent modifier l'état complet d'OpenClaw | permissions du système de fichiers sur `~/.openclaw` | oui |
| `fs.config.perms_writable` | critique | D'autres peuvent changer la politique d'authentification/outils/config | permissions du système de fichiers sur `~/.openclaw/openclaw.json` | oui |
| `fs.config.perms_world_readable` | critique | La configuration peut exposer des jetons/paramètres | permissions du système de fichiers sur le fichier de config | oui |
| `gateway.bind_no_auth` | critique | Liaison distante sans secret partagé | `gateway.bind`, `gateway.auth.*` | non |
| `gateway.loopback_no_auth` | critique | La boucle locale avec proxy inverse peut devenir non authentifiée | `gateway.auth.*`, configuration du proxy | non |
| `gateway.http.no_auth` | avert/critique | Les API HTTP de la passerelle sont accessibles avec `auth.mode="none"` | `gateway.auth.mode`, `gateway.http.endpoints.*` | non |
| `gateway.tools_invoke_http.dangerous_allow` | avert/critique | Réactive les outils dangereux via l'API HTTP | `gateway.tools.allow` | non |
| `gateway.nodes.allow_commands_dangerous` | avert/critique | Active les commandes de nœud à fort impact (caméra/écran/contacts/calendrier/SMS) | `gateway.nodes.allowCommands` | non |
| `gateway.tailscale_funnel` | critique | Exposition à l'internet public | `gateway.tailscale.mode` | non |
| `gateway.control_ui.allowed_origins_required` | critique | Interface de contrôle non en boucle locale sans liste d'autorisation d'origine de navigateur explicite | `gateway.controlUi.allowedOrigins` | non |
| `gateway.control_ui.host_header_origin_fallback` | avert/critique | Active la retombée sur l'origine de l'en-tête Host (dégradation du durcissement contre le rebond DNS) | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | non |
| `gateway.control_ui.insecure_auth` | avert | Option de compatibilité d'authentification non sécurisée activée | `gateway.controlUi.allowInsecureAuth` | non |
| `gateway.control_ui.device_auth_disabled` | critique | Désactive la vérification d'identité de l'appareil | `gateway.controlUi.dangerouslyDisableDeviceAuth` | non |
| `gateway.real_ip_fallback_enabled` | avert/critique | Faire confiance à la retombée `X-Real-IP` peut permettre l'usurpation d'IP source via une mauvaise configuration de proxy | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | non |
| `discovery.mdns_full_mode` | avert/critique | Le mode complet mDNS diffuse les métadonnées `cliPath`/`sshPort` sur le réseau local | `discovery.mdns.mode`, `gateway.bind` | non |
| `config.insecure_or_dangerous_flags` | avert | Tout indicateur de débogage non sécurisé/dangereux activé | plusieurs clés (voir le détail de la constatation) | non |
| `hooks.token_too_short` | avert | Force brute plus facile sur l'entrée des hooks | `hooks.token` | non |
| `hooks.request_session_key_enabled` | avert/critique | L'appelant externe peut choisir sessionKey | `hooks.allowRequestSessionKey` | non |
| `hooks.request_session_key_prefixes_missing` | avert/critique | Aucune limite sur les formes de clé de session externe | `hooks.allowedSessionKeyPrefixes` | non |
| `logging.redact_off` | avert | Les valeurs sensibles fuient dans les logs/statut | `logging.redactSensitive` | oui |
| `sandbox.docker_config_mode_off` | avert | Configuration Docker de sandbox présente mais inactive | `agents.*.sandbox.mode` | non |
| `sandbox.dangerous_network_mode` | critique | Le réseau Docker du sandbox utilise le mode `host` ou `container:*` (jonction d'espace de noms) | `agents.*.sandbox.docker.network` | non |
| `tools.exec.host_sandbox_no_sandbox_defaults` | avert | `exec host=sandbox` se résout en exécution sur l'hôte lorsque le sandbox est désactivé | `tools.exec.host`, `agents.defaults.sandbox.mode` | non |
| `tools.exec.host_sandbox_no_sandbox_agents` | avert | `exec host=sandbox` par agent se résout en exécution sur l'hôte lorsque le sandbox est désactivé | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | non |
| `tools.exec.safe_bins_interpreter_unprofiled` | avert | Les binaires d'interpréteur/runtime dans `safeBins` sans profils explicites élargissent le risque d'exécution | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | non |
| `skills.workspace.symlink_escape` | avert | Le workspace `skills/**/SKILL.md` résout en dehors de la racine du workspace (dérive de chaîne de liens symboliques) | état du système de fichiers `skills/**` du workspace | non |
| `security.exposure.open_groups_with_elevated` | critique | Groupes ouverts + outils élevés créent des chemins d'injection de prompt à fort impact | `channels.*.groupPolicy`, `tools.elevated.*` | non |
| `security.exposure.open_groups_with_runtime_or_fs` | critique/avert | Les groupes ouverts peuvent atteindre les outils de commande/fichier sans garde-fous de sandbox/workspace | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | non |
| `security.trust_model.multi_user_heuristic` | avert | La configuration semble multi-utilisateur alors que le modèle de confiance de la passerelle est assistant personnel | séparez les limites de confiance, ou durcissez l'utilisateur partagé (`sandbox.mode`, restriction/portée des outils) | non |
| `tools.profile_minimal_overridden` | avert | Les remplacements d'agent contournent le profil minimal global | `agents.list[].tools.profile` | non |
| `plugins.tools_reachable_permissive_policy` | avert | Outils d'extension accessibles dans des contextes permissifs | `tools.profile` + autorisation/interdiction d'outils | non |
| `models.small_params` | critique/info | Petits modèles + surfaces d'outils non sécurisées augmentent le risque d'injection | choix du modèle + politique de sandbox/outils | non |

## Interface de contrôle sur HTTP

L'interface de contrôle a besoin d'un **contexte sécurisé** (HTTPS ou localhost) pour générer l'identité de l'appareil. `gateway.controlUi.allowInsecureAuth` ne contourne **pas** les vérifications de contexte sécurisé, d'identité de l'appareil ou de jumelage d'appareil. Préférez HTTPS (Tailscale Serve) ou ouvrez l'interface sur `127.0.0.1`. Pour les scénarios de secours uniquement, `gateway.controlUi.dangerouslyDisableDeviceAuth` désactive complètement les vérifications d'identité de l'appareil. Il s'agit d'une dégradation sévère de la sécurité ; gardez-la désactivée sauf si vous déboguez activement et pouvez revenir rapidement. `openclaw security audit` avertit lorsque ce paramètre est activé.

## Résumé des indicateurs non sécurisés ou dangereux

`openclaw security audit` inclut `config.insecure_or_dangerous_flags` lorsque des commutateurs de débogage non sécurisés/dangereux connus sont activés. Cette vérification agrège actuellement :

-   `gateway.controlUi.allowInsecureAuth=true`
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
-   `hooks.gmail.allowUnsafeExternalContent=true`
-   `hooks.mappings[].allowUnsafeExternalContent=true`
-   `tools.exec.applyPatch.workspaceOnly=false`

Clés de configuration `dangerous*` / `dangerously*` complètes définies dans le schéma de configuration d'OpenClaw :

-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth`
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
-   `channels.discord.dangerouslyAllowNameMatching`
-   `channels.discord.accounts..dangerouslyAllowNameMatching`
-   `channels.slack.dangerouslyAllowNameMatching`
-   `channels.slack.accounts..dangerouslyAllowNameMatching`
-   `channels.googlechat.dangerouslyAllowNameMatching`
-   `channels.googlechat.accounts..dangerouslyAllowNameMatching`
-   `channels.msteams.dangerouslyAllowNameMatching`
-   `channels.irc.dangerouslyAllowNameMatching` (canal d'extension)
-   `channels.irc.accounts..dangerouslyAllowNameMatching` (canal d'extension)
-   `channels.mattermost.dangerouslyAllowNameMatching` (canal d'extension)
-   `channels.mattermost.accounts..dangerouslyAllowNameMatching` (canal d'extension)
-   `agents.defaults.sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.defaults.sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin`
-   `agents.list[].sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.list[].sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.list[].sandbox.docker.dangerouslyAllowContainerNamespaceJoin`

## Configuration du proxy inverse

Si vous exécutez la passerelle derrière un proxy inverse (nginx, Caddy, Traefik, etc.), vous devez configurer `gateway.trustedProxies` pour une détection correcte de l'IP client. Lorsque la passerelle détecte des en-têtes de proxy provenant d'une adresse qui n'est **pas** dans `trustedProxies`, elle ne traitera **pas** les connexions comme des clients locaux. Si l'authentification de la passerelle est désactivée, ces connexions sont rejetées. Cela empêche un contournement d'authentification où les connexions proxy apparaîtraient autrement comme provenant de localhost et recevraient une confiance automatique.

```
gateway:
  trustedProxies:
    - "127.0.0.1" # si votre proxy s'exécute sur localhost
  # Optionnel. Par défaut false.
  # Activez uniquement si votre proxy ne peut pas fournir X-Forwarded-For.
  allowRealIpFallback: false
  auth:
    mode: password
    password: ${OPENCLAW_GATEWAY_PASSWORD}
```

Lorsque `trustedProxies` est configuré, la passerelle utilise `X-Forwarded-For` pour déterminer l'IP client. `X-Real-IP` est ignoré par défaut sauf si `gateway.allowRealIpFallback: true` est explicitement défini. Comportement de proxy inverse correct (écraser les en-têtes de transfert entrants) :

```bash
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
```

Comportement de proxy inverse incorrect (ajouter/conserver les en-têtes de transfert non fiables) :

```bash
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## Notes sur HSTS et l'origine

-   La passerelle OpenClaw est d'abord locale/en boucle locale. Si vous terminez TLS sur un proxy inverse, définissez HSTS sur le domaine HTTPS côté proxy là-bas.
-   Si la passerelle elle-même termine HTTPS, vous pouvez définir `gateway.http.securityHeaders.strictTransportSecurity` pour émettre l'en-tête HSTS depuis les réponses d'OpenClaw.
-   Des conseils de déploiement détaillés sont dans [Authentification par proxy de confiance](./trusted-proxy-auth.md#tls-termination-and-hsts).
-   Pour les déploiements d'interface de contrôle non en boucle locale, `gateway.controlUi.allowedOrigins` est requis par défaut.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` active le mode de retombée sur l'origine de l'en-tête Host ; traitez-le comme une politique dangereuse sélectionnée par l'opérateur.
-   Traitez le rebond DNS et le comportement de l'en-tête Host du proxy comme des préoccupations de durcissement de déploiement ; gardez `trustedProxies` restreint et évitez d'exposer la passerelle directement à l'internet public.

## Les journaux de session locaux vivent sur le disque

OpenClaw stocke les transcriptions de session sur le disque sous `~/.openclaw/agents//sessions/*.jsonl`. Ceci est requis pour la continuité des sessions et (optionnellement) l'indexation de la mémoire des sessions, mais cela signifie aussi que **tout processus/utilisateur avec un accès au système de fichiers peut lire ces journaux**. Traitez l'accès au disque comme la limite de confiance et verrouillez les permissions sur `~/.openclaw` (voir la section d'audit ci-dessous). Si vous avez besoin d'une isolation plus forte entre les agents, exécutez-les sous des utilisateurs OS ou des hôtes séparés.

## Exécution de nœud (system.run)

Si un nœud macOS est jumelé, la passerelle peut invoquer `system.run` sur ce nœud. Il s'agit d'une **exécution de code à distance** sur le Mac :

-   Nécessite un jumelage de nœud (approbation + jeton).
-   Contrôlé sur le Mac via **Paramètres → Approbations d'exécution** (sécurité + demande + liste d'autorisation).
-   Si vous ne voulez pas d'exécution distante, définissez la sécurité sur **deny** et supprimez le jumelage de nœud pour ce Mac.

## Compétences dynamiques (surveillance / nœuds distants)

OpenClaw peut rafraîchir la liste des