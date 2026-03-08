

  Mantenimiento

  
# Guía de Migración

Esta guía migra un OpenClaw Gateway de una máquina a otra **sin rehacer la configuración inicial**. La migración es conceptualmente simple:

-   Copia el **directorio de estado** (`$OPENCLAW_STATE_DIR`, por defecto: `~/.openclaw/`) — esto incluye configuración, autenticación, sesiones y estado de los canales.
-   Copia tu **espacio de trabajo** (`~/.openclaw/workspace/` por defecto) — esto incluye tus archivos de agente (memoria, prompts, etc.).

Pero hay errores comunes relacionados con **perfiles**, **permisos** y **copias parciales**.

## Antes de empezar (qué estás migrando)

### 1) Identifica tu directorio de estado

La mayoría de las instalaciones usan el predeterminado:

-   **Directorio de estado:** `~/.openclaw/`

Pero puede ser diferente si usas:

-   `--profile ` (a menudo se convierte en `~/.openclaw-/`)
-   `OPENCLAW_STATE_DIR=/alguna/ruta`

Si no estás seguro, ejecuta en la máquina **antigua**:

```bash
openclaw status
```

Busca menciones de `OPENCLAW_STATE_DIR` / perfil en la salida. Si ejecutas múltiples gateways, repite para cada perfil.

### 2) Identifica tu espacio de trabajo

Valores predeterminados comunes:

-   `~/.openclaw/workspace/` (espacio de trabajo recomendado)
-   una carpeta personalizada que creaste

Tu espacio de trabajo es donde viven archivos como `MEMORY.md`, `USER.md` y `memory/*.md`.

### 3) Comprende qué preservarás

Si copias **tanto** el directorio de estado como el espacio de trabajo, conservas:

-   Configuración del Gateway (`openclaw.json`)
-   Perfiles de autenticación / claves API / tokens OAuth
-   Historial de sesiones + estado del agente
-   Estado del canal (ej. inicio de sesión/sesión de WhatsApp)
-   Tus archivos del espacio de trabajo (memoria, notas de habilidades, etc.)

Si copias **solo** el espacio de trabajo (ej., vía Git), **no** preservas:

-   sesiones
-   credenciales
-   inicios de sesión de canales

Esos residen bajo `$OPENCLAW_STATE_DIR`.

## Pasos de migración (recomendados)

### Paso 0 — Haz una copia de seguridad (máquina antigua)

En la máquina **antigua**, detén primero el gateway para que los archivos no cambien durante la copia:

```bash
openclaw gateway stop
```

(Opcional pero recomendado) archiva el directorio de estado y el espacio de trabajo:

```bash
# Ajusta las rutas si usas un perfil o ubicaciones personalizadas
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Si tienes múltiples perfiles/directorios de estado (ej. `~/.openclaw-main`, `~/.openclaw-work`), archiva cada uno.

### Paso 1 — Instala OpenClaw en la nueva máquina

En la máquina **nueva**, instala la CLI (y Node si es necesario):

-   Ver: [Instalar](../install.md)

En esta etapa, está bien si la configuración inicial crea un `~/.openclaw/` nuevo — lo sobrescribirás en el siguiente paso.

### Paso 2 — Copia el directorio de estado + espacio de trabajo a la nueva máquina

Copia **ambos**:

-   `$OPENCLAW_STATE_DIR` (predeterminado `~/.openclaw/`)
-   tu espacio de trabajo (predeterminado `~/.openclaw/workspace/`)

Enfoques comunes:

-   `scp` los archivos comprimidos y extrae
-   `rsync -a` sobre SSH
-   disco externo

Después de copiar, asegúrate de que:

-   Se incluyeron los directorios ocultos (ej. `.openclaw/`)
-   La propiedad de los archivos es correcta para el usuario que ejecuta el gateway

### Paso 3 — Ejecuta Doctor (migraciones + reparación de servicio)

En la máquina **nueva**:

```bash
openclaw doctor
```

Doctor es el comando "seguro y aburrido". Repara servicios, aplica migraciones de configuración y advierte sobre inconsistencias. Luego:

```bash
openclaw gateway restart
openclaw status
```

## Errores comunes (y cómo evitarlos)

### Error: discrepancia de perfil / directorio de estado

Si ejecutaste el gateway antiguo con un perfil (o `OPENCLAW_STATE_DIR`), y el nuevo gateway usa uno diferente, verás síntomas como:

-   cambios de configuración que no surten efecto
-   canales faltantes / desconectados
-   historial de sesiones vacío

Solución: ejecuta el gateway/servicio usando el **mismo** perfil/directorio de estado que migraste, luego vuelve a ejecutar:

```bash
openclaw doctor
```

### Error: copiar solo openclaw.json

`openclaw.json` no es suficiente. Muchos proveedores almacenan estado bajo:

-   `$OPENCLAW_STATE_DIR/credentials/`
-   `$OPENCLAW_STATE_DIR/agents//...`

Siempre migra la carpeta completa `$OPENCLAW_STATE_DIR`.

### Error: permisos / propiedad

Si copiaste como root o cambiaste de usuario, el gateway podría fallar al leer credenciales/sesiones. Solución: asegúrate de que el directorio de estado + espacio de trabajo sean propiedad del usuario que ejecuta el gateway.

### Error: migrar entre modos remoto/local

-   Si tu interfaz (WebUI/TUI) apunta a un gateway **remoto**, el host remoto posee el almacén de sesiones + espacio de trabajo.
-   Migrar tu portátil no moverá el estado del gateway remoto.

Si estás en modo remoto, migra el **host del gateway**.

### Error: secretos en las copias de seguridad

`$OPENCLAW_STATE_DIR` contiene secretos (claves API, tokens OAuth, credenciales de WhatsApp). Trata las copias de seguridad como secretos de producción:

-   almacénalas cifradas
-   evita compartirlas por canales inseguros
-   rota las claves si sospechas exposición

## Lista de verificación

En la nueva máquina, confirma:

-   `openclaw status` muestra el gateway en ejecución
-   Tus canales siguen conectados (ej. WhatsApp no requiere re-emparejamiento)
-   El panel de control se abre y muestra las sesiones existentes
-   Tus archivos del espacio de trabajo (memoria, configuraciones) están presentes

## Relacionado

-   [Doctor](../gateway/doctor.md)
-   [Solución de problemas del Gateway](../gateway/troubleshooting.md)
-   [¿Dónde almacena OpenClaw sus datos?](../help/faq.md#where-does-openclaw-store-its-data)

[Actualizando](./updating.md)[Desinstalar](./uninstall.md)