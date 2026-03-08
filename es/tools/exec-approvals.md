

  Herramientas integradas

  
# Aprobaciones Exec

Las aprobaciones exec son la **aplicación complementaria / guardarraíl del host del nodo** para permitir que un agente en sandbox ejecute comandos en un host real (`gateway` o `node`). Piensa en ello como un enclavamiento de seguridad: los comandos solo se permiten cuando la política + la lista de permitidos + la aprobación (opcional) del usuario están de acuerdo. Las aprobaciones exec son **adicionales** a la política de herramientas y a la puerta elevada (a menos que `elevated` esté configurado en `full`, lo que omite las aprobaciones). La política efectiva es la **más estricta** entre `tools.exec.*` y los valores predeterminados de aprobaciones; si se omite un campo de aprobaciones, se usa el valor de `tools.exec`. Si la interfaz de usuario de la aplicación complementaria **no está disponible**, cualquier solicitud que requiera una confirmación se resuelve mediante la **respuesta alternativa de consulta** (predeterminado: denegar).

## Dónde se aplica

Las aprobaciones exec se aplican localmente en el host de ejecución:

-   **host del gateway** → proceso `openclaw` en la máquina del gateway
-   **host del nodo** → ejecutor del nodo (aplicación complementaria de macOS o host de nodo sin interfaz)

División en macOS:

-   El **servicio del host del nodo** reenvía `system.run` a la **aplicación de macOS** a través de IPC local.
-   La **aplicación de macOS** aplica las aprobaciones + ejecuta el comando en el contexto de la interfaz de usuario.

## Configuración y almacenamiento

Las aprobaciones residen en un archivo JSON local en el host de ejecución: `~/.openclaw/exec-approvals.json` Ejemplo de esquema:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## Controles de política

### Seguridad (exec.security)

-   **deny**: bloquear todas las solicitudes de ejecución en el host.
-   **allowlist**: permitir solo comandos en la lista de permitidos.
-   **full**: permitir todo (equivalente a elevated).

### Preguntar (exec.ask)

-   **off**: nunca solicitar confirmación.
-   **on-miss**: solicitar solo cuando la lista de permitidos no coincida.
-   **always**: solicitar en cada comando.

### Respuesta alternativa de consulta (askFallback)

Si se requiere una confirmación pero no hay interfaz de usuario disponible, la respuesta alternativa decide:

-   **deny**: bloquear.
-   **allowlist**: permitir solo si coincide con la lista de permitidos.
-   **full**: permitir.

## Lista de permitidos (por agente)

Las listas de permitidos son **por agente**. Si existen múltiples agentes, cambia qué agente estás editando en la aplicación de macOS. Los patrones son **coincidencias glob que no distinguen mayúsculas y minúsculas**. Los patrones deben resolverse a **rutas de binarios** (las entradas que son solo el nombre base se ignoran). Las entradas heredadas de `agents.default` se migran a `agents.main` al cargar. Ejemplos:

-   `~/Projects/**/bin/peekaboo`
-   `~/.local/bin/*`
-   `/opt/homebrew/bin/rg`

Cada entrada en la lista de permitidos rastrea:

-   **id**: UUID estable usado para la identidad en la interfaz de usuario (opcional)
-   **último uso**: marca de tiempo
-   **último comando usado**
-   **última ruta resuelta**

## Permitir automáticamente CLIs de habilidades

Cuando **Permitir automáticamente CLIs de habilidades** está habilitado, los ejecutables referenciados por habilidades conocidas se tratan como permitidos en los nodos (nodo macOS o host de nodo sin interfaz). Esto usa `skills.bins` a través del RPC del Gateway para obtener la lista de binarios de habilidades. Deshabilita esto si quieres listas de permitidos manuales estrictas.

## Binarios seguros (solo entrada estándar)

