

  Configuration et opérations

  
# Contrat de Plan d'Application des Secrets

Cette page définit le contrat strict appliqué par `openclaw secrets apply`. Si une cible ne correspond pas à ces règles, l'application échoue avant de modifier la configuration.

## Forme du fichier de plan

`openclaw secrets apply --from <plan.json>` attend un tableau `targets` de cibles de plan :

```json
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
    {
      type: "auth-profiles.api_key.key",
      path: "profiles.openai:default.key",
      pathSegments: ["profiles", "openai:default", "key"],
      agentId: "main",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## Périmètre des cibles prises en charge

Les cibles de plan sont acceptées pour les chemins d'identification pris en charge dans :

-   [Surface d'Identification SecretRef](../reference/secretref-credential-surface.md)

## Comportement du type de cible

Règle générale :

-   `target.type` doit être reconnu et doit correspondre à la forme normalisée de `target.path`.

Les alias de compatibilité restent acceptés pour les plans existants :

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## Règles de validation des chemins

Chaque cible est validée avec toutes les règles suivantes :

-   `type` doit être un type de cible reconnu.
-   `path` doit être un chemin pointé non vide.
-   `pathSegments` peut être omis. S'il est fourni, il doit se normaliser exactement au même chemin que `path`.
-   Les segments interdits sont rejetés : `__proto__`, `prototype`, `constructor`.
-   Le chemin normalisé doit correspondre à la forme de chemin enregistrée pour le type de cible.
-   Si `providerId` ou `accountId` est défini, il doit correspondre à l'id encodé dans le chemin.
-   Les cibles `auth-profiles.json` nécessitent `agentId`.
-   Lors de la création d'un nouveau mappage `auth-profiles.json`, incluez `authProfileProvider`.

## Comportement en cas d'échec

Si une cible échoue la validation, l'application se termine avec une erreur du type :

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

Aucune écriture n'est validée pour un plan invalide.

## Notes sur le périmètre d'exécution et d'audit

-   Les entrées `auth-profiles.json` uniquement par référence (`keyRef`/`tokenRef`) sont incluses dans la résolution à l'exécution et la couverture d'audit.
-   `secrets apply` écrit les cibles `openclaw.json` prises en charge, les cibles `auth-profiles.json` prises en charge et les cibles de nettoyage optionnelles.

## Vérifications de l'opérateur

```bash
# Valider le plan sans écriture
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Puis appliquer pour de vrai
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

Si l'application échoue avec un message de chemin de cible invalide, régénérez le plan avec `openclaw secrets configure` ou corrigez le chemin de cible pour qu'il corresponde à une forme prise en charge ci-dessus.

## Documentation associée

-   [Gestion des Secrets](./secrets.md)
-   [CLI `secrets`](../cli/secrets.md)
-   [Surface d'Identification SecretRef](../reference/secretref-credential-surface.md)
-   [Référence de Configuration](./configuration-reference.md)

[Gestion des Secrets](./secrets.md)[Authentification par proxy de confiance](./trusted-proxy-auth.md)

---