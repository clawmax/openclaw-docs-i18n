title: "Guide de configuration et d'installation du canal Signal pour OpenClaw"
description: "Apprenez à configurer et installer le canal Signal pour OpenClaw en utilisant signal-cli. Inclut les prérequis, les étapes de configuration rapide et les options de configuration détaillées."
keywords: ["openclaw signal", "intégration signal-cli", "configuration bot signal", "configuration plateforme de messagerie", "guide canal signal", "passerelle openclaw", "automatisation signal", "appairage politique dm"]
---

  Plateformes de messagerie

  
# Signal

Statut : intégration CLI externe. La passerelle communique avec `signal-cli` via HTTP JSON-RPC + SSE.

## Prérequis

-   OpenClaw installé sur votre serveur (le flux Linux ci-dessous testé sur Ubuntu 24).
-   `signal-cli` disponible sur l'hôte où la passerelle s'exécute.
-   Un numéro de téléphone pouvant recevoir un SMS de vérification (pour le chemin d'inscription par SMS).
-   Accès à un navigateur pour le captcha Signal (`signalcaptchas.org`) pendant l'inscription.

## Configuration rapide (débutant)

1.  Utilisez un **numéro Signal séparé** pour le bot (recommandé).
2.  Installez `signal-cli` (Java requis si vous utilisez la version JVM).
3.  Choisissez un chemin de configuration :
    -   **Chemin A (lien QR) :** `signal-cli link -n "OpenClaw"` et scannez avec Signal.
    -   **Chemin B (inscription SMS) :** enregistrez un numéro dédié avec captcha + vérification par SMS.
4.  Configurez OpenClaw et redémarrez la passerelle.
5.  Envoyez un premier message privé et approuvez l'appairage (`openclaw pairing approve signal `).

Configuration minimale :

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Référence des champs :

| Champ | Description |
| --- | --- |
| `account` | Numéro de téléphone du bot au format E.164 (`+15551234567`) |
| `cliPath` | Chemin vers `signal-cli` (`signal-cli` si dans le `PATH`) |
| `dmPolicy` | Politique d'accès aux messages privés (`pairing` recommandé) |
| `allowFrom` | Numéros de téléphone ou valeurs `uuid:` autorisés à envoyer des messages privés |

## Ce que c'est

-   Canal Signal via `signal-cli` (pas de libsignal intégrée).
-   Routage déterministe : les réponses retournent toujours vers Signal.
-   Les messages privés partagent la session principale de l'agent ; les groupes sont isolés (`agent::signal:group:`).

## Écritures de configuration

Par défaut, Signal est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`). Désactivez avec :

```json
{
  channels: { signal: { configWrites: false } },
}
```

## Le modèle de numéro (important)

-   La passerelle se connecte à un **appareil Signal** (le compte `signal-cli`).
-   Si vous exécutez le bot sur **votre compte Signal personnel**, il ignorera vos propres messages (protection contre les boucles).
-   Pour le scénario « Je texte le bot et il répond », utilisez un **numéro de bot séparé**.

## Chemin de configuration A : lier un compte Signal existant (QR)

1.  Installez `signal-cli` (version JVM ou native).
2.  Liez un compte de bot :
    -   `signal-cli link -n "OpenClaw"` puis scannez le QR dans Signal.
3.  Configurez Signal et démarrez la passerelle.

Exemple :

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Support multi-comptes : utilisez `channels.signal.accounts` avec une configuration par compte et un `name` optionnel. Voir [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modèle partagé.

## Chemin de configuration B : enregistrer un numéro de bot dédié (SMS, Linux)

Utilisez ceci lorsque vous voulez un numéro de bot dédié au lieu de lier un compte d'application Signal existant.

1.  Obtenez un numéro pouvant recevoir des SMS (ou une vérification vocale pour les lignes fixes).
    -   Utilisez un numéro de bot dédié pour éviter les conflits de compte/session.
2.  Installez `signal-cli` sur l'hôte de la passerelle :

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Si vous utilisez la version JVM (`signal-cli-${VERSION}.tar.gz`), installez d'abord JRE 25+. Gardez `signal-cli` à jour ; les notes en amont indiquent que les anciennes versions peuvent casser lorsque les API du serveur Signal changent.

3.  Enregistrez et vérifiez le numéro :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Si un captcha est requis :

1.  Ouvrez `https://signalcaptchas.org/registration/generate.html`.
2.  Complétez le captcha, copiez la cible du lien `signalcaptcha://...` depuis « Ouvrir Signal ».
3.  Exécutez depuis la même IP externe que la session du navigateur si possible.
4.  Relancez l'enregistrement immédiatement (les jetons captcha expirent rapidement) :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4.  Configurez OpenClaw, redémarrez la passerelle, vérifiez le canal :

```bash
# Si vous exécutez la passerelle en tant que service systemd utilisateur :
systemctl --user restart openclaw-gateway

# Puis vérifiez :
openclaw doctor
openclaw channels status --probe
```

5.  Appairez votre expéditeur de message privé :
    -   Envoyez n'importe quel message au numéro du bot.
    -   Approuvez le code sur le serveur : `openclaw pairing approve signal <PAIRING_CODE>`.
    -   Enregistrez le numéro du bot comme contact sur votre téléphone pour éviter « Contact inconnu ».

Important : l'enregistrement d'un compte avec numéro de téléphone via `signal-cli` peut déconnecter la session principale de l'application Signal pour ce numéro. Préférez un numéro de bot dédié, ou utilisez le mode lien QR si vous devez conserver votre configuration d'application téléphonique existante. Références en amont :

-   README de `signal-cli` : `https://github.com/AsamK/signal-cli`
-   Flux captcha : `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   Flux de liaison : `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Mode démon externe (httpUrl)

Si vous souhaitez gérer `signal-cli` vous-même (démarrages à froid JVM lents, initialisation de conteneur, ou CPU partagés), exécutez le démon séparément et pointez OpenClaw vers lui :

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Cela évite le lancement automatique et l'attente de démarrage dans OpenClaw. Pour les démarrages lents lors du lancement automatique, définissez `channels.signal.startupTimeoutMs`.

## Contrôle d'accès (messages privés + groupes)

Messages privés :

-   Par défaut : `channels.signal.dmPolicy = "pairing"`.
-   Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'à approbation (les codes expirent après 1 heure).
-   Approuvez via :
    -   `openclaw pairing list signal`
    -   `openclaw pairing approve signal `
-   L'appairage est l'échange de jetons par défaut pour les messages privés Signal. Détails : [Appairage](./pairing.md)
-   Les expéditeurs uniquement UUID (depuis `sourceUuid`) sont stockés comme `uuid:` dans `channels.signal.allowFrom`.

Groupes :

-   `channels.signal.groupPolicy = open | allowlist | disabled`.
-   `channels.signal.groupAllowFrom` contrôle qui peut déclencher dans les groupes lorsque `allowlist` est défini.
-   Note d'exécution : si `channels.signal` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

## Fonctionnement (comportement)

-   `signal-cli` s'exécute en tant que démon ; la passerelle lit les événements via SSE.
-   Les messages entrants sont normalisés dans l'enveloppe de canal partagée.
-   Les réponses sont toujours routées vers le même numéro ou groupe.

## Médias + limites

-   Le texte sortant est découpé en morceaux de `channels.signal.textChunkLimit` (par défaut 4000).
-   Découpage optionnel par sauts de ligne : définissez `channels.signal.chunkMode="newline"` pour diviser sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   Pièces jointes supportées (base64 récupéré depuis `signal-cli`).
-   Limite média par défaut : `channels.signal.mediaMaxMb` (par défaut 8).
-   Utilisez `channels.signal.ignoreAttachments` pour ignorer le téléchargement des médias.
-   Le contexte d'historique de groupe utilise `channels.signal.historyLimit` (ou `channels.signal.accounts.*.historyLimit`), avec repli sur `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (par défaut 50).

## Indicateurs de frappe + accusés de lecture

-   **Indicateurs de frappe** : OpenClaw envoie des signaux de frappe via `signal-cli sendTyping` et les rafraîchit pendant qu'une réponse est en cours.
-   **Accusés de lecture** : lorsque `channels.signal.sendReadReceipts` est vrai, OpenClaw transmet les accusés de lecture pour les messages privés autorisés.
-   Signal-cli n'expose pas les accusés de lecture pour les groupes.

## Réactions (outil message)

-   Utilisez `message action=react` avec `channel=signal`.
-   Cibles : expéditeur E.164 ou UUID (utilisez `uuid:` depuis la sortie d'appairage ; l'UUID seul fonctionne aussi).
-   `messageId` est l'horodatage Signal du message auquel vous réagissez.
-   Les réactions en groupe nécessitent `targetAuthor` ou `targetAuthorUuid`.

Exemples :

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuration :

-   `channels.signal.actions.reactions` : active/désactive les actions de réaction (par défaut vrai).
-   `channels.signal.reactionLevel` : `off | ack | minimal | extensive`.
    -   `off`/`ack` désactive les réactions de l'agent (l'outil message `react` générera une erreur).
    -   `minimal`/`extensive` active les réactions de l'agent et définit le niveau de guidage.
