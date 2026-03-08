

  Configuración

  
# Conmutación por Error de Modelo

OpenClaw maneja los fallos en dos etapas:

1.  **Rotación del perfil de autenticación** dentro del proveedor actual.
2.  **Conmutación por error del modelo** al siguiente modelo en `agents.defaults.model.fallbacks`.

Este documento explica las reglas en tiempo de ejecución y los datos que las respaldan.

## Almacenamiento de autenticación (claves + OAuth)

OpenClaw utiliza **perfiles de autenticación** tanto para claves API como para tokens OAuth.

-   Los secretos residen en `~/.openclaw/agents//agent/auth-profiles.json` (heredado: `~/.openclaw/agent/auth-profiles.json`).
-   La configuración `auth.profiles` / `auth.order` es **solo metadatos y enrutamiento** (sin secretos).
-   Archivo OAuth heredado solo para importación: `~/.openclaw/credentials/oauth.json` (importado a `auth-profiles.json` en el primer uso).

Más detalles: [/concepts/oauth](./oauth.md) Tipos de credenciales:

-   `type: "api_key"` → `{ provider, key }`
-   `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` para algunos proveedores)

## IDs de Perfil

Los inicios de sesión OAuth crean perfiles distintos para que puedan coexistir múltiples cuentas.

-   Por defecto: `provider:default` cuando no hay correo electrónico disponible.
-   OAuth con correo electrónico: `provider:` (por ejemplo `google-antigravity:user@gmail.com`).

Los perfiles residen en `~/.openclaw/agents//agent/auth-profiles.json` bajo `profiles`.

## Orden de rotación

Cuando un proveedor tiene múltiples perfiles, OpenClaw elige un orden como este:

1.  **Configuración explícita**: `auth.order[provider]` (si está configurado).
2.  **Perfiles configurados**: `auth.profiles` filtrados por proveedor.
3.  **Perfiles almacenados**: entradas en `auth-profiles.json` para el proveedor.

Si no se configura un orden explícito, OpenClaw utiliza un orden round‑robin:

-   **Clave primaria:** tipo de perfil (**OAuth antes que claves API**).
-   **Clave secundaria:** `usageStats.lastUsed` (el más antiguo primero, dentro de cada tipo).
-   Los **perfiles en periodo de espera/deshabilitados** se mueven al final, ordenados por la expiración más próxima.

### Persistencia de sesión (amigable con la caché)

OpenClaw **fija el perfil de autenticación elegido por sesión** para mantener calientes las cachés del proveedor. **No** rota en cada solicitud. El perfil fijado se reutiliza hasta que:

-   la sesión se reinicia (`/new` / `/reset`)
-   se completa una compactación (el contador de compactación se incrementa)
-   el perfil está en periodo de espera/deshabilitado

La selección manual mediante `/model …@` establece una **sobreescritura del usuario** para esa sesión y no se rota automáticamente hasta que comience una nueva sesión. Los perfiles fijados automáticamente (seleccionados por el enrutador de sesión) se tratan como una **preferencia**: se intentan primero, pero OpenClaw puede rotar a otro perfil ante límites de tasa/timeouts. Los perfiles fijados por el usuario permanecen bloqueados a ese perfil; si falla y hay conmutaciones por error de modelo configuradas, OpenClaw pasa al siguiente modelo en lugar de cambiar de perfil.

### Por qué OAuth puede "parecer perdido"

Si tienes tanto un perfil OAuth como un perfil de clave API para el mismo proveedor, el round‑robin puede cambiar entre ellos a través de los mensajes a menos que estén fijados. Para forzar un solo perfil:

-   Fíjalo con `auth.order[provider] = ["provider:profileId"]`, o
-   Usa una sobreescritura por sesión mediante `/model …` con una anulación de perfil (cuando sea compatible con tu interfaz/superficie de chat).

## Períodos de espera

Cuando un perfil falla debido a errores de autenticación/límite de tasa (o un timeout que parece limitación de tasa), OpenClaw lo marca en periodo de espera y pasa al siguiente perfil. Los errores de formato/solicitud no válida (por ejemplo, fallos de validación de ID de llamada a herramientas de Cloud Code Assist) se tratan como merecedores de conmutación por error y usan los mismos períodos de espera. Los erroes compatibles con OpenAI de razón de parada como `Unhandled stop reason: error`, `stop reason: error`, y `reason: error` se clasifican como señales de timeout/conmutación por error. Los períodos de espera usan retroceso exponencial:

-   1 minuto
-   5 minutos
-   25 minutos
-   1 hora (límite)

El estado se almacena en `auth-profiles.json` bajo `usageStats`:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## Deshabilitaciones por facturación

Los fallos de facturación/crédito (por ejemplo, "créditos insuficientes" / "saldo de crédito demasiado bajo") se tratan como merecedores de conmutación por error, pero normalmente no son transitorios. En lugar de un período de espera corto, OpenClaw marca el perfil como **deshabilitado** (con un retroceso más largo) y rota al siguiente perfil/proveedor. El estado se almacena en `auth-profiles.json`:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Valores por defecto:

-   El retroceso por facturación comienza en **5 horas**, se duplica por cada fallo de facturación y tiene un límite de **24 horas**.
-   Los contadores de retroceso se reinician si el perfil no ha fallado durante **24 horas** (configurable).

## Conmutación por error de modelo

Si todos los perfiles de un proveedor fallan, OpenClaw pasa al siguiente modelo en `agents.defaults.model.fallbacks`. Esto se aplica a fallos de autenticación, límites de tasa y timeouts que agotaron la rotación de perfiles (otros errores no avanzan la conmutación por error). Cuando una ejecución comienza con una anulación de modelo (hooks o CLI), las conmutaciones por error aún terminan en `agents.defaults.model.primary` después de intentar cualquier conmutación configurada.

## Configuración relacionada

Consulta [Configuración de Gateway](../gateway/configuration.md) para:

-   `auth.profiles` / `auth.order`
-   `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
-   `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
-   `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
-   enrutamiento de `agents.defaults.imageModel`

Consulta [Modelos](./models.md) para obtener una visión general más amplia de la selección de modelos y la conmutación por error.

[Proveedores de Modelos](./model-providers.md)[Anthropic](../providers/anthropic.md)