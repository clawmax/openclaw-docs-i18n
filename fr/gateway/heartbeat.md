

  Configuration et opérations

  
# Heartbeat

> **Heartbeat vs Cron ?** Voir [Cron vs Heartbeat](../automation/cron-vs-heartbeat.md) pour savoir quand utiliser chacun.

Le Heartbeat exécute des **tours d'agent périodiques** dans la session principale afin que le modèle puisse signaler tout ce qui nécessite une attention sans vous spammer. Dépannage : [/automation/troubleshooting](../automation/troubleshooting.md)

## Démarrage rapide (débutant)

1.  Laissez les heartbeats activés (par défaut `30m`, ou `1h` pour OAuth/setup-token Anthropic) ou définissez votre propre cadence.
2.  Créez une petite checklist `HEARTBEAT.md` dans l'espace de travail de l'agent (optionnel mais recommandé).
3.  Décidez où les messages de heartbeat doivent aller (`target: "none"` est la valeur par défaut ; définissez `target: "last"` pour les acheminer vers le dernier contact).
4.  Optionnel : activez la livraison du raisonnement du heartbeat pour la transparence.
5.  Optionnel : utilisez un contexte bootstrap léger si les heartbeats n'ont besoin que de `HEARTBEAT.md`.
6.  Optionnel : restreignez les heartbeats aux heures actives (heure locale).

Exemple de configuration :

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
        directPolicy: "allow", // par défaut : autoriser les cibles directes/DM ; définir "block" pour supprimer
        lightContext: true, // optionnel : n'injecte que HEARTBEAT.md depuis les fichiers bootstrap
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // optionnel : envoie aussi un message séparé `Reasoning:`
      },
    },
  },
}
```

## Valeurs par défaut

-   Intervalle : `30m` (ou `1h` lorsque OAuth/setup-token Anthropic est le mode d'authentification détecté). Définissez `agents.defaults.heartbeat.every` ou par agent `agents.list[].heartbeat.every` ; utilisez `0m` pour désactiver.
-   Corps de l'invite (configurable via `agents.defaults.heartbeat.prompt`) : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
-   L'invite de heartbeat est envoyée **textuellement** en tant que message utilisateur. L'invite système inclut une section "Heartbeat" et l'exécution est marquée en interne.
-   Les heures actives (`heartbeat.activeHours`) sont vérifiées dans le fuseau horaire configuré. En dehors de la fenêtre, les heartbeats sont ignorés jusqu'au prochain tick à l'intérieur de la fenêtre.

## À quoi sert l'invite de heartbeat

L'invite par défaut est intentionnellement large :

-   **Tâches en arrière-plan** : "Considérer les tâches en suspens" incite l'agent à examiner les relances (boîte de réception, calendrier, rappels, travail en file d'attente) et à signaler tout élément urgent.
-   **Vérification humaine** : "Vérifiez parfois votre humain pendant la journée" incite à un message léger occasionnel "avez-vous besoin de quelque chose ?", mais évite le spam nocturne en utilisant votre fuseau horaire local configuré (voir [/concepts/timezone](../concepts/timezone.md)).

Si vous voulez qu'un heartbeat fasse quelque chose de très spécifique (par exemple "vérifier les stats Gmail PubSub" ou "vérifier l'état de la passerelle"), définissez `agents.defaults.heartbeat.prompt` (ou `agents.list[].heartbeat.prompt`) sur un corps personnalisé (envoyé textuellement).

## Contrat de réponse

-   Si rien ne nécessite d'attention, répondez avec **`HEARTBEAT_OK`**.
-   Pendant les exécutions de heartbeat, OpenClaw traite `HEARTBEAT_OK` comme un accusé de réception lorsqu'il apparaît au **début ou à la fin** de la réponse. Le jeton est supprimé et la réponse est ignorée si le contenu restant est **≤ `ackMaxChars`** (par défaut : 300).
-   Si `HEARTBEAT_OK` apparaît au **milieu** d'une réponse, il n'est pas traité spécialement.
-   Pour les alertes, **ne pas** inclure `HEARTBEAT_OK` ; retournez uniquement le texte de l'alerte.

En dehors des heartbeats, un `HEARTBEAT_OK` isolé au début/à la fin d'un message est supprimé et journalisé ; un message qui n'est que `HEARTBEAT_OK` est ignoré.

## Configuration

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // par défaut : 30m (0m désactive)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // par défaut : false (livre un message séparé Reasoning: quand disponible)
        lightContext: false, // par défaut : false ; true ne garde que HEARTBEAT.md des fichiers bootstrap de l'espace de travail
        target: "last", // par défaut : none | options : last | none | <channel id> (core ou plugin, ex. "bluebubbles")
        to: "+15551234567", // remplacement optionnel spécifique au canal
        accountId: "ops-bot", // identifiant de canal multi-compte optionnel
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // caractères maximum autorisés après HEARTBEAT_OK
      },
    },
  },
}
```

