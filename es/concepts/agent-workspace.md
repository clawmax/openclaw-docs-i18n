

  Fundamentos

  
# Espacio de trabajo del agente

El espacio de trabajo es el hogar del agente. Es el único directorio de trabajo utilizado para las herramientas de archivos y para el contexto del espacio de trabajo. Mantenlo privado y trátalo como memoria. Esto está separado de `~/.openclaw/`, que almacena la configuración, credenciales y sesiones. **Importante:** el espacio de trabajo es el **cwd predeterminado**, no un sandbox estricto. Las herramientas resuelven las rutas relativas contra el espacio de trabajo, pero las rutas absolutas aún pueden acceder a otros lugares del host a menos que se habilite el sandboxing. Si necesitas aislamiento, usa [`agents.defaults.sandbox`](../gateway/sandboxing.md) (y/o la configuración de sandbox por agente). Cuando el sandboxing está habilitado y `workspaceAccess` no es `"rw"`, las herramientas operan dentro de un espacio de trabajo sandbox bajo `~/.openclaw/sandboxes`, no dentro de tu espacio de trabajo del host.

## Ubicación predeterminada

-   Predeterminada: `~/.openclaw/workspace`
-   Si `OPENCLAW_PROFILE` está configurado y no es `"default"`, la ubicación predeterminada se convierte en `~/.openclaw/workspace-`.
-   Sobrescribir en `~/.openclaw/openclaw.json`:

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`, `openclaw configure`, o `openclaw setup` crearán el espacio de trabajo y sembrarán los archivos de arranque si faltan. Las copias de semilla del sandbox solo aceptan archivos regulares dentro del espacio de trabajo; los alias de enlace simbólico/enlace físico que resuelvan fuera del espacio de trabajo fuente son ignorados. Si ya gestionas los archivos del espacio de trabajo tú mismo, puedes deshabilitar la creación de archivos de arranque:

```json
{ agent: { skipBootstrap: true } }
```

## Carpetas adicionales del espacio de trabajo

Las instalaciones antiguas pueden haber creado `~/openclaw`. Mantener múltiples directorios de espacio de trabajo puede causar confusión en la autenticación o desviación de estado, porque solo un espacio de trabajo está activo a la vez. **Recomendación:** mantén un solo espacio de trabajo activo. Si ya no usas las carpetas adicionales, archívalas o muévelas a la Papelera (por ejemplo `trash ~/openclaw`). Si mantienes múltiples espacios de trabajo intencionalmente, asegúrate de que `agents.defaults.workspace` apunte al activo. `openclaw doctor` advierte cuando detecta directorios de espacio de trabajo adicionales.

## Mapa de archivos del espacio de trabajo (qué significa cada archivo)

Estos son los archivos estándar que OpenClaw espera dentro del espacio de trabajo:

-   `AGENTS.md`
    -   Instrucciones de operación para el agente y cómo debe usar la memoria.
    -   Se carga al inicio de cada sesión.
    -   Buen lugar para reglas, prioridades y detalles sobre "cómo comportarse".
-   `SOUL.md`
    -   Persona, tono y límites.
    -   Se carga en cada sesión.
-   `USER.md`
    -   Quién es el usuario y cómo dirigirse a él.
    -   Se carga en cada sesión.
-   `IDENTITY.md`
    -   El nombre, estilo y emoji del agente.
    -   Creado/actualizado durante el ritual de arranque.
-   `TOOLS.md`
    -   Notas sobre tus herramientas locales y convenciones.
    -   No controla la disponibilidad de herramientas; es solo una guía.
-   `HEARTBEAT.md`
    -   Lista de verificación opcional pequeña para ejecuciones de heartbeat.
    -   Mantenla corta para evitar consumo de tokens.
-   `BOOT.md`
    -   Lista de verificación de inicio opcional ejecutada al reiniciar el gateway cuando los hooks internos están habilitados.
    -   Mantenla corta; usa la herramienta de mensaje para envíos salientes.
-   `BOOTSTRAP.md`
    -   Ritual de primera ejecución único.
    -   Solo se crea para un espacio de trabajo completamente nuevo.
    -   Elimínalo después de completar el ritual.
-   `memory/AAAA-MM-DD.md`
    -   Registro de memoria diario (un archivo por día).
    -   Se recomienda leer el de hoy + el de ayer al inicio de la sesión.
-   `MEMORY.md` (opcional)
    -   Memoria a largo plazo curada.
    -   Solo se carga en la sesión principal y privada (no en contextos compartidos/grupales).

Consulta [Memoria](./memory.md) para el flujo de trabajo y el vaciado automático de memoria.

-   `skills/` (opcional)
    -   Habilidades específicas del espacio de trabajo.
    -   Sobrescribe las habilidades gestionadas/empaquetadas cuando los nombres coinciden.
-   `canvas/` (opcional)
    -   Archivos de la interfaz de usuario Canvas para visualizaciones de nodos (por ejemplo `canvas/index.html`).

Si falta algún archivo de arranque, OpenClaw inyecta un marcador de "archivo faltante" en la sesión y continúa. Los archivos de arranque grandes se truncan al inyectarse; ajusta los límites con `agents.defaults.bootstrapMaxChars` (predeterminado: 20000) y `agents.defaults.bootstrapTotalMaxChars` (predeterminado: 150000). `openclaw setup` puede recrear los archivos predeterminados faltantes sin sobrescribir los existentes.

## Qué NO está en el espacio de trabajo

Estos viven bajo `~/.openclaw/` y NO deben comprometerse en el repositorio del espacio de trabajo:

-   `~/.openclaw/openclaw.json` (configuración)
-   `~/.openclaw/credentials/` (tokens OAuth, claves API)
-   `~/.openclaw/agents//sessions/` (transcripciones de sesión + metadatos)
-   `~/.openclaw/skills/` (habilidades gestionadas)

