

  Configuración y operaciones

  
# Contrato de Plan de Aplicación de Secretos

Esta página define el contrato estricto que aplica `openclaw secrets apply`. Si un objetivo no coincide con estas reglas, la aplicación falla antes de mutar la configuración.

## Formato del archivo de plan

`openclaw secrets apply --from <plan.json>` espera un array `targets` de objetivos de plan:

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

## Alcance de objetivos admitidos

Se aceptan objetivos de plan para rutas de credenciales admitidas en:

-   [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md)

## Comportamiento del tipo de objetivo

Regla general:

-   `target.type` debe ser reconocido y debe coincidir con la forma normalizada de `target.path`.

Los alias de compatibilidad siguen siendo aceptados para planes existentes:

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## Reglas de validación de ruta

Cada objetivo se valida con todo lo siguiente:

-   `type` debe ser un tipo de objetivo reconocido.
-   `path` debe ser una ruta de puntos no vacía.
-   `pathSegments` se puede omitir. Si se proporciona, debe normalizarse exactamente a la misma ruta que `path`.
-   Los segmentos prohibidos son rechazados: `__proto__`, `prototype`, `constructor`.
-   La ruta normalizada debe coincidir con la forma de ruta registrada para el tipo de objetivo.
-   Si `providerId` o `accountId` está configurado, debe coincidir con el id codificado en la ruta.
-   Los objetivos de `auth-profiles.json` requieren `agentId`.
-   Al crear un nuevo mapeo en `auth-profiles.json`, incluya `authProfileProvider`.

## Comportamiento ante fallos

Si un objetivo falla la validación, la aplicación termina con un error como:

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

No se confirma ninguna escritura para un plan no válido.

## Notas sobre el alcance de ejecución y auditoría

-   Las entradas de `auth-profiles.json` solo de referencia (`keyRef`/`tokenRef`) se incluyen en la resolución en tiempo de ejecución y la cobertura de auditoría.
-   `secrets apply` escribe objetivos admitidos de `openclaw.json`, objetivos admitidos de `auth-profiles.json` y objetivos opcionales de limpieza.

## Comprobaciones del operador

```bash
# Validar plan sin escrituras
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Luego aplicar realmente
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

Si la aplicación falla con un mensaje de ruta de objetivo no válida, regenere el plan con `openclaw secrets configure` o corrija la ruta del objetivo a una forma admitida mencionada anteriormente.

## Documentación relacionada

-   [Gestión de Secretos](./secrets.md)
-   [CLI `secrets`](../cli/secrets.md)
-   [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md)
-   [Referencia de Configuración](./configuration-reference.md)

[Gestión de Secretos](./secrets.md)[Autenticación de proxy de confianza](./trusted-proxy-auth.md)