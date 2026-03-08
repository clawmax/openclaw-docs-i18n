

  Configuration et opérations

  
# Doctor

`openclaw doctor` est l'outil de réparation et de migration pour OpenClaw. Il corrige les configurations/états obsolètes, vérifie l'état de santé et fournit des étapes de réparation actionnables.

## Démarrage rapide

```bash
openclaw doctor
```

### Sans interface / automatisation

```bash
openclaw doctor --yes
```

Accepter les valeurs par défaut sans invite (y compris les étapes de redémarrage/service/réparation du bac à sable le cas échéant).

```bash
openclaw doctor --repair
```

Appliquer les réparations recommandées sans invite (réparations + redémarrages lorsque c'est sûr).

```bash
openclaw doctor --repair --force
```

Appliquer également les réparations agressives (écrase les configurations personnalisées du superviseur).

```bash
openclaw doctor --non-interactive
```

Exécuter sans invites et n'appliquer que les migrations sûres (normalisation de configuration + déplacements d'état sur disque). Ignore les actions de redémarrage/service/bac à sable qui nécessitent une confirmation humaine. Les migrations d'état hérité s'exécutent automatiquement lorsqu'elles sont détectées.

```bash
openclaw doctor --deep
```

Analyser les services système pour des installations supplémentaires de la passerelle (launchd/systemd/schtasks). Si vous souhaitez examiner les changements avant de les écrire, ouvrez d'abord le fichier de configuration :

```bash
cat ~/.openclaw/openclaw.json
```

## Ce qu'il fait (résumé)

-   Mise à jour facultative pré-vol pour les installations git (interactif uniquement).
-   Vérification de fraîcheur du protocole UI (reconstruit l'interface de contrôle lorsque le schéma de protocole est plus récent).
-   Vérification d'état de santé + invite de redémarrage.
-   Résumé de l'état des compétences (éligibles/manquantes/bloquées).
-   Normalisation de configuration pour les valeurs héritées.
-   Avertissements concernant les surcharges de fournisseur OpenCode Zen (`models.providers.opencode`).
-   Migration d'état hérité sur disque (sessions/répertoire agent/auth WhatsApp).
-   Vérifications d'intégrité et de permissions de l'état (sessions, transcriptions, répertoire d'état).
-   Vérifications des permissions du fichier de configuration (chmod 600) lors d'une exécution locale.
-   État de santé de l'authentification des modèles : vérifie l'expiration OAuth, peut rafraîchir les jetons expirants, et signale les états de refroidissement/désactivés des profils d'authentification.
-   Détection de répertoire d'espace de travail supplémentaire (`~/openclaw`).
-   Réparation de l'image du bac à sable lorsque le bac à sable est activé.
-   Migration de service hérité et détection de passerelle supplémentaire.
-   Vérifications d'exécution de la passerelle (service installé mais non en cours d'exécution ; étiquette launchd en cache).
-   Avertissements sur l'état des canaux (sondés depuis la passerelle en cours d'exécution).
-   Audit de configuration du superviseur (launchd/systemd/schtasks) avec réparation facultative.
-   Vérifications des bonnes pratiques d'exécution de la passerelle (Node vs Bun, chemins des gestionnaires de version).
-   Diagnostics de collision de port de la passerelle (par défaut `18789`).
-   Avertissements de sécurité pour les politiques de DM ouvertes.
-   Vérifications d'authentification de la passerelle pour le mode jeton local (propose la génération d'un jeton lorsqu'aucune source de jeton n'existe ; n'écrase pas les configurations SecretRef de jeton).
-   Vérification du linger systemd sur Linux.
-   Vérifications d'installation depuis la source (incompatibilité de l'espace de travail pnpm, ressources UI manquantes, binaire tsx manquant).
-   Écrit la configuration mise à jour + les métadonnées de l'assistant.

## Comportement détaillé et justification

### 0) Mise à jour facultative (installations git)

S'il s'agit d'un dépôt git et que doctor s'exécute de manière interactive, il propose de mettre à jour (fetch/rebase/build) avant d'exécuter doctor.

### 1) Normalisation de configuration

Si la configuration contient des formes de valeurs héritées (par exemple `messages.ackReaction` sans une surcharge spécifique au canal), doctor les normalise dans le schéma actuel.

