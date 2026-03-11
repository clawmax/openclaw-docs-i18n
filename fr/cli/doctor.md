

  Commandes CLI

  
# doctor

Vérifications de santé + corrections rapides pour la passerelle et les canaux. Liens utiles :

-   Dépannage : [Dépannage](../gateway/troubleshooting.md)
-   Audit de sécurité : [Sécurité](../gateway/security.md)

## Exemples

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes :

-   Les invites interactives (comme les corrections de trousseau de clés/OAuth) ne s'exécutent que lorsque stdin est un TTY et que `--non-interactive` n'est **pas** défini. Les exécutions sans interface (cron, Telegram, sans terminal) ignoreront les invites.
-   `--fix` (alias de `--repair`) écrit une sauvegarde dans `~/.openclaw/openclaw.json.bak` et supprime les clés de configuration inconnues, en listant chaque suppression.
-   Les vérifications d'intégrité de l'état détectent désormais les fichiers de transcription orphelins dans le répertoire des sessions et peuvent les archiver sous `.deleted.` pour récupérer de l'espace en toute sécurité.
-   Doctor inclut une vérification de préparation de la recherche en mémoire et peut recommander `openclaw configure --section model` lorsque les informations d'identification d'intégration sont manquantes.
-   Si le mode sandbox est activé mais que Docker n'est pas disponible, doctor signale un avertissement important avec une solution (`installer Docker` ou `openclaw config set agents.defaults.sandbox.mode off`).

## macOS : Surcharges d'environnement launchctl

Si vous avez précédemment exécuté `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (ou `...PASSWORD`), cette valeur écrase votre fichier de configuration et peut causer des erreurs persistantes de type « non autorisé ».

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[docs](./docs.md)[gateway](./gateway.md)