Si necesitas migrar sesiones o configuración, cópialas por separado y mantenlas fuera del control de versiones.

## Copia de seguridad con Git (recomendado, privado)

Trata el espacio de trabajo como memoria privada. Ponlo en un repositorio git **privado** para que esté respaldado y sea recuperable. Ejecuta estos pasos en la máquina donde se ejecuta el Gateway (ahí es donde vive el espacio de trabajo).

### 1) Inicializar el repositorio

Si git está instalado, los espacios de trabajo nuevos se inicializan automáticamente. Si este espacio de trabajo aún no es un repositorio, ejecuta:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) Agregar un remoto privado (opciones para principiantes)

Opción A: Interfaz web de GitHub

1.  Crea un nuevo repositorio **privado** en GitHub.
2.  No lo inicialices con un README (evita conflictos de fusión).
3.  Copia la URL remota HTTPS.
4.  Agrega el remoto y haz push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Opción B: CLI de GitHub (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Opción C: Interfaz web de GitLab

1.  Crea un nuevo repositorio **privado** en GitLab.
2.  No lo inicialices con un README (evita conflictos de fusión).
3.  Copia la URL remota HTTPS.
4.  Agrega el remoto y haz push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) Actualizaciones continuas

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## No comprometas secretos

Incluso en un repositorio privado, evita almacenar secretos en el espacio de trabajo:

-   Claves API, tokens OAuth, contraseñas o credenciales privadas.
-   Cualquier cosa bajo `~/.openclaw/`.
-   Volcados crudos de chats o archivos adjuntos sensibles.

Si debes almacenar referencias sensibles, usa marcadores de posición y guarda el secreto real en otro lugar (gestor de contraseñas, variables de entorno o `~/.openclaw/`). Sugerencia de `.gitignore` inicial:

```
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## Mover el espacio de trabajo a una nueva máquina

1.  Clona el repositorio en la ruta deseada (predeterminado `~/.openclaw/workspace`).
2.  Configura `agents.defaults.workspace` con esa ruta en `~/.openclaw/openclaw.json`.
3.  Ejecuta `openclaw setup --workspace ` para sembrar cualquier archivo faltante.
4.  Si necesitas sesiones, copia `~/.openclaw/agents//sessions/` de la máquina antigua por separado.

## Notas avanzadas

-   El enrutamiento multi-agente puede usar diferentes espacios de trabajo por agente. Consulta [Enrutamiento de canales](../channels/channel-routing.md) para la configuración de enrutamiento.
-   Si `agents.defaults.sandbox` está habilitado, las sesiones no principales pueden usar espacios de trabajo sandbox por sesión bajo `agents.defaults.sandbox.workspaceRoot`.

[Contexto](./context.md)[OAuth](./oauth.md)