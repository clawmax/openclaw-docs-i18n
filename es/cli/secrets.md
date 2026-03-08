

  Comandos CLI

  
# secrets

Usa `openclaw secrets` para gestionar SecretRefs y mantener saludable la instantánea activa del entorno de ejecución. Roles de los comandos:

-   `reload`: RPC de la puerta de enlace (`secrets.reload`) que vuelve a resolver las referencias e intercambia la instantánea del entorno de ejecución solo en caso de éxito total (sin escrituras de configuración).
-   `audit`: escaneo de solo lectura de los almacenes de configuración/autenticación/modelos generados y residuos heredados en busca de texto plano, referencias no resueltas y desviación de precedencia.
-   `configure`: planificador interactivo para la configuración del proveedor, mapeo de objetivos y verificación previa (se requiere TTY).
-   `apply`: ejecuta un plan guardado (`--dry-run` solo para validación) y luego limpia los residuos de texto plano objetivo.

Bucle operativo recomendado:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

Nota sobre el código de salida para CI/compuertas:

-   `audit --check` devuelve `1` si encuentra problemas.
-   las referencias no resueltas devuelven `2`.

Relacionado:

-   Guía de secretos: [Gestión de Secretos](../gateway/secrets.md)
-   Superficie de credenciales: [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md)
-   Guía de seguridad: [Seguridad](../gateway/security.md)

## Recargar instantánea del entorno de ejecución

Vuelve a resolver las referencias secretas e intercambia atómicamente la instantánea del entorno de ejecución.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

Notas:

-   Utiliza el método RPC de la puerta de enlace `secrets.reload`.
-   Si la resolución falla, la puerta de enlace mantiene la última instantánea buena conocida y devuelve un error (sin activación parcial).
-   La respuesta JSON incluye `warningCount`.

## Auditoría

Escanea el estado de OpenClaw en busca de:

-   almacenamiento de secretos en texto plano
-   referencias no resueltas
-   desviación de precedencia (credenciales de `auth-profiles.json` que ocultan referencias de `openclaw.json`)
-   residuos generados en `agents/*/agent/models.json` (valores `apiKey` del proveedor y encabezados sensibles del proveedor)
-   residuos heredados (entradas del almacén de autenticación heredado, recordatorios de OAuth)

Nota sobre residuos en encabezados:

-   La detección de encabezados sensibles del proveedor se basa en heurísticas de nombres (nombres comunes de encabezados de autenticación/credenciales y fragmentos como `authorization`, `x-api-key`, `token`, `secret`, `password` y `credential`).

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

Comportamiento de salida:

-   `--check` sale con código distinto de cero si encuentra problemas.
-   las referencias no resueltas salen con un código de error de mayor prioridad.

Aspectos destacados de la forma del informe:

-   `status`: `clean | findings | unresolved`
-   `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
-   códigos de hallazgos:
    -   `PLAINTEXT_FOUND`
    -   `REF_UNRESOLVED`
    -   `REF_SHADOWED`
    -   `LEGACY_RESIDUE`

## Configurar (asistente interactivo)

Construye cambios de proveedor y SecretRef de forma interactiva, ejecuta la verificación previa y opcionalmente aplica:

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

Flujo:

-   Primero configuración del proveedor (`add/edit/remove` para alias de `secrets.providers`).
-   Segundo, mapeo de credenciales (selecciona campos y asigna referencias `{source, provider, id}`).
-   Por último, verificación previa y aplicación opcional.

Banderas:

-   `--providers-only`: configura solo `secrets.providers`, omite el mapeo de credenciales.
-   `--skip-provider-setup`: omite la configuración del proveedor y mapea credenciales a proveedores existentes.
-   `--agent `: limita el descubrimiento de objetivos y las escrituras de `auth-profiles.json` a un almacén de agente.

Notas:

-   Requiere una TTY interactiva.
-   No se pueden combinar `--providers-only` con `--skip-provider-setup`.
-   `configure` se dirige a campos que contienen secretos en `openclaw.json` más `auth-profiles.json` para el ámbito del agente seleccionado.
-   `configure` admite la creación de nuevos mapeos en `auth-profiles.json` directamente en el flujo del selector.
-   Superficie compatible canónica: [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md).
-   Realiza una resolución de verificación previa antes de aplicar.
-   Los planes generados tienen habilitadas por defecto las opciones de limpieza (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson`).
-   La ruta de aplicación es unidireccional para los valores en texto plano limpiados.
-   Sin `--apply`, la CLI aún pregunta `¿Aplicar este plan ahora?` después de la verificación previa.
-   Con `--apply` (y sin `--yes`), la CLI solicita una confirmación irreversible adicional.

Nota de seguridad del proveedor exec:

-   Las instalaciones de Homebrew a menudo exponen binarios con enlaces simbólicos bajo `/opt/homebrew/bin/*`.
-   Establece `allowSymlinkCommand: true` solo cuando sea necesario para rutas confiables del gestor de paquetes, y combínalo con `trustedDirs` (por ejemplo `["/opt/homebrew"]`).
-   En Windows, si la verificación de ACL no está disponible para una ruta de proveedor, OpenClaw falla cerrado. Solo para rutas confiables, establece `allowInsecurePath: true` en ese proveedor para omitir las comprobaciones de seguridad de ruta.

## Aplicar un plan guardado

Aplica o realiza una verificación previa de un plan generado anteriormente:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

Detalles del contrato del plan (rutas de destino permitidas, reglas de validación y semántica de fallos):

-   [Contrato del Plan de Aplicación de Secretos](../gateway/secrets-plan-contract.md)

Lo que `apply` puede actualizar:

-   `openclaw.json` (objetivos SecretRef + inserciones/eliminaciones de proveedor)
-   `auth-profiles.json` (limpieza de objetivos del proveedor)
-   residuos heredados de `auth.json`
-   `~/.openclaw/.env` claves secretas conocidas cuyos valores fueron migrados

## Por qué no hay copias de seguridad para revertir

`secrets apply` intencionalmente no escribe copias de seguridad de reversión que contengan los antiguos valores en texto plano. La seguridad proviene de una verificación previa estricta + una aplicación casi atómica con restauración en memoria de mejor esfuerzo en caso de fallo.

## Ejemplo

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

Si `audit --check` aún reporta hallazgos en texto plano, actualiza las rutas objetivo restantes reportadas y vuelve a ejecutar la auditoría.

[CLI Sandbox](./sandbox.md)[seguridad](./security.md)