

  Configuration

  
# Dépannage des canaux

Utilisez cette page lorsqu'un canal se connecte mais que son comportement est incorrect.

## Échelle de commandes

Exécutez-les d'abord dans l'ordre :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

État sain de base :

-   `Runtime: running`
-   `RPC probe: ok`
-   La sonde du canal affiche connecté/prêt

## WhatsApp

### Signatures d'échec WhatsApp

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Connecté mais pas de réponses en MP | `openclaw pairing list whatsapp` | Approuvez l'expéditeur ou changez la politique de MP/liste autorisée. |
| Messages de groupe ignorés | Vérifiez `requireMention` + modèles de mention dans la config | Mentionnez le bot ou assouplissez la politique de mention pour ce groupe. |
| Boucles de déconnexion/reconnexion aléatoires | `openclaw channels status --probe` + logs | Reconnectez-vous et vérifiez que le répertoire des identifiants est sain. |

Dépannage complet : [/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### Signatures d'échec Telegram

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| `/start` mais pas de flux de réponse utilisable | `openclaw pairing list telegram` | Approuvez l'appairage ou changez la politique de MP. |
| Bot en ligne mais le groupe reste silencieux | Vérifiez l'exigence de mention et le mode confidentialité du bot | Désactivez le mode confidentialité pour la visibilité du groupe ou mentionnez le bot. |
| Échecs d'envoi avec erreurs réseau | Inspectez les logs pour les échecs d'appel API Telegram | Corrigez le routage DNS/IPv6/proxy vers `api.telegram.org`. |
| Mise à niveau et liste autorisée vous bloque | `openclaw security audit` et listes autorisées de config | Exécutez `openclaw doctor --fix` ou remplacez `@username` par des ID d'expéditeur numériques. |

Dépannage complet : [/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### Signatures d'échec Discord

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Bot en ligne mais pas de réponses dans la guilde | `openclaw channels status --probe` | Autorisez la guilde/le canal et vérifiez l'intention de contenu des messages. |
| Messages de groupe ignorés | Vérifiez les logs pour les rejets de mention | Mentionnez le bot ou définissez `requireMention: false` pour la guilde/le canal. |
| Réponses en MP manquantes | `openclaw pairing list discord` | Approuvez l'appairage MP ou ajustez la politique de MP. |

Dépannage complet : [/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### Signatures d'échec Slack

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Mode socket connecté mais pas de réponses | `openclaw channels status --probe` | Vérifiez le jeton d'application + le jeton du bot et les portées requises. |
| MPs bloqués | `openclaw pairing list slack` | Approuvez l'appairage ou assouplissez la politique de MP. |
| Message de canal ignoré | Vérifiez `groupPolicy` et la liste autorisée du canal | Autorisez le canal ou passez à la politique `open`. |

Dépannage complet : [/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage et BlueBubbles

### Signatures d'échec iMessage et BlueBubbles

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Aucun événement entrant | Vérifiez l'accessibilité du webhook/du serveur et les permissions de l'application | Corrigez l'URL du webhook ou l'état du serveur BlueBubbles. |
| Peut envoyer mais pas recevoir sur macOS | Vérifiez les permissions de confidentialité macOS pour l'automatisation de Messages | Réattribuez les permissions TCC et redémarrez le processus du canal. |
| Expéditeur en MP bloqué | `openclaw pairing list imessage` ou `openclaw pairing list bluebubbles` | Approuvez l'appairage ou mettez à jour la liste autorisée. |

Dépannage complet :

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### Signatures d'échec Signal

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Démon accessible mais bot silencieux | `openclaw channels status --probe` | Vérifiez l'URL/le compte du démon `signal-cli` et le mode de réception. |
| MP bloqué | `openclaw pairing list signal` | Approuvez l'expéditeur ou ajustez la politique de MP. |
| Les réponses de groupe ne se déclenchent pas | Vérifiez la liste autorisée du groupe et les modèles de mention | Ajoutez l'expéditeur/le groupe ou assouplissez la restriction. |

Dépannage complet : [/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### Signatures d'échec Matrix

| Symptôme | Vérification la plus rapide | Correction |
| --- | --- | --- |
| Connecté mais ignore les messages du salon | `openclaw channels status --probe` | Vérifiez `groupPolicy` et la liste autorisée du salon. |
| Les MPs ne sont pas traités | `openclaw pairing list matrix` | Approuvez l'expéditeur ou ajustez la politique de MP. |
| Les salons chiffrés échouent | Vérifiez le module de chiffrement et les paramètres de chiffrement | Activez le support du chiffrement et rejoignez/resynchronisez le salon. |

Dépannage complet : [/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[Analyse de l'emplacement des canaux](./location.md)

---