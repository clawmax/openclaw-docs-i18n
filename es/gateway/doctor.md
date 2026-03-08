

  Configuración y operaciones

  
# Doctor

`openclaw doctor` es la herramienta de reparación y migración para OpenClaw. Corrige configuraciones/estados obsoletos, verifica el estado del sistema y proporciona pasos de reparación accionables.

## Inicio rápido

```bash
openclaw doctor
```

### Sin interfaz / automatización

```bash
openclaw doctor --yes
```

Acepta los valores predeterminados sin solicitar confirmación (incluye pasos de reinicio/servicio/reparación del sandbox cuando corresponda).

```bash
openclaw doctor --repair
```

Aplica las reparaciones recomendadas sin solicitar confirmación (reparaciones + reinicios donde sea seguro).

```bash
openclaw doctor --repair --force
```

Aplica también reparaciones agresivas (sobrescribe configuraciones personalizadas del supervisor).

```bash
openclaw doctor --non-interactive
```

Ejecuta sin solicitudes de confirmación y solo aplica migraciones seguras (normalización de configuración + movimientos de estado en disco). Omite acciones de reinicio/servicio/sandbox que requieren confirmación humana. Las migraciones de estado heredado se ejecutan automáticamente cuando se detectan.

```bash
openclaw doctor --deep
```

Escanea los servicios del sistema en busca de instalaciones adicionales del gateway (launchd/systemd/schtasks). Si deseas revisar los cambios antes de escribirlos, abre primero el archivo de configuración:

```bash
cat ~/.openclaw/openclaw.json
```

## Qué hace (resumen)

-   Actualización opcional previa al vuelo para instalaciones git (solo interactivo).
-   Verificación de frescura del protocolo de UI (reconstruye la UI de Control cuando el esquema del protocolo es más nuevo).
-   Verificación de estado + solicitud de reinicio.
-   Resumen del estado de las habilidades (elegibles/faltantes/bloqueadas).
-   Normalización de configuración para valores heredados.
-   Advertencias de anulación del proveedor OpenCode Zen (`models.providers.opencode`).
-   Migración de estado heredado en disco (sesiones/directorio del agente/autenticación de WhatsApp).
-   Verificaciones de integridad y permisos del estado (sesiones, transcripciones, directorio de estado).
-   Verificaciones de permisos del archivo de configuración (chmod 600) cuando se ejecuta localmente.
-   Estado de autenticación del modelo: verifica la caducidad de OAuth, puede refrescar tokens próximos a caducar e informa estados de deshabilitación/enfriamiento del perfil de autenticación.
-   Detección de directorios de espacio de trabajo adicionales (`~/openclaw`).
-   Reparación de la imagen del sandbox cuando el sandboxing está habilitado.
-   Migración de servicios heredados y detección de gateways adicionales.
-   Verificaciones del tiempo de ejecución del gateway (servicio instalado pero no en ejecución; etiqueta launchd en caché).
-   Advertencias de estado de los canales (sondeados desde el gateway en ejecución).
-   Auditoría de configuración del supervisor (launchd/systemd/schtasks) con reparación opcional.
-   Verificaciones de mejores prácticas del tiempo de ejecución del gateway (Node vs Bun, rutas de gestores de versiones).
-   Diagnóstico de colisiones de puertos del gateway (predeterminado `18789`).
-   Advertencias de seguridad para políticas de DM abiertas.
-   Verificaciones de autenticación del gateway para el modo de token local (ofrece generación de token cuando no existe una fuente de token; no sobrescribe configuraciones de token SecretRef).
-   Verificación de systemd linger en Linux.
-   Verificaciones de instalación desde fuente (incompatibilidad del espacio de trabajo pnpm, activos de UI faltantes, binario tsx faltante).
-   Escribe la configuración actualizada + metadatos del asistente.

## Comportamiento detallado y justificación

### 0) Actualización opcional (instalaciones git)

Si se trata de un repositorio git y doctor se ejecuta de forma interactiva, ofrece actualizar (fetch/rebase/build) antes de ejecutar doctor.

### 1) Normalización de configuración