`tools.exec.safeBins` define una pequeña lista de binarios **solo de entrada estándar** (por ejemplo `jq`) que pueden ejecutarse en modo de lista de permitidos **sin** entradas explícitas en la lista. Los binarios seguros rechazan argumentos de archivo posicionales y tokens similares a rutas, por lo que solo pueden operar en el flujo de entrada. La validación es determinista solo a partir de la forma de argv (sin comprobaciones de existencia en el sistema de archivos del host), lo que evita el comportamiento de oráculo de existencia de archivos a partir de diferencias de permitir/denegar. Las opciones orientadas a archivos se deniegan para los binarios seguros predeterminados (por ejemplo `sort -o`, `sort --output`, `sort --files0-from`, `sort --compress-program`, `wc --files0-from`, `jq -f/--from-file`, `grep -f/--file`). Los binarios seguros también aplican una política de banderas explícita por binario para opciones que rompen el comportamiento de solo entrada estándar (por ejemplo `sort -o/--output/--compress-program` y las banderas recursivas de grep). Los binarios seguros también fuerzan que los tokens de argv se traten como **texto literal** en el momento de ejecución (sin expansión de glob ni de `$VARS`) para segmentos de solo entrada estándar, por lo que patrones como `*` o `$HOME/...` no pueden usarse para filtrar lecturas de archivos. Los binarios seguros también deben resolverse desde directorios de binarios confiables (predeterminados del sistema más el `PATH` del proceso del gateway al inicio). Esto bloquea intentos de secuestro del PATH con alcance de solicitud. El encadenamiento de shell y las redirecciones no se permiten automáticamente en modo de lista de permitidos. El encadenamiento de shell (`&&`, `||`, `;`) se permite cuando cada segmento de nivel superior cumple con la lista de permitidos (incluyendo binarios seguros o la auto-autorización de habilidades). Las redirecciones siguen sin ser compatibles en modo de lista de permitidos. La sustitución de comandos (`$()` / comillas invertidas) se rechaza durante el análisis de la lista de permitidos, incluso dentro de comillas dobles; usa comillas simples si necesitas texto literal `$()`. En las aprobaciones de la aplicación complementaria de macOS, el texto de shell crudo que contenga sintaxis de control o expansión de shell (`&&`, `||`, `;`, `|`, ```, `$`, `<`, `>`, `(`, `)`) se trata como una falta en la lista de permitidos a menos que el propio binario del shell esté en la lista. Binarios seguros predeterminados: `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`. `grep` y `sort` no están en la lista predeterminada. Si los incluyes, mantén entradas explícitas en la lista de permitidos para sus flujos de trabajo que no son de solo entrada estándar. Para `grep` en modo de binario seguro, proporciona el patrón con `-e`/`--regexp`; la forma posicional del patrón se rechaza para que los operandos de archivo no puedan filtrarse como posicionales ambiguos.

## Edición en la interfaz de Control

Usa la tarjeta **Interfaz de Control → Nodos → Aprobaciones Exec** para editar los valores predeterminados, las anulaciones por agente y las listas de permitidos. Elige un alcance (Predeterminados o un agente), ajusta la política, añade/elimina patrones de la lista de permitidos, luego **Guarda**. La interfaz muestra metadatos de **último uso** por patrón para que puedas mantener la lista ordenada. El selector de destino elige **Gateway** (aprobaciones locales) o un **Nodo**. Los nodos deben anunciar `system.execApprovals.get/set` (aplicación de macOS o host de nodo sin interfaz). Si un nodo aún no anuncia aprobaciones exec, edita su archivo local `~/.openclaw/exec-approvals.json` directamente. CLI: `openclaw approvals` soporta edición en gateway o nodo (ver [CLI de Aprobaciones](/cli/approvals)).

## Flujo de aprobación

Cuando se requiere una confirmación, el gateway transmite `exec.approval.requested` a los clientes operadores. La Interfaz de Control y la aplicación de macOS la resuelven mediante `exec.approval.resolve`, luego el gateway reenvía la solicitud aprobada al host del nodo. Cuando se requieren aprobaciones, la herramienta exec regresa inmediatamente con un id de aprobación. Usa ese id para correlacionar eventos posteriores del sistema (`Exec finished` / `Exec denied`). Si no llega ninguna decisión antes del tiempo de espera, la solicitud se trata como un tiempo de espera de aprobación y se muestra como un motivo de denegación. El diálogo de confirmación incluye:

-   comando + argumentos
-   cwd
-   id del agente
-   ruta del ejecutable resuelta
-   host + metadatos de política

Acciones:

-   **Permitir una vez** → ejecutar ahora
-   **Permitir siempre** → añadir a la lista de permitidos + ejecutar
-   **Denegar** → bloquear

## Reenvío de aprobaciones a canales de chat

Puedes reenviar solicitudes de aprobación exec a cualquier canal de chat (incluyendo canales de plugins) y aprobarlas con `/approve`. Esto usa la canalización de entrega saliente normal. Configuración:

```json
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring o regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

Responde en el chat:

```bash
/approve  allow-once
/approve  allow-always
/approve  deny
```

### Flujo IPC de macOS

```
Gateway -> Servicio del Nodo (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Aplicación Mac (UI + aprobaciones + system.run)
```

Notas de seguridad:

-   Modo de socket Unix `0600`, token almacenado en `exec-approvals.json`.
-   Verificación de par del mismo UID.
-   Desafío/respuesta (nonce + HMAC token + hash de solicitud) + TTL corto.

## Eventos del sistema

El ciclo de vida de Exec se muestra como mensajes del sistema:

-   `Exec running` (solo si el comando excede el umbral de notificación de ejecución)
-   `Exec finished`
-   `Exec denied`

Estos se publican en la sesión del agente después de que el nodo reporte el evento. Las aprobaciones exec en el host del gateway emiten los mismos eventos de ciclo de vida cuando el comando termina (y opcionalmente cuando se ejecuta más tiempo que el umbral). Las ejecuciones con aprobación reutilizan el id de aprobación como `runId` en estos mensajes para una fácil correlación.

## Implicaciones

-   **full** es poderoso; prefiere listas de permitidos cuando sea posible.
-   **ask** te mantiene informado mientras aún permite aprobaciones rápidas.
-   Las listas de permitidos por agente evitan que las aprobaciones de un agente se filtren a otros.
-   Las aprobaciones solo se aplican a solicitudes de ejecución en el host de **remitentes autorizados**. Los remitentes no autorizados no pueden emitir `/exec`.
-   `/exec security=full` es una conveniencia a nivel de sesión para operadores autorizados y omite las aprobaciones por diseño. Para bloquear duramente la ejecución en el host, configura la seguridad de aprobaciones en `deny` o deniega la herramienta `exec` mediante la política de herramientas.

Relacionado:

-   [Herramienta Exec](/tools/exec)
-   [Modo Elevated](/tools/elevated)
-   [Habilidades](/tools/skills)

[Herramienta Exec](/tools/exec)[Firecrawl](/tools/firecrawl)