### Portée et précédence

-   `agents.defaults.heartbeat` définit le comportement global du heartbeat.
-   `agents.list[].heartbeat` fusionne par-dessus ; si un agent a un bloc `heartbeat`, **seuls ces agents** exécutent des heartbeats.
-   `channels.defaults.heartbeat` définit les valeurs par défaut de visibilité pour tous les canaux.
-   `channels..heartbeat` remplace les valeurs par défaut du canal.
-   `channels..accounts..heartbeat` (canaux multi-comptes) remplace les paramètres par canal.

### Heartbeats par agent

Si une entrée `agents.list[]` inclut un bloc `heartbeat`, **seuls ces agents** exécutent des heartbeats. Le bloc par agent fusionne par-dessus `agents.defaults.heartbeat` (vous pouvez donc définir des valeurs par défaut partagées une fois et les remplacer par agent). Exemple : deux agents, seul le second exécute des heartbeats.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### Exemple d'heures actives

Restreindre les heartbeats aux heures de bureau dans un fuseau horaire spécifique :

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // optionnel ; utilise votre userTimezone si défini, sinon le fuseau de l'hôte
        },
      },
    },
  },
}
```

En dehors de cette fenêtre (avant 9h ou après 22h heure de l'Est), les heartbeats sont ignorés. Le prochain tick planifié à l'intérieur de la fenêtre s'exécutera normalement.

### Configuration 24/7

Si vous voulez que les heartbeats s'exécutent toute la journée, utilisez l'un de ces modèles :

-   Omettez entièrement `activeHours` (pas de restriction de fenêtre horaire ; c'est le comportement par défaut).
-   Définissez une fenêtre de journée complète : `activeHours: { start: "00:00", end: "24:00" }`.

Ne définissez pas la même heure pour `start` et `end` (par exemple `08:00` à `08:00`). Cela est traité comme une fenêtre de largeur nulle, donc les heartbeats sont toujours ignorés.

### Exemple multi-compte

Utilisez `accountId` pour cibler un compte spécifique sur les canaux multi-comptes comme Telegram :

```json
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // optionnel : acheminer vers un sujet/fil spécifique
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### Notes sur les champs

-   `every` : intervalle du heartbeat (chaîne de durée ; unité par défaut = minutes).
-   `model` : remplacement de modèle optionnel pour les exécutions de heartbeat (`provider/model`).
-   `includeReasoning` : lorsqu'activé, livre aussi le message séparé `Reasoning:` quand disponible (même forme que `/reasoning on`).
-   `lightContext` : quand true, les exécutions de heartbeat utilisent un contexte bootstrap léger et ne gardent que `HEARTBEAT.md` des fichiers bootstrap de l'espace de travail.
-   `session` : clé de session optionnelle pour les exécutions de heartbeat.
    -   `main` (par défaut) : session principale de l'agent.
    -   Clé de session explicite (copiée depuis `openclaw sessions --json` ou la [CLI des sessions](../cli/sessions.md)).
    -   Formats de clé de session : voir [Sessions](../concepts/session.md) et [Groups](../channels/groups.md).