Si la configuración contiene formas de valores heredados (por ejemplo, `messages.ackReaction` sin una anulación específica del canal), doctor los normaliza al esquema actual.

### 2) Migraciones de claves de configuración heredadas

Cuando la configuración contiene claves obsoletas, otros comandos se niegan a ejecutarse y piden que ejecutes `openclaw doctor`. Doctor hará:

-   Explicará qué claves heredadas se encontraron.
-   Mostrará la migración que aplicó.
-   Reescribirá `~/.openclaw/openclaw.json` con el esquema actualizado.

El Gateway también ejecuta automáticamente las migraciones de doctor al iniciar cuando detecta un formato de configuración heredado, por lo que las configuraciones obsoletas se reparan sin intervención manual. Migraciones actuales:

-   `routing.allowFrom` → `channels.whatsapp.allowFrom`
-   `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
-   `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
-   `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
-   `routing.queue` → `messages.queue`
-   `routing.bindings` → `bindings` de nivel superior
-   `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
-   `routing.agentToAgent` → `tools.agentToAgent`
-   `routing.transcribeAudio` → `tools.media.audio.models`
-   `bindings[].match.accountID` → `bindings[].match.accountId`
-   Para canales con `accounts` nombrados pero sin `accounts.default`, mover los valores de canal de nivel superior de cuenta única al ámbito de la cuenta a `channels..accounts.default` cuando estén presentes
-   `identity` → `agents.list[].identity`
-   `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
-   `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
-   `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

Las advertencias de doctor también incluyen orientación sobre la cuenta predeterminada para canales multi-cuenta:

-   Si se configuran dos o más entradas `channels..accounts` sin `channels..defaultAccount` o `accounts.default`, doctor advierte que el enrutamiento de respaldo puede elegir una cuenta inesperada.
-   Si `channels..defaultAccount` está configurado con un ID de cuenta desconocido, doctor advierte y enumera los ID de cuenta configurados.

### 2b) Anulaciones del proveedor OpenCode Zen

Si has agregado `models.providers.opencode` (o `opencode-zen`) manualmente, anula el catálogo integrado de OpenCode Zen de `@mariozechner/pi-ai`. Eso puede forzar a cada modelo a usar una única API o anular los costos. Doctor advierte para que puedas eliminar la anulación y restaurar el enrutamiento de API por modelo + costos.

### 3) Migraciones de estado heredado (disposición en disco)

Doctor puede migrar disposiciones en disco antiguas a la estructura actual:

-   Almacén de sesiones + transcripciones:
    -   de `~/.openclaw/sessions/` a `~/.openclaw/agents//sessions/`
-   Directorio del agente:
    -   de `~/.openclaw/agent/` a `~/.openclaw/agents//agent/`
-   Estado de autenticación de WhatsApp (Baileys):
    -   de `~/.openclaw/credentials/*.json` heredado (excepto `oauth.json`)
    -   a `~/.openclaw/credentials/whatsapp//...` (ID de cuenta predeterminado: `default`)

Estas migraciones son de mejor esfuerzo e idempotentes; doctor emitirá advertencias cuando deje carpetas heredadas como respaldo. El Gateway/CLI también migra automáticamente las sesiones heredadas + el directorio del agente al iniciar, por lo que el historial/autenticación/modelos terminan en la ruta por agente sin necesidad de ejecutar doctor manualmente. La autenticación de WhatsApp se migra intencionalmente solo mediante `openclaw doctor`.

### 4) Verificaciones de integridad del estado (persistencia de sesiones, enrutamiento y seguridad)

El directorio de estado es el tronco cerebral operativo. Si desaparece, pierdes sesiones, credenciales, registros y configuración (a menos que tengas copias de seguridad en otro lugar). Doctor verifica:

-   **Directorio de estado faltante**: advierte sobre la pérdida catastrófica de estado, solicita recrear el directorio y recuerda que no puede recuperar datos faltantes.
-   **Permisos del directorio de estado**: verifica la capacidad de escritura; ofrece reparar permisos (y emite una sugerencia `chown` cuando se detecta una discrepancia de propietario/grupo).
-   **Directorio de estado sincronizado en la nube de macOS**: advierte cuando el estado se resuelve bajo iCloud Drive (`~/Library/Mobile Documents/com~apple~CloudDocs/...`) o `~/Library/CloudStorage/...` porque las rutas respaldadas por sincronización pueden causar E/S más lentas y carreras de bloqueo/sincronización.
-   **Directorio de estado en SD o eMMC en Linux**: advierte cuando el estado se resuelve en un origen de montaje `mmcblk*`, porque las E/S aleatorias respaldadas por SD o eMMC pueden ser más lentas y desgastarse más rápido bajo escrituras de sesiones y credenciales.
-   **Directorios de sesiones faltantes**: `sessions/` y el directorio del almacén de sesiones son necesarios para persistir el historial y evitar fallos `ENOENT`.
-   **Desajuste de transcripción**: advierte cuando las entradas de sesión recientes tienen archivos de transcripción faltantes.
-   **"JSONL de 1 línea" de la sesión principal**: marca cuando la transcripción principal tiene solo una línea (el historial no se está acumulando).
-   **Múltiples directorios de estado**: advierte cuando existen múltiples carpetas `~/.openclaw` en diferentes directorios de inicio o cuando `OPENCLAW_STATE_DIR` apunta a otro lugar (el historial puede dividirse entre instalaciones).
-   **Recordatorio de modo remoto**: si `gateway.mode=remote`, doctor recuerda ejecutarlo en el host remoto (el estado reside allí).
-   **Permisos del archivo de configuración**: advierte si `~/.openclaw/openclaw.json` es legible por grupo/mundo y ofrece restringirlo a `600`.

### 5) Estado de autenticación del modelo (caducidad de OAuth)

Doctor inspecciona los perfiles OAuth en el almacén de autenticación, advierte cuando los tokens están por caducar/caducados y puede refrescarlos cuando sea seguro. Si el perfil de Anthropic Claude Code está obsoleto, sugiere ejecutar `claude setup-token` (o pegar un setup-token). Las solicitudes de refresco solo aparecen cuando se ejecuta de forma interactiva (TTY); `--non-interactive` omite los intentos de refresco. Doctor también informa perfiles de autenticación que son temporalmente inutilizables debido a:

-   períodos de enfriamiento cortos (límites de tasa/timeouts/fallos de autenticación)
-   deshabilitaciones más largas (fallos de facturación/crédito)

### 6) Validación del modelo de hooks

Si `hooks.gmail.model` está configurado, doctor valida la referencia del modelo contra el catálogo y la lista de permitidos y advierte cuando no se resolverá o no está permitido.

### 7) Reparación de la imagen del sandbox

Cuando el sandboxing está habilitado, doctor verifica las imágenes de Docker y ofrece construirlas o cambiar a nombres heredados si falta la imagen actual.

### 8) Migraciones de servicios del gateway y sugerencias de limpieza

Doctor detecta servicios de gateway heredados (launchd/systemd/schtasks) y ofrece eliminarlos e instalar el servicio OpenClaw usando el puerto actual del gateway. También puede escanear en busca de servicios adicionales similares a gateway e imprimir sugerencias de limpieza. Los servicios de gateway OpenClaw con nombre de perfil se consideran de primera clase y no se marcan como "extra".

### 9) Advertencias de seguridad

Doctor emite advertencias cuando un proveedor está abierto a DMs sin una lista de permitidos, o cuando una política está configurada de manera peligrosa.

### 10) systemd linger (Linux)

Si se ejecuta como un servicio de usuario de systemd, doctor asegura que lingering esté habilitado para que el gateway permanezca activo después del cierre de sesión.

### 11) Estado de las habilidades

Doctor imprime un resumen rápido de las habilidades elegibles/faltantes/bloqueadas para el espacio de trabajo actual.

### 12) Verificaciones de autenticación del gateway (token local)

Doctor verifica la preparación de la autenticación por token local del gateway.

-   Si el modo de token necesita un token y no existe una fuente de token, doctor ofrece generar uno.
-   Si `gateway.auth.token` está gestionado por SecretRef pero no está disponible, doctor advierte y no lo sobrescribe con texto plano.
-   `openclaw doctor --generate-gateway-token` fuerza la generación solo cuando no hay una configuración de token SecretRef.

