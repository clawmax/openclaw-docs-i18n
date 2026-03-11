

  Environnement et débogage

  
# Scripts

Le répertoire `scripts/` contient des scripts d'aide pour les workflows locaux et les tâches opérationnelles. Utilisez-les lorsqu'une tâche est clairement liée à un script ; sinon, préférez l'interface CLI.

## Conventions

-   Les scripts sont **optionnels** sauf s'ils sont référencés dans la documentation ou les listes de contrôle de version.
-   Préférez les interfaces CLI lorsqu'elles existent (exemple : la surveillance d'authentification utilise `openclaw models status --check`).
-   Supposez que les scripts sont spécifiques à l'hôte ; lisez-les avant de les exécuter sur une nouvelle machine.

## Scripts de surveillance d'authentification

Les scripts de surveillance d'authentification sont documentés ici : [/automation/auth-monitoring](../automation/auth-monitoring.md)

## Lors de l'ajout de scripts

-   Gardez les scripts ciblés et documentés.
-   Ajoutez une brève entrée dans la documentation pertinente (ou créez-en une si elle manque).

[Tests](./testing.md)[Plantage Node + tsx](../debug/node-issue.md)

---