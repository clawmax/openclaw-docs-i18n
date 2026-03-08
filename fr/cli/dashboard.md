

  Commandes CLI

  
# dashboard

Ouvrez l'interface de contrôle en utilisant votre authentification actuelle.

```bash
openclaw dashboard
openclaw dashboard --no-open
```

Notes :

-   `dashboard` résout les SecretRefs configurés pour `gateway.auth.token` lorsque c'est possible.
-   Pour les jetons gérés par SecretRef (résolus ou non), `dashboard` imprime/copie/ouvre une URL non tokenisée pour éviter d'exposer des secrets externes dans la sortie du terminal, l'historique du presse-papiers ou les arguments de lancement du navigateur.
-   Si `gateway.auth.token` est géré par SecretRef mais non résolu dans ce chemin de commande, la commande imprime une URL non tokenisée et des instructions de correction explicites au lieu d'intégrer un espace réservé de jeton invalide.

[daemon](./daemon.md)[devices](./devices.md)