### 2) Migrations de clés de configuration héritées

Lorsque la configuration contient des clés obsolètes, les autres commandes refusent de s'exécuter et vous demandent d'exécuter `openclaw doctor`. Doctor va :

-   Expliquer quelles clés héritées ont été trouvées.
-   Montrer la migration qu'il a appliquée.
-   Réécrire `~/.openclaw/openclaw.json` avec le schéma mis à jour.

La Passerelle exécute également automatiquement les migrations de doctor au démarrage lorsqu'elle détecte un format de configuration hérité, de sorte que les configurations obsolètes sont réparées sans intervention manuelle. Migrations actuelles :

-   `routing.allowFrom` → `channels.whatsapp.allowFrom`
-   `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
-   `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
-   `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
-   `routing.queue` → `messages.queue`
-   `routing.bindings` → `bindings` de premier niveau
-   `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
-   `routing.agentToAgent` → `tools.agentToAgent`
-   `routing.transcribeAudio` → `tools.media.audio.models`
-   `bindings[].match.accountID` → `bindings[].match.accountId`
-   Pour les canaux avec des `accounts` nommés mais manquant `accounts.default`, déplacer les valeurs de canal de premier niveau à compte unique dans `channels..accounts.default` lorsqu'elles sont présentes
-   `identity` → `agents.list[].identity`
-   `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
-   `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
-   `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

Les avertissements de doctor incluent également des conseils sur le compte par défaut pour les canaux multi-comptes :

-   Si deux entrées `channels..accounts` ou plus sont configurées sans `channels..defaultAccount` ou `accounts.default`, doctor avertit que le routage de secours peut choisir un compte inattendu.
-   Si `channels..defaultAccount` est défini sur un ID de compte inconnu, doctor avertit et liste les ID de compte configurés.

### 2b) Surcharges de fournisseur OpenCode Zen

Si vous avez ajouté manuellement `models.providers.opencode` (ou `opencode-zen`), cela remplace le catalogue OpenCode Zen intégré de `@mariozechner/pi-ai`. Cela peut forcer chaque modèle à utiliser une seule API ou annuler les coûts. Doctor avertit afin que vous puissiez supprimer la surcharge et restaurer le routage API et les coûts par modèle.

### 3) Migrations d'état hérité (disposition du disque)

Doctor peut migrer les anciennes dispositions sur disque vers la structure actuelle :

-   Magasin de sessions + transcriptions :
    -   de `~/.openclaw/sessions/` vers `~/.openclaw/agents//sessions/`
-   Répertoire agent :
    -   de `~/.openclaw/agent/` vers `~/.openclaw/agents//agent/`
-   État d'authentification WhatsApp (Baileys) :
    -   des anciens `~/.openclaw/credentials/*.json` (sauf `oauth.json`)
    -   vers `~/.openclaw/credentials/whatsapp//...` (ID de compte par défaut : `default`)

Ces migrations sont de type "best-effort" et idempotentes ; doctor émettra des avertissements lorsqu'il laissera des dossiers hérités en tant que sauvegardes. La Passerelle/CLI migre également automatiquement les sessions héritées et le répertoire agent au démarrage, de sorte que l'historique/auth/modèles atterrissent dans le chemin par agent sans exécution manuelle de doctor. L'authentification WhatsApp n'est intentionnellement migrée que via `openclaw doctor`.

### 4) Vérifications d'intégrité de l'état (persistance des sessions, routage et sécurité)

Le répertoire d'état est le tronc cérébral opérationnel. S'il disparaît, vous perdez les sessions, les identifiants, les journaux et la configuration (sauf si vous avez des sauvegardes ailleurs). Doctor vérifie :

