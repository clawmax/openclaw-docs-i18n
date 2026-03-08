

  Plateformes de messagerie

  
# iMessage

> **⚠️** Pour les nouveaux déploiements iMessage, utilisez [BlueBubbles](./bluebubbles.md). L'intégration `imsg` est héritée et pourrait être supprimée dans une future version.

 Statut : intégration CLI externe héritée. La passerelle lance `imsg rpc` et communique via JSON-RPC sur stdio (pas de démon/port séparé). 

## Configuration rapide

## Prérequis et permissions (macOS)

-   L'application Messages doit être connectée sur le Mac exécutant `imsg`.
-   L'accès complet au disque est requis pour le contexte de processus exécutant OpenClaw/`imsg` (accès à la base de données Messages).
-   La permission d'automatisation est requise pour envoyer des messages via Messages.app.

> **💡** Les permissions sont accordées par contexte de processus. Si la passerelle s'exécute sans interface (LaunchAgent/SSH), exécutez une commande interactive unique dans ce même contexte pour déclencher les invites :
> 
> Copier
> 
> ```
> imsg chats --limit 1
> # ou
> imsg send <handle> "test"
> ```

## Contrôle d'accès et routage

`channels.imessage.dmPolicy` contrôle les messages directs :

-   `pairing` (par défaut)
-   `allowlist`
-   `open` (nécessite que `allowFrom` inclue `"*"`)
-   `disabled`

Champ de liste d'autorisation : `channels.imessage.allowFrom`.Les entrées de la liste d'autorisation peuvent être des identifiants ou des cibles de discussion (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

`channels.imessage.groupPolicy` contrôle la gestion des groupes :

-   `allowlist` (par défaut lorsqu'il est configuré)
-   `open`
-   `disabled`

Liste d'autorisation des expéditeurs de groupe : `channels.imessage.groupAllowFrom`.Fallback d'exécution : si `groupAllowFrom` n'est pas défini, les vérifications de l'expéditeur de groupe iMessage utilisent `allowFrom` lorsqu'il est disponible. Note d'exécution : si `channels.imessage` est complètement absent, l'exécution utilise `groupPolicy="allowlist"` et enregistre un avertissement (même si `channels.defaults.groupPolicy` est défini).Contrôle des mentions pour les groupes :

-   iMessage n'a pas de métadonnées de mention natives
-   la détection des mentions utilise des motifs regex (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
-   sans motifs configurés, le contrôle des mentions ne peut pas être appliqué

Les commandes de contrôle des expéditeurs autorisés peuvent contourner le contrôle des mentions dans les groupes.

-   Les messages directs utilisent le routage direct ; les groupes utilisent le routage de groupe.
-   Avec `session.dmScope=main` par défaut, les messages directs iMessage sont regroupés dans la session principale de l'agent.
-   Les sessions de groupe sont isolées (`agent::imessage:group:<chat_id>`).
-   Les réponses sont routées vers iMessage en utilisant les métadonnées du canal/cible d'origine.

Comportement de fil de discussion de type groupe :Certains fils de discussion iMessage multi-participants peuvent arriver avec `is_group=false`. Si ce `chat_id` est explicitement configuré sous `channels.imessage.groups`, OpenClaw le traite comme du trafic de groupe (contrôle de groupe + isolation de session de groupe).

## Modèles de déploiement

Utilisez un identifiant Apple et un utilisateur macOS dédiés pour isoler le trafic du bot de votre profil Messages personnel.Flux typique :

1.  Créez/connectez un utilisateur macOS dédié.
2.  Connectez-vous à Messages avec l'identifiant Apple du bot dans cet utilisateur.
3.  Installez `imsg` dans cet utilisateur.
4.  Créez un wrapper SSH pour qu'OpenClaw puisse exécuter `imsg` dans le contexte de cet utilisateur.
5.  Pointez `channels.imessage.accounts..cliPath` et `.dbPath` vers le profil de cet utilisateur.

Le premier lancement peut nécessiter des approbations via l'interface graphique (Automatisation + Accès complet au disque) dans la session de l'utilisateur bot.

Topologie courante :

-   la passerelle s'exécute sur Linux/VM
-   iMessage + `imsg` s'exécute sur un Mac dans votre tailnet
-   le wrapper `cliPath` utilise SSH pour exécuter `imsg`
-   `remoteHost` active la récupération des pièces jointes via SCP

Exemple :

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Utilisez des clés SSH pour que SSH et SCP soient non interactifs. Assurez-vous d'abord que la clé hôte est approuvée (par exemple `ssh bot@mac-mini.tailnet-1234.ts.net`) pour que `known_hosts` soit rempli.

iMessage prend en charge la configuration par compte sous `channels.imessage.accounts`.Chaque compte peut remplacer des champs tels que `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, les paramètres d'historique et les listes d'autorisation racine des pièces jointes.

## Médias, fragmentation et cibles d'envoi

-   l'ingestion des pièces jointes entrantes est optionnelle : `channels.imessage.includeAttachments`
-   les chemins distants des pièces jointes peuvent être récupérés via SCP lorsque `remoteHost` est défini
-   les chemins des pièces jointes doivent correspondre aux racines autorisées :
    -   `channels.imessage.attachmentRoots` (local)
    -   `channels.imessage.remoteAttachmentRoots` (mode SCP distant)
    -   motif racine par défaut : `/Users/*/Library/Messages/Attachments`
-   SCP utilise une vérification stricte des clés hôtes (`StrictHostKeyChecking=yes`)
-   la taille des médias sortants utilise `channels.imessage.mediaMaxMb` (16 Mo par défaut)

-   limite de fragmentation de texte : `channels.imessage.textChunkLimit` (4000 par défaut)
-   mode de fragmentation : `channels.imessage.chunkMode`
    -   `length` (par défaut)
    -   `newline` (division par paragraphe en priorité)

Cibles explicites préférées :

-   `chat_id:123` (recommandé pour un routage stable)
-   `chat_guid:...`
-   `chat_identifier:...`

Les cibles par identifiant sont également prises en charge :

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## Écritures de configuration

iMessage autorise par défaut les écritures de configuration initiées par le canal (pour `/config set|unset` lorsque `commands.config: true`). Désactivez :

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Dépannage

Validez le binaire et la prise en charge RPC :

```bash
imsg rpc --help
openclaw channels status --probe
```

Si la sonde signale que RPC n'est pas pris en charge, mettez à jour `imsg`.

Vérifiez :

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   les approbations d'appairage (`openclaw pairing list imessage`)

Vérifiez :

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   le comportement de la liste d'autorisation `channels.imessage.groups`
-   la configuration des motifs de mention (`agents.list[].groupChat.mentionPatterns`)

Vérifiez :

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   l'authentification par clé SSH/SCP depuis l'hôte de la passerelle
-   la clé hôte existe dans `~/.ssh/known_hosts` sur l'hôte de la passerelle
-   la lisibilité du chemin distant sur le Mac exécutant Messages

Ré-exécutez dans un terminal graphique interactif dans le même contexte utilisateur/session et approuvez les invites :

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

Confirmez que l'Accès complet au disque + l'Automatisation sont accordés pour le contexte de processus qui exécute OpenClaw/`imsg`.

## Pointeurs vers la référence de configuration

-   [Référence de configuration - iMessage](../gateway/configuration-reference.md#imessage)
-   [Configuration de la passerelle](../gateway/configuration.md)
-   [Appairage](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)

---