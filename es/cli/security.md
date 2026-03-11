

  Comandos CLI

  
# security

Herramientas de seguridad (auditoría + correcciones opcionales). Relacionado:

-   Guía de seguridad: [Seguridad](../gateway/security.md)

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

La auditoría advierte cuando múltiples remitentes de DM comparten la sesión principal y recomienda el **modo seguro de DM**: `session.dmScope="per-channel-peer"` (o `per-account-channel-peer` para canales multi-cuenta) para bandejas de entrada compartidas. Esto es para el endurecimiento de bandejas de entrada cooperativas/compartidas. Un único Gateway compartido por operadores mutuamente no confiables/adversarios no es una configuración recomendada; divida los límites de confianza con gateways separados (o usuarios/hosts del SO separados). También emite `security.trust_model.multi_user_heuristic` cuando la configuración sugiere un ingreso probable de usuario compartido (por ejemplo, política de DM/grupo abierta, destinos de grupo configurados, o reglas de remitente con comodín), y le recuerda que OpenClaw es un modelo de confianza de asistente personal por defecto. Para configuraciones intencionales de usuario compartido, la guía de auditoría es aislar todas las sesiones, mantener el acceso al sistema de archivos limitado al área de trabajo, y mantener identidades o credenciales personales/privadas fuera de ese entorno de ejecución. También advierte cuando se usan modelos pequeños (`<=300B`) sin aislamiento y con herramientas web/navegador habilitadas. Para ingreso por webhook, advierte cuando `hooks.defaultSessionKey` no está establecido, cuando las anulaciones de `sessionKey` de solicitud están habilitadas, y cuando las anulaciones están habilitadas sin `hooks.allowedSessionKeyPrefixes`. También advierte cuando la configuración de Docker del sandbox está configurada mientras el modo sandbox está desactivado, cuando `gateway.nodes.denyCommands` usa entradas ineficaces similares a patrones/desconocidas (solo coincidencia exacta del nombre del comando del nodo, no filtrado de texto de shell), cuando `gateway.nodes.allowCommands` habilita explícitamente comandos de nodo peligrosos, cuando el perfil global `tools.profile="minimal"` es anulado por perfiles de herramientas del agente, cuando grupos abiertos exponen herramientas de tiempo de ejecución/sistema de archivos sin protecciones de sandbox/área de trabajo, y cuando las herramientas de extensiones de plugins instaladas pueden ser accesibles bajo una política de herramientas permisiva. También marca `gateway.allowRealIpFallback=true` (riesgo de suplantación de encabezados si los proxies están mal configurados) y `discovery.mdns.mode="full"` (fuga de metadatos a través de registros TXT de mDNS). También advierte cuando el navegador del sandbox usa la red Docker `bridge` sin `sandbox.browser.cdpSourceRange`. También marca modos de red Docker de sandbox peligrosos (incluyendo `host` y uniones de espacio de nombres `container:*`). También advierte cuando los contenedores Docker existentes del navegador sandbox tienen etiquetas hash faltantes/obsoletas (por ejemplo, contenedores previos a la migración que faltan `openclaw.browserConfigEpoch`) y recomienda `openclaw sandbox recreate --browser --all`. También advierte cuando los registros de instalación de plugins/hooks basados en npm no están fijos, carecen de metadatos de integridad, o se desvían de las versiones de paquetes actualmente instaladas. Advierte cuando las listas de permitidos de canales dependen de nombres/correos electrónicos/etiquetas mutables en lugar de IDs estables (Discord, Slack, Google Chat, MS Teams, Mattermost, ámbitos de IRC donde corresponda). Advierte cuando `gateway.auth.mode="none"` deja las APIs HTTP del Gateway accesibles sin un secreto compartido (`/tools/invoke` más cualquier endpoint `/v1/*` habilitado). Los ajustes con prefijo `dangerous`/`dangerously` son anulaciones explícitas de operador de último recurso; habilitar uno no es, por sí mismo, un informe de vulnerabilidad de seguridad. Para el inventario completo de parámetros peligrosos, consulte la sección "Resumen de banderas inseguras o peligrosas" en [Seguridad](../gateway/security.md).

## Salida JSON

Use `--json` para comprobaciones de CI/políticas:

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

Si se combinan `--fix` y `--json`, la salida incluye tanto las acciones de corrección como el informe final:

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## Qué cambia --fix

`--fix` aplica remediaciones seguras y deterministas:

-   cambia `groupPolicy="open"` común a `groupPolicy="allowlist"` (incluyendo variantes de cuenta en canales soportados)
-   establece `logging.redactSensitive` de `"off"` a `"tools"`
-   restringe permisos para estado/configuración y archivos sensibles comunes (`credentials/*.json`, `auth-profiles.json`, `sessions.json`, sesión `*.jsonl`)

`--fix` **no**:

-   rota tokens/contraseñas/claves API
-   deshabilita herramientas (`gateway`, `cron`, `exec`, etc.)
-   cambia elecciones de exposición de red/auth/enlace del gateway
-   elimina o reescribe plugins/habilidades

[secrets](./secrets.md)[sessions](./sessions.md)