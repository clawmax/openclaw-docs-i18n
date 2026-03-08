

  Configuration

  
# Messages de groupe

Objectif : laisser Clawd dans les groupes WhatsApp, ne le réveiller que lorsqu'il est mentionné, et garder ce fil séparé de la session de messages privés. Note : `agents.list[].groupChat.mentionPatterns` est maintenant utilisé par Telegram/Discord/Slack/iMessage également ; ce document se concentre sur le comportement spécifique à WhatsApp. Pour les configurations multi-agents, définissez `agents.list[].groupChat.mentionPatterns` par agent (ou utilisez `messages.groupChat.mentionPatterns` comme valeur globale de repli).

## Ce qui est implémenté (2025-12-03)

-   Modes d'activation : `mention` (par défaut) ou `always`. `mention` nécessite une mention (vrais @-mentions WhatsApp via `mentionedJids`, motifs regex, ou le numéro E.164 du bot n'importe où dans le texte). `always` réveille l'agent sur chaque message mais il ne doit répondre que lorsqu'il peut apporter une valeur ajoutée ; sinon il retourne le jeton silencieux `NO_REPLY`. Les valeurs par défaut peuvent être définies dans la configuration (`channels.whatsapp.groups`) et remplacées par groupe via `/activation`. Lorsque `channels.whatsapp.groups` est défini, il agit également comme une liste d'autorisation de groupes (incluez `"*"` pour tout autoriser).
-   Politique de groupe : `channels.whatsapp.groupPolicy` contrôle si les messages de groupe sont acceptés (`open|disabled|allowlist`). `allowlist` utilise `channels.whatsapp.groupAllowFrom` (repli : `channels.whatsapp.allowFrom` explicite). La valeur par défaut est `allowlist` (bloqué jusqu'à ce que vous ajoutiez des expéditeurs).
-   Sessions par groupe : les clés de session ressemblent à `agent::whatsapp:group:` donc les commandes telles que `/verbose on` ou `/think high` (envoyées comme messages autonomes) sont limitées à ce groupe ; l'état des messages privés n'est pas affecté. Les battements de cœur sont ignorés pour les fils de groupe.
-   Injection de contexte : les messages de groupe **en attente uniquement** (50 par défaut) qui *n'ont pas* déclenché une exécution sont préfixés par `[Messages du chat depuis votre dernière réponse - pour le contexte]`, avec la ligne déclenchante sous `[Message actuel - répondez à celui-ci]`. Les messages déjà dans la session ne sont pas réinjectés.
-   Exposition de l'expéditeur : chaque lot de groupe se termine maintenant par `[de : Nom Expéditeur (+E164)]` pour que Pi sache qui parle.
-   Éphémères/à visualisation unique : nous les déballons avant d'extraire le texte/les mentions, donc les mentions à l'intérieur déclenchent toujours.
-   Invite système de groupe : au premier tour d'une session de groupe (et chaque fois que `/activation` change le mode) nous injectons un court texte dans l'invite système comme `Vous répondez dans le groupe WhatsApp "". Membres du groupe : Alice (+44...), Bob (+43...), … Activation : déclenchement uniquement … Adressez-vous à l'expéditeur spécifique indiqué dans le contexte du message.` Si les métadonnées ne sont pas disponibles, nous indiquons tout de même à l'agent qu'il s'agit d'un chat de groupe.

## Exemple de configuration (WhatsApp)

Ajoutez un bloc `groupChat` à `~/.openclaw/openclaw.json` pour que les mentions par nom d'affichage fonctionnent même lorsque WhatsApp supprime le `@` visuel dans le corps du texte :

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

Notes :

-   Les regex sont insensibles à la casse ; elles couvrent une mention par nom d'affichage comme `@openclaw` et le numéro brut avec ou sans `+`/espaces.
-   WhatsApp envoie toujours des mentions canoniques via `mentionedJids` lorsque quelqu'un appuie sur le contact, donc le repli sur le numéro est rarement nécessaire mais constitue un filet de sécurité utile.

### Commande d'activation (propriétaire uniquement)

Utilisez la commande de chat de groupe :

-   `/activation mention`
-   `/activation always`

Seul le numéro propriétaire (de `channels.whatsapp.allowFrom`, ou le propre numéro E.164 du bot si non défini) peut changer cela. Envoyez `/status` comme message autonome dans le groupe pour voir le mode d'activation actuel.

## Comment utiliser

1.  Ajoutez votre compte WhatsApp (celui qui exécute OpenClaw) au groupe.
2.  Dites `@openclaw …` (ou incluez le numéro). Seuls les expéditeurs autorisés peuvent le déclencher sauf si vous définissez `groupPolicy: "open"`.
3.  L'invite de l'agent inclura le contexte récent du groupe plus le marqueur final `[de : …]` pour qu'il puisse s'adresser à la bonne personne.
4.  Les directives au niveau de la session (`/verbose on`, `/think high`, `/new` ou `/reset`, `/compact`) s'appliquent uniquement à la session de ce groupe ; envoyez-les comme messages autonomes pour qu'elles soient enregistrées. Votre session de messages privés reste indépendante.

## Tests / vérification

-   Test manuel rapide :
    -   Envoyez une mention `@openclaw` dans le groupe et confirmez une réponse qui référence le nom de l'expéditeur.
    -   Envoyez une deuxième mention et vérifiez que le bloc d'historique est inclus puis effacé au tour suivant.
-   Vérifiez les journaux de la passerelle (exécutez avec `--verbose`) pour voir les entrées `inbound web message` montrant `from: ` et le suffixe `[from: …]`.

## Considérations connues

-   Les battements de cœur sont intentionnellement ignorés pour les groupes pour éviter les diffusions bruyantes.
-   La suppression des échos utilise la chaîne de lot combinée ; si vous envoyez deux fois le même texte sans mentions, seule la première obtiendra une réponse.
-   Les entrées du magasin de session apparaîtront comme `agent::whatsapp:group:` dans le magasin de session (`~/.openclaw/agents//sessions/sessions.json` par défaut) ; une entrée manquante signifie simplement que le groupe n'a pas encore déclenché d'exécution.
-   Les indicateurs de saisie dans les groupes suivent `agents.defaults.typingMode` (par défaut : `message` lorsqu'il n'est pas mentionné).

[Jumelage](./pairing.md)[Groupes](./groups.md)