-   `target` :
    -   `last` : livrer au dernier canal externe utilisé.
    -   canal explicite : `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
    -   `none` (par défaut) : exécuter le heartbeat mais **ne pas livrer** en externe.
-   `directPolicy` : contrôle le comportement de livraison directe/DM :
    -   `allow` (par défaut) : autoriser la livraison de heartbeat directe/DM.
    -   `block` : supprimer la livraison directe/DM (`reason=dm-blocked`).
-   `to` : remplacement de destinataire optionnel (identifiant spécifique au canal, ex. E.164 pour WhatsApp ou un identifiant de chat Telegram). Pour les sujets/fils Telegram, utilisez `:topic:`.
-   `accountId` : identifiant de compte optionnel pour les canaux multi-comptes. Quand `target: "last"`, l'identifiant de compte s'applique au canal dernier résolu s'il prend en charge les comptes ; sinon il est ignoré. Si l'identifiant de compte ne correspond pas à un compte configuré pour le canal résolu, la livraison est ignorée.
-   `prompt` : remplace le corps de l'invite par défaut (non fusionné).
-   `ackMaxChars` : caractères maximum autorisés après `HEARTBEAT_OK` avant livraison.
-   `suppressToolErrorWarnings` : quand true, supprime les charges utiles d'avertissement d'erreur d'outil pendant les exécutions de heartbeat.
-   `activeHours` : restreint les exécutions de heartbeat à une fenêtre horaire. Objet avec `start` (HH:MM, inclusif ; utilisez `00:00` pour début de journée), `end` (HH:MM exclusif ; `24:00` autorisé pour fin de journée), et `timezone` optionnel.
    -   Omis ou `"user"` : utilise votre `agents.defaults.userTimezone` si défini, sinon utilise le fuseau horaire du système hôte.
    -   `"local"` : utilise toujours le fuseau horaire du système hôte.
    -   Tout identifiant IANA (ex. `America/New_York`) : utilisé directement ; si invalide, utilise le comportement `"user"` ci-dessus.
    -   `start` et `end` ne doivent pas être égaux pour une fenêtre active ; des valeurs égales sont traitées comme une fenêtre de largeur nulle (toujours en dehors de la fenêtre).
    -   En dehors de la fenêtre active, les heartbeats sont ignorés jusqu'au prochain tick à l'intérieur de la fenêtre.

## Comportement de livraison

-   Les heartbeats s'exécutent dans la session principale de l'agent par défaut (`agent::`), ou `global` quand `session.scope = "global"`. Définissez `session` pour remplacer par une session de canal spécifique (Discord/WhatsApp/etc.).
-   `session` n'affecte que le contexte d'exécution ; la livraison est contrôlée par `target` et `to`.
-   Pour livrer à un canal/destinataire spécifique, définissez `target` + `to`. Avec `target: "last"`, la livraison utilise le dernier canal externe pour cette session.
-   Les livraisons de heartbeat autorisent les cibles directes/DM par défaut. Définissez `directPolicy: "block"` pour supprimer les envois vers des cibles directes tout en exécutant le tour de heartbeat.
-   Si la file principale est occupée, le heartbeat est ignoré et réessayé plus tard.
-   Si `target` ne résout aucune destination externe, l'exécution a quand même lieu mais aucun message sortant n'est envoyé.
-   Les réponses uniquement de heartbeat **ne** maintiennent **pas** la session active ; le dernier `updatedAt` est restauré pour que l'expiration d'inactivité se comporte normalement.

## Contrôles de visibilité

Par défaut, les accusés de réception `HEARTBEAT_OK` sont supprimés tandis que le contenu d'alerte est livré. Vous pouvez ajuster cela par canal ou par compte :

```
channels:
  defaults:
    heartbeat:
      showOk: false # Masquer HEARTBEAT_OK (par défaut)
      showAlerts: true # Afficher les messages d'alerte (par défaut)
      useIndicator: true # Émettre des événements indicateurs (par défaut)
  telegram:
    heartbeat:
      showOk: true # Afficher les accusés OK sur Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Supprimer la livraison d'alertes pour ce compte