### 12b) Reparaciones conscientes de SecretRef de solo lectura

Algunos flujos de reparación necesitan inspeccionar credenciales configuradas sin debilitar el comportamiento de fallo rápido del tiempo de ejecución.

-   `openclaw doctor --fix` ahora usa el mismo modelo de resumen de SecretRef de solo lectura que los comandos de la familia status para reparaciones de configuración específicas.
-   Ejemplo: la reparación de `allowFrom` / `groupAllowFrom` `@username` de Telegram intenta usar las credenciales configuradas del bot cuando están disponibles.
-   Si el token del bot de Telegram está configurado mediante SecretRef pero no está disponible en la ruta del comando actual, doctor informa que la credencial está configurada-pero-no-disponible y omite la resolución automática en lugar de fallar o informar incorrectamente que el token falta.

### 13) Verificación de estado del gateway + reinicio

Doctor ejecuta una verificación de estado y ofrece reiniciar el gateway cuando parece no estar saludable.

### 14) Advertencias de estado del canal

Si el gateway está saludable, doctor ejecuta un sondeo del estado del canal e informa advertencias con correcciones sugeridas.

### 15) Auditoría de configuración del supervisor + reparación

Doctor verifica la configuración del supervisor instalado (launchd/systemd/schtasks) en busca de valores predeterminados faltantes o desactualizados (por ejemplo, dependencias de network-online de systemd y retraso de reinicio). Cuando encuentra una discrepancia, recomienda una actualización y puede reescribir el archivo/tarea del servicio a los valores predeterminados actuales. Notas:

-   `openclaw doctor` solicita confirmación antes de reescribir la configuración del supervisor.
-   `openclaw doctor --yes` acepta las solicitudes de reparación predeterminadas.
-   `openclaw doctor --repair` aplica las correcciones recomendadas sin solicitudes de confirmación.
-   `openclaw doctor --repair --force` sobrescribe configuraciones personalizadas del supervisor.
-   Si la autenticación por token requiere un token y `gateway.auth.token` está gestionado por SecretRef, la instalación/reparación del servicio doctor valida el SecretRef pero no persiste los valores de token resueltos en texto plano en los metadatos del entorno del servicio supervisor.
-   Si la autenticación por token requiere un token y el SecretRef de token configurado no está resuelto, doctor bloquea la ruta de instalación/reparación con orientación accionable.
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está configurado, doctor bloquea la instalación/reparación hasta que el modo se establezca explícitamente.
-   Siempre puedes forzar una reescritura completa mediante `openclaw gateway install --force`.

### 16) Diagnóstico del tiempo de ejecución del gateway + puertos

Doctor inspecciona el tiempo de ejecución del servicio (PID, último estado de salida) y advierte cuando el servicio está instalado pero no se está ejecutando realmente. También verifica colisiones de puertos en el puerto del gateway (predeterminado `18789`) e informa causas probables (gateway ya en ejecución, túnel SSH).

### 17) Mejores prácticas del tiempo de ejecución del gateway

Doctor advierte cuando el servicio del gateway se ejecuta en Bun o en una ruta de Node gestionada por versiones (`nvm`, `fnm`, `volta`, `asdf`, etc.). Los canales de WhatsApp + Telegram requieren Node, y las rutas de gestores de versiones pueden romperse después de las actualizaciones porque el servicio no carga la inicialización de tu shell. Doctor ofrece migrar a una instalación de Node del sistema cuando esté disponible (Homebrew/apt/choco).

### 18) Escritura de configuración + metadatos del asistente

Doctor persiste cualquier cambio de configuración y estampa metadatos del asistente para registrar la ejecución de doctor.

### 19) Consejos del espacio de trabajo (copia de seguridad + sistema de memoria)

Doctor sugiere un sistema de memoria del espacio de trabajo cuando falta e imprime un consejo de copia de seguridad si el espacio de trabajo no está bajo git. Consulta [/concepts/agent-workspace](../concepts/agent-workspace.md) para una guía completa de la estructura del espacio de trabajo y copia de seguridad con git (se recomienda GitHub o GitLab privado).

[Heartbeat](./heartbeat.md)[Logging](./logging.md)