-   Surcharges par compte : `channels.signal.accounts..actions.reactions`, `channels.signal.accounts..reactionLevel`.

## Cibles de livraison (CLI/cron)

-   Messages privés : `signal:+15551234567` (ou E.164 simple).
-   Messages privés UUID : `uuid:` (ou UUID seul).
-   Groupes : `signal:group:`.
-   Noms d'utilisateur : `username:` (si supporté par votre compte Signal).

## Dépannage

Exécutez d'abord cette échelle :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis confirmez l'état d'appairage des messages privés si nécessaire :

```bash
openclaw pairing list signal
```

Échecs courants :

-   Démon accessible mais pas de réponses : vérifiez les paramètres compte/démon (`httpUrl`, `account`) et le mode de réception.
-   Messages privés ignorés : l'expéditeur est en attente d'approbation d'appairage.
-   Messages de groupe ignorés : le filtrage de l'expéditeur/mention de groupe bloque la livraison.
-   Erreurs de validation de configuration après modifications : exécutez `openclaw doctor --fix`.
-   Signal absent des diagnostics : confirmez `channels.signal.enabled: true`.

Vérifications supplémentaires :

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Pour le flux de triage : [/channels/troubleshooting](./troubleshooting.md).

## Notes de sécurité

-   `signal-cli` stocke les clés de compte localement (typiquement `~/.local/share/signal-cli/data/`).
-   Sauvegardez l'état du compte Signal avant une migration ou reconstruction de serveur.
-   Gardez `channels.signal.dmPolicy: "pairing"` sauf si vous voulez explicitement un accès plus large aux messages privés.
-   La vérification par SMS n'est nécessaire que pour les flux d'inscription ou de récupération, mais perdre le contrôle du numéro/compte peut compliquer la réinscription.