```

Précédence : par-compte → par-canal → valeurs par défaut du canal → valeurs par défaut intégrées.

### Ce que fait chaque drapeau

-   `showOk` : envoie un accusé de réception `HEARTBEAT_OK` quand le modèle retourne une réponse uniquement OK.
-   `showAlerts` : envoie le contenu d'alerte quand le modèle retourne une réponse non-OK.
-   `useIndicator` : émet des événements indicateurs pour les surfaces d'état de l'interface utilisateur.

Si **les trois** sont faux, OpenClaw ignore entièrement l'exécution du heartbeat (pas d'appel au modèle).

### Exemples par canal vs par compte

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # tous les comptes Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # supprimer les alertes uniquement pour le compte ops
  telegram:
    heartbeat:
      showOk: true
```

### Modèles courants

| Objectif | Configuration |
| --- | --- |
| Comportement par défaut (OK silencieux, alertes activées) | *(aucune config nécessaire)* |
| Complètement silencieux (pas de messages, pas d'indicateur) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Indicateur uniquement (pas de messages) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OK dans un seul canal | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md (optionnel)

Si un fichier `HEARTBEAT.md` existe dans l'espace de travail, l'invite par défaut dit à l'agent de le lire. Considérez-le comme votre "checklist de heartbeat" : petite, stable et sûre à inclure toutes les 30 minutes. Si `HEARTBEAT.md` existe mais est effectivement vide (seulement des lignes vides et des en-têtes markdown comme `# Heading`), OpenClaw ignore l'exécution du heartbeat pour économiser des appels API. Si le fichier est manquant, le heartbeat s'exécute quand même et le modèle décide quoi faire. Gardez-le minuscule (courte checklist ou rappels) pour éviter l'encombrement de l'invite. Exemple `HEARTBEAT.md` :

```bash
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it’s daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### L'agent peut-il mettre à jour HEARTBEAT.md ?

Oui — si vous le lui demandez. `HEARTBEAT.md` est juste un fichier normal dans l'espace de travail de l'agent, donc vous pouvez dire à l'agent (dans un chat normal) quelque chose comme :

-   "Mettez à jour `HEARTBEAT.md` pour ajouter une vérification quotidienne du calendrier."
-   "Réécrivez `HEARTBEAT.md` pour qu'il soit plus court et se concentre sur les relances de boîte de réception."

Si vous voulez que cela se produise de manière proactive, vous pouvez aussi inclure une ligne explicite dans votre invite de heartbeat comme : "Si la checklist devient obsolète, mettez à jour HEARTBEAT.md avec une meilleure." Note de sécurité : ne mettez pas de secrets (clés API, numéros de téléphone, jetons privés) dans `HEARTBEAT.md` — cela fait partie du contexte de l'invite.

## Réveil manuel (à la demande)

Vous pouvez mettre en file d'attente un événement système et déclencher un heartbeat immédiat avec :

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Si plusieurs agents ont `heartbeat` configuré, un réveil manuel exécute immédiatement le heartbeat de chacun de ces agents. Utilisez `--mode next-heartbeat` pour attendre le prochain tick planifié.

## Livraison du raisonnement (optionnel)

Par défaut, les heartbeats livrent uniquement la charge utile "réponse" finale. Si vous voulez de la transparence, activez :

-   `agents.defaults.heartbeat.includeReasoning: true`

Quand activé, les heartbeats livreront aussi un message séparé préfixé `Reasoning:` (même forme que `/reasoning on`). Cela peut être utile quand l'agent gère plusieurs sessions/codex et que vous voulez voir pourquoi il a décidé de vous notifier — mais cela peut aussi divulguer plus de détails internes que vous ne le souhaitez. Préférez le garder désactivé dans les chats de groupe.

## Conscience des coûts

Les heartbeats exécutent des tours d'agent complets. Des intervalles plus courts consomment plus de jetons. Gardez `HEARTBEAT.md` petit et envisagez un `model` moins cher ou `target: "none"` si vous ne voulez que des mises à jour d'état internes.

[Health Checks](./health.md)[Doctor](./doctor.md)