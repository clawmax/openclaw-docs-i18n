

  Habilidades

  
# ClawHub

ClawHub es el **registro público de habilidades para OpenClaw**. Es un servicio gratuito: todas las habilidades son públicas, abiertas y visibles para todos para compartir y reutilizar. Una habilidad es solo una carpeta con un archivo `SKILL.md` (más archivos de texto de soporte). Puedes explorar habilidades en la aplicación web o usar la CLI para buscar, instalar, actualizar y publicar habilidades. Sitio: [clawhub.ai](https://clawhub.ai)

## Qué es ClawHub

-   Un registro público para habilidades de OpenClaw.
-   Un almacén versionado de paquetes de habilidades y metadatos.
-   Una superficie de descubrimiento para búsqueda, etiquetas y señales de uso.

## Cómo funciona

1.  Un usuario publica un paquete de habilidades (archivos + metadatos).
2.  ClawHub almacena el paquete, analiza los metadatos y asigna una versión.
3.  El registro indexa la habilidad para búsqueda y descubrimiento.
4.  Los usuarios exploran, descargan e instalan habilidades en OpenClaw.

## Qué puedes hacer

-   Publicar nuevas habilidades y nuevas versiones de habilidades existentes.
-   Descubrir habilidades por nombre, etiquetas o búsqueda.
-   Descargar paquetes de habilidades e inspeccionar sus archivos.
-   Reportar habilidades que sean abusivas o inseguras.
-   Si eres moderador, ocultar, mostrar, eliminar o prohibir.

## Para quién es esto (apto para principiantes)

Si quieres añadir nuevas capacidades a tu agente OpenClaw, ClawHub es la forma más fácil de encontrar e instalar habilidades. No necesitas saber cómo funciona el backend. Puedes:

-   Buscar habilidades con lenguaje natural.
-   Instalar una habilidad en tu espacio de trabajo.
-   Actualizar habilidades más tarde con un comando.
-   Hacer copias de seguridad de tus propias habilidades publicándolas.

## Inicio rápido (no técnico)

1.  Instala la CLI (ver la siguiente sección).
2.  Busca algo que necesites:
    -   `clawhub search "calendar"`
3.  Instala una habilidad:
    -   `clawhub install <skill-slug>`
4.  Inicia una nueva sesión de OpenClaw para que detecte la nueva habilidad.

## Instalar la CLI

Elige una:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## Cómo se integra en OpenClaw

Por defecto, la CLI instala habilidades en `./skills` bajo tu directorio de trabajo actual. Si hay un espacio de trabajo de OpenClaw configurado, `clawhub` recurre a ese espacio de trabajo a menos que lo anules con `--workdir` (o `CLAWHUB_WORKDIR`). OpenClaw carga las habilidades del espacio de trabajo desde `/skills` y las detectará en la **siguiente** sesión. Si ya usas `~/.openclaw/skills` o habilidades empaquetadas, las habilidades del espacio de trabajo tienen prioridad. Para más detalles sobre cómo se cargan, comparten y controlan las habilidades, consulta [Habilidades](./skills.md).

## Descripción general del sistema de habilidades

Una habilidad es un paquete versionado de archivos que enseña a OpenClaw cómo realizar una tarea específica. Cada publicación crea una nueva versión, y el registro mantiene un historial de versiones para que los usuarios puedan auditar los cambios. Una habilidad típica incluye:

-   Un archivo `SKILL.md` con la descripción principal y el uso.
-   Configuraciones opcionales, scripts o archivos de soporte utilizados por la habilidad.
-   Metadatos como etiquetas, resumen y requisitos de instalación.

ClawHub utiliza metadatos para potenciar el descubrimiento y exponer de forma segura las capacidades de las habilidades. El registro también rastrea señales de uso (como estrellas y descargas) para mejorar la clasificación y visibilidad.

## Qué proporciona el servicio (características)

-   **Navegación pública** de habilidades y su contenido `SKILL.md`.
-   **Búsqueda** potenciada por incrustaciones (búsqueda vectorial), no solo palabras clave.
-   **Control de versiones** con semver, registros de cambios y etiquetas (incluyendo `latest`).
-   **Descargas** como un zip por versión.
-   **Estrellas y comentarios** para retroalimentación de la comunidad.
-   **Ganchos de moderación** para aprobaciones y auditorías.
-   **API compatible con CLI** para automatización y scripting.

## Seguridad y moderación

ClawHub es abierto por defecto. Cualquiera puede subir habilidades, pero una cuenta de GitHub debe tener al menos una semana de antigüedad para publicar. Esto ayuda a ralentizar el abuso sin bloquear a los contribuyentes legítimos. Reporte y moderación:

-   Cualquier usuario registrado puede reportar una habilidad.
-   Se requieren y registran las razones del reporte.
-   Cada usuario puede tener hasta 20 reportes activos a la vez.
-   Las habilidades con más de 3 reportes únicos se ocultan automáticamente por defecto.
-   Los moderadores pueden ver habilidades ocultas, mostrarlas, eliminarlas o prohibir usuarios.
-   Abusar de la función de reporte puede resultar en la prohibición de la cuenta.

¿Interesado en convertirte en moderador? Pregunta en el Discord de OpenClaw y contacta a un moderador o mantenedor.

## Comandos y parámetros de la CLI

Opciones globales (se aplican a todos los comandos):

-   `--workdir `: Directorio de trabajo (por defecto: directorio actual; recurre al espacio de trabajo de OpenClaw).
-   `--dir `: Directorio de habilidades, relativo a workdir (por defecto: `skills`).
-   `--site `: URL base del sitio (inicio de sesión en el navegador).
-   `--registry `: URL base de la API del registro.
-   `--no-input`: Deshabilitar prompts (no interactivo).
-   `-V, --cli-version`: Imprimir versión de la CLI.

Autenticación:

-   `clawhub login` (flujo del navegador) o `clawhub login --token `
-   `clawhub logout`
-   `clawhub whoami`

Opciones:

-   `--token `: Pegar un token de API.
-   `--label `: Etiqueta almacenada para tokens de inicio de sesión en el navegador (por defecto: `CLI token`).
-   `--no-browser`: No abrir un navegador (requiere `--token`).

Buscar:

-   `clawhub search "query"`
-   `--limit `: Resultados máximos.

Instalar:

-   `clawhub install `
-   `--version `: Instalar una versión específica.
-   `--force`: Sobrescribir si la carpeta ya existe.

Actualizar:

-   `clawhub update `
-   `clawhub update --all`
-   `--version `: Actualizar a una versión específica (solo un slug).
-   `--force`: Sobrescribir cuando los archivos locales no coincidan con ninguna versión publicada.

Listar:

-   `clawhub list` (lee `.clawhub/lock.json`)

Publicar:

-   `clawhub publish `
-   `--slug `: Slug de la habilidad.
-   `--name `: Nombre para mostrar.
-   `--version `: Versión semver.
-   `--changelog `: Texto del registro de cambios (puede estar vacío).
-   `--tags `: Etiquetas separadas por comas (por defecto: `latest`).

Eliminar/restaurar (solo propietario/administrador):

-   `clawhub delete  --yes`
-   `clawhub undelete  --yes`

Sincronizar (escanear habilidades locales + publicar nuevas/actualizadas):

-   `clawhub sync`
-   `--root <dir...>`: Raíces de escaneo adicionales.
-   `--all`: Subir todo sin prompts.
-   `--dry-run`: Mostrar lo que se subiría.
-   `--bump `: `patch|minor|major` para actualizaciones (por defecto: `patch`).
-   `--changelog `: Registro de cambios para actualizaciones no interactivas.
-   `--tags `: Etiquetas separadas por comas (por defecto: `latest`).
-   `--concurrency `: Comprobaciones del registro (por defecto: 4).

## Flujos de trabajo comunes para agentes

### Buscar habilidades

```bash
clawhub search "postgres backups"
```

### Descargar nuevas habilidades

```bash
clawhub install my-skill-pack
```

### Actualizar habilidades instaladas

```bash
clawhub update --all
```

### Hacer copia de seguridad de tus habilidades (publicar o sincronizar)

Para una sola carpeta de habilidad:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Para escanear y hacer copia de seguridad de muchas habilidades a la vez:

```bash
clawhub sync --all
```

## Detalles avanzados (técnicos)

### Control de versiones y etiquetas

-   Cada publicación crea una nueva **semver** `SkillVersion`.
-   Las etiquetas (como `latest`) apuntan a una versión; mover etiquetas permite retroceder.
-   Los registros de cambios se adjuntan por versión y pueden estar vacíos al sincronizar o publicar actualizaciones.

### Cambios locales vs versiones del registro

Las actualizaciones comparan el contenido local de la habilidad con las versiones del registro usando un hash de contenido. Si los archivos locales no coinciden con ninguna versión publicada, la CLI pregunta antes de sobrescribir (o requiere `--force` en ejecuciones no interactivas).

### Escaneo de sincronización y raíces de respaldo

`clawhub sync` escanea primero tu directorio de trabajo actual. Si no se encuentran habilidades, recurre a ubicaciones heredadas conocidas (por ejemplo `~/openclaw/skills` y `~/.openclaw/skills`). Esto está diseñado para encontrar instalaciones de habilidades más antiguas sin banderas adicionales.

### Almacenamiento y archivo de bloqueo

-   Las habilidades instaladas se registran en `.clawhub/lock.json` bajo tu directorio de trabajo.
-   Los tokens de autenticación se almacenan en el archivo de configuración de la CLI de ClawHub (anular mediante `CLAWHUB_CONFIG_PATH`).

### Telemetría (recuentos de instalación)

Cuando ejecutas `clawhub sync` mientras estás conectado, la CLI envía una instantánea mínima para calcular los recuentos de instalación. Puedes deshabilitar esto por completo:

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

## Variables de entorno

-   `CLAWHUB_SITE`: Anular la URL del sitio.
-   `CLAWHUB_REGISTRY`: Anular la URL de la API del registro.
-   `CLAWHUB_CONFIG_PATH`: Anular dónde la CLI almacena el token/configuración.
-   `CLAWHUB_WORKDIR`: Anular el directorio de trabajo por defecto.
-   `CLAWHUB_DISABLE_TELEMETRY=1`: Deshabilitar la telemetría en `sync`.

[Configuración de Habilidades](./skills-config.md)[Complementos](./plugin.md)