## Référence de configuration (Signal)

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.signal.enabled` : active/désactive le démarrage du canal.
-   `channels.signal.account` : E.164 pour le compte du bot.
-   `channels.signal.cliPath` : chemin vers `signal-cli`.
-   `channels.signal.httpUrl` : URL complète du démon (remplace hôte/port).
-   `channels.signal.httpHost`, `channels.signal.httpPort` : liaison du démon (par défaut 127.0.0.1:8080).
-   `channels.signal.autoStart` : lancement automatique du démon (par défaut vrai si `httpUrl` non défini).
-   `channels.signal.startupTimeoutMs` : délai d'attente de démarrage en ms (plafond 120000).
-   `channels.signal.receiveMode` : `on-start | manual`.
-   `channels.signal.ignoreAttachments` : ignore le téléchargement des pièces jointes.
-   `channels.signal.ignoreStories` : ignore les stories du démon.
-   `channels.signal.sendReadReceipts` : transmet les accusés de lecture.
-   `channels.signal.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
-   `channels.signal.allowFrom` : liste d'autorisation pour les messages privés (E.164 ou `uuid:`). `open` nécessite `"*"`. Signal n'a pas de noms d'utilisateur ; utilisez les identifiants téléphone/UUID.
-   `channels.signal.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
-   `channels.signal.groupAllowFrom` : liste d'autorisation des expéditeurs de groupe.
-   `channels.signal.historyLimit` : nombre maximum de messages de groupe à inclure comme contexte (0 désactive).
-   `channels.signal.dmHistoryLimit` : limite d'historique des messages privés en tours utilisateur. Surcharges par utilisateur : `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
-   `channels.signal.textChunkLimit` : taille des morceaux sortants (caractères).
-   `channels.signal.chunkMode` : `length` (par défaut) ou `newline` pour diviser sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   `channels.signal.mediaMaxMb` : plafond média entrant/sortant (MB).

Options globales connexes :

-   `agents.list[].groupChat.mentionPatterns` (Signal ne supporte pas les mentions natives).
-   `messages.groupChat.mentionPatterns` (repli global).
-   `messages.responsePrefix`.

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)