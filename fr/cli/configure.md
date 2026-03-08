title: "OpenClaw CLI Commande Configure Configuration Identifiants Appareils Agents"
description: "Apprenez à utiliser la commande CLI configure d'OpenClaw pour configurer les identifiants, les appareils, les paramètres par défaut des agents et les listes autorisées de modèles via un assistant interactif."
keywords: ["openclaw configure", "configuration cli", "paramètres par défaut des agents", "configuration des identifiants", "liste autorisée de modèles", "mode passerelle", "assistant cli", "config set"]
---

  Commandes CLI

  
# configure

Assistant interactif pour configurer les identifiants, les appareils et les paramètres par défaut des agents. Remarque : La section **Modèle** inclut désormais une sélection multiple pour la liste autorisée `agents.defaults.models` (ce qui apparaît dans `/model` et le sélecteur de modèles). Astuce : `openclaw config` sans sous-commande ouvre le même assistant. Utilisez `openclaw config get|set|unset` pour des modifications non interactives. Liens connexes :

-   Référence de configuration de la passerelle : [Configuration](../gateway/configuration.md)
-   CLI Config : [Config](./config.md)

Notes :

-   Le choix de l'emplacement d'exécution de la passerelle met toujours à jour `gateway.mode`. Vous pouvez sélectionner "Continuer" sans les autres sections si c'est tout ce dont vous avez besoin.
-   Les services orientés canaux (Slack/Discord/Matrix/Microsoft Teams) demandent les listes autorisées de canaux/salles pendant la configuration. Vous pouvez saisir des noms ou des ID ; l'assistant résout les noms en ID lorsque c'est possible.
-   Si vous exécutez l'étape d'installation du démon, l'authentification par jeton nécessite un jeton, et `gateway.auth.token` est géré par SecretRef. Configure valide le SecretRef mais ne persiste pas les valeurs de jeton en clair résolues dans les métadonnées d'environnement du service superviseur.
-   Si l'authentification par jeton nécessite un jeton et que le SecretRef de jeton configuré n'est pas résolu, configure bloque l'installation du démon avec des conseils d'action.
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, configure bloque l'installation du démon jusqu'à ce que le mode soit défini explicitement.

## Exemples

```bash
openclaw configure
openclaw configure --section model --section channels
```

[config](./config.md)[cron](./cron.md)

---