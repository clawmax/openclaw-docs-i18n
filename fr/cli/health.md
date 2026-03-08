

  Commandes CLI

  
# health

Récupère l'état de santé de la Passerelle en cours d'exécution.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes :

-   `--verbose` exécute des sondes en direct et affiche les délais par compte lorsque plusieurs comptes sont configurés.
-   La sortie inclut les magasins de session par agent lorsque plusieurs agents sont configurés.

[gateway](./gateway.md)[hooks](./hooks.md)

---