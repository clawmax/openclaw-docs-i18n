

  Commandes CLI

  
# status

Diagnostics pour les canaux et les sessions.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes :

-   `--deep` exécute des sondages en direct (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
-   La sortie inclut les magasins de session par agent lorsque plusieurs agents sont configurés.
-   L'aperçu inclut l'état d'installation/exécution du service hôte Gateway + node lorsqu'il est disponible.
-   L'aperçu inclut le canal de mise à jour + le SHA git (pour les installations depuis les sources).
-   Les informations de mise à jour apparaissent dans l'Aperçu ; si une mise à jour est disponible, status affiche une indication pour exécuter `openclaw update` (voir [Mise à jour](../install/updating.md)).
-   Les surfaces d'état en lecture seule (`status`, `status --json`, `status --all`) résolvent les SecretRefs pris en charge pour leurs chemins de configuration ciblés lorsque cela est possible.
-   Si un SecretRef de canal pris en charge est configuré mais indisponible dans le chemin de commande actuel, status reste en lecture seule et rapporte une sortie dégradée au lieu de planter. La sortie humaine affiche des avertissements tels que "jeton configuré indisponible dans ce chemin de commande", et la sortie JSON inclut `secretDiagnostics`.

[skills](./skills.md)[system](./system.md)

---