-   **Répertoire d'état manquant** : avertit d'une perte d'état catastrophique, invite à recréer le répertoire et rappelle qu'il ne peut pas récupérer les données manquantes.
-   **Permissions du répertoire d'état** : vérifie la possibilité d'écriture ; propose de réparer les permissions (et émet un indice `chown` lorsqu'un décalage propriétaire/groupe est détecté).
-   **Répertoire d'état synchronisé avec le cloud macOS** : avertit lorsque l'état se résout sous iCloud Drive (`~/Library/Mobile Documents/com~apple~CloudDocs/...`) ou `~/Library/CloudStorage/...` car les chemins synchronisés peuvent causer des E/S plus lentes et des conflits de verrouillage/synchronisation.
-   **Répertoire d'état sur SD ou eMMC Linux** : avertit lorsque l'état se résout vers une source de montage `mmcblk*`, car les E/S aléatoires sur SD ou eMMC peuvent être plus lentes et s'user plus rapidement sous les écritures de sessions et d'identifiants.
-   **Répertoires de sessions manquants** : `sessions/` et le répertoire de stockage des sessions sont requis pour persister l'historique et éviter les plantages `ENOENT`.
-   **Incompatibilité de transcription** : avertit lorsque des entrées de session récentes ont des fichiers de transcription manquants.
-   **Session principale "JSONL à 1 ligne"** : signale lorsque la transcription principale n'a qu'une seule ligne (l'historique ne s'accumule pas).
-   **Répertoires d'état multiples** : avertit lorsque plusieurs dossiers `~/.openclaw` existent dans différents répertoires personnels ou lorsque `OPENCLAW_STATE_DIR` pointe ailleurs (l'historique peut être divisé entre les installations).
-   **Rappel du mode distant** : si `gateway.mode=remote`, doctor rappelle de l'exécuter sur l'hôte distant (l'état y réside).
-   **Permissions du fichier de configuration** : avertit si `~/.openclaw/openclaw.json` est lisible par le groupe/le monde et propose de le restreindre à `600`.

### 5) État de santé de l'authentification des modèles (expiration OAuth)

Doctor inspecte les profils OAuth dans le magasin d'authentification, avertit lorsque les jetons expirent/sont expirés et peut les rafraîchir en toute sécurité. Si le profil Anthropic Claude Code est obsolète, il suggère d'exécuter `claude setup-token` (ou de coller un setup-token). Les invites de rafraîchissement n'apparaissent qu'en mode interactif (TTY) ; `--non-interactive` ignore les tentatives de rafraîchissement. Doctor signale également les profils d'authentification temporairement inutilisables en raison de :

-   courts refroidissements (limites de débit/timeouts/échecs d'authentification)
-   désactivations plus longues (problèmes de facturation/crédit)

### 6) Validation du modèle des hooks

Si `hooks.gmail.model` est défini, doctor valide la référence du modèle par rapport au catalogue et à la liste autorisée et avertit lorsqu'elle ne se résoudra pas ou est interdite.

### 7) Réparation de l'image du bac à sable

Lorsque le bac à sable est activé, doctor vérifie les images Docker et propose de les construire ou de passer à des noms hérités si l'image actuelle est manquante.

### 8) Migrations de service de la passerelle et conseils de nettoyage

Doctor détecte les services de passerelle hérités (launchd/systemd/schtasks) et propose de les supprimer et d'installer le service OpenClaw en utilisant le port de passerelle actuel. Il peut également analyser les services supplémentaires de type passerelle et afficher des conseils de nettoyage. Les services de passerelle OpenClaw nommés par profil sont considérés comme de première classe et ne sont pas signalés comme "supplémentaires".

### 9) Avertissements de sécurité

Doctor émet des avertissements lorsqu'un fournisseur est ouvert aux DM sans liste autorisée, ou lorsqu'une politique est configurée de manière dangereuse.

### 10) Linger systemd (Linux)

S'il s'exécute en tant que service utilisateur systemd, doctor s'assure que le linger est activé pour que la passerelle reste active après la déconnexion.

### 11) État des compétences

Doctor affiche un résumé rapide des compétences éligibles/manquantes/bloquées pour l'espace de travail actuel.

### 12) Vérifications d'authentification de la passerelle (jeton local)

Doctor vérifie la préparation de l'authentification par jeton local de la passerelle.

-   Si le mode jeton nécessite un jeton et qu'aucune source de jeton n'existe, doctor propose d'en générer un.
-   Si `gateway.auth.token` est géré par SecretRef mais indisponible, doctor avertit et ne l'écrase pas avec du texte clair.
-   `openclaw doctor --generate-gateway-token` force la génération uniquement lorsqu'aucun SecretRef de jeton n'est configuré.

### 12b) Réparations en lecture seule conscientes des SecretRef

Certains flux de réparation doivent inspecter les identifiants configurés sans affaiblir le comportement "fail-fast" d'exécution.

-   `openclaw doctor --fix` utilise désormais le même modèle de résumé en lecture seule des SecretRef que les commandes de la famille status pour les réparations ciblées de configuration.
-   Exemple : la réparation `@username` pour `allowFrom` / `groupAllowFrom` de Telegram essaie d'utiliser les identifiants de bot configurés lorsqu'ils sont disponibles.
-   Si le jeton du bot Telegram est configuré via SecretRef mais indisponible dans le chemin de commande actuel, doctor signale que l'identifiant est configuré-mais-indisponible et ignore la résolution automatique au lieu de planter ou de signaler incorrectement le jeton comme manquant.

### 13) Vérification d'état de santé de la passerelle + redémarrage

Doctor exécute une vérification d'état de santé et propose de redémarrer la passerelle lorsqu'elle semble malsaine.

### 14) Avertissements sur l'état des canaux

Si la passerelle est saine, doctor exécute une sonde d'état des canaux et rapporte des avertissements avec des corrections suggérées.

### 15) Audit + réparation de la configuration du superviseur

Doctor vérifie la configuration du superviseur installé (launchd/systemd/schtasks) pour des valeurs par défaut manquantes ou obsolètes (par exemple, les dépendances network-online systemd et le délai de redémarrage). Lorsqu'il trouve un décalage, il recommande une mise à jour et peut réécrire le fichier/tâche de service selon les valeurs par défaut actuelles. Notes :

-   `openclaw doctor` invite avant de réécrire la configuration du superviseur.
-   `openclaw doctor --yes` accepte les invites de réparation par défaut.
-   `openclaw doctor --repair` applique les corrections recommandées sans invite.
-   `openclaw doctor --repair --force` écrase les configurations personnalisées du superviseur.
-   Si l'authentification par jeton nécessite un jeton et que `gateway.auth.token` est géré par SecretRef, l'installation/réparation du service doctor valide le SecretRef mais ne persiste pas les valeurs de jeton en texte clair résolues dans les métadonnées d'environnement du service superviseur.
-   Si l'authentification par jeton nécessite un jeton et que le SecretRef de jeton configuré n'est pas résolu, doctor bloque le chemin d'installation/réparation avec des conseils actionnables.
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, doctor bloque l'installation/réparation jusqu'à ce que le mode soit défini explicitement.
-   Vous pouvez toujours forcer une réécriture complète via `openclaw gateway install --force`.

### 16) Diagnostics d'exécution + port de la passerelle

Doctor inspecte l'exécution du service (PID, dernier état de sortie) et avertit lorsque le service est installé mais n'est pas réellement en cours d'exécution. Il vérifie également les collisions de port sur le port de la passerelle (par défaut `18789`) et rapporte les causes probables (passerelle déjà en cours d'exécution, tunnel SSH).

### 17) Bonnes pratiques d'exécution de la passerelle

Doctor avertit lorsque le service de passerelle s'exécute sur Bun ou un chemin Node géré par un gestionnaire de version (`nvm`, `fnm`, `volta`, `asdf`, etc.). Les canaux WhatsApp + Telegram nécessitent Node, et les chemins des gestionnaires de version peuvent casser après des mises à niveau car le service ne charge pas votre initialisation de shell. Doctor propose de migrer vers une installation Node système lorsqu'elle est disponible (Homebrew/apt/choco).

### 18) Écriture de configuration + métadonnées de l'assistant

Doctor persiste tous les changements de configuration et tamponne les métadonnées de l'assistant pour enregistrer l'exécution de doctor.

### 19) Conseils sur l'espace de travail (sauvegarde + système de mémoire)

Doctor suggère un système de mémoire d'espace de travail lorsqu'il est manquant et affiche un conseil de sauvegarde si l'espace de travail n'est pas déjà sous git. Voir [/concepts/agent-workspace](../concepts/agent-workspace.md) pour un guide complet sur la structure de l'espace de travail et la sauvegarde git (GitHub ou GitLab privé recommandé).

[Heartbeat](./heartbeat.md)[Logging](./logging.md)