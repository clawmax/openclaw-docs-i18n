

  Multi-agente

  
# Presencia

La “presencia” de OpenClaw es una vista ligera y de mejor esfuerzo de:

-   la propia **Pasarela**, y
-   los **clientes conectados a la Pasarela** (app de mac, WebChat, CLI, etc.)

La presencia se utiliza principalmente para renderizar la pestaña **Instancias** de la app de macOS y para proporcionar visibilidad rápida al operador.

## Campos de presencia (lo que aparece)

Las entradas de presencia son objetos estructurados con campos como:

-   `instanceId` (opcional pero muy recomendado): identidad estable del cliente (normalmente `connect.client.instanceId`)
-   `host`: nombre de host legible para humanos
-   `ip`: dirección IP de mejor esfuerzo
-   `version`: cadena de versión del cliente
-   `deviceFamily` / `modelIdentifier`: pistas sobre el hardware
-   `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, …
-   `lastInputSeconds`: “segundos desde la última entrada del usuario” (si se conoce)
-   `reason`: `self`, `connect`, `node-connected`, `periodic`, …
-   `ts`: marca de tiempo de la última actualización (ms desde la época)

## Productores (de dónde viene la presencia)

Las entradas de presencia son producidas por múltiples fuentes y se **fusionan**.

### 1) Entrada propia de la Pasarela

La Pasarela siempre genera una entrada “self” al inicio para que las UIs muestren el host de la pasarela incluso antes de que se conecte ningún cliente.

### 2) Conexión WebSocket

Cada cliente WS comienza con una solicitud `connect`. En un apretón de manos exitoso, la Pasarela actualiza o inserta una entrada de presencia para esa conexión.

#### Por qué los comandos CLI únicos no aparecen

La CLI a menudo se conecta para comandos únicos y breves. Para evitar saturar la lista de Instancias, `client.mode === "cli"` **no** se convierte en una entrada de presencia.

### 3) Balizas periódicas de system-event

Los clientes pueden enviar balizas periódicas más detalladas mediante el método `system-event`. La app de mac usa esto para informar el nombre del host, la IP y `lastInputSeconds`.

### 4) Conexiones de nodo (role: node)

Cuando un nodo se conecta a través del WebSocket de la Pasarela con `role: node`, la Pasarela actualiza o inserta una entrada de presencia para ese nodo (mismo flujo que otros clientes WS).

## Reglas de fusión + deduplicación (por qué importa instanceId)

Las entradas de presencia se almacenan en un único mapa en memoria:

-   Las entradas se indexan por una **clave de presencia**.
-   La mejor clave es un `instanceId` estable (de `connect.client.instanceId`) que sobreviva a reinicios.
-   Las claves no distinguen entre mayúsculas y minúsculas.

Si un cliente se reconecta sin un `instanceId` estable, puede aparecer como una fila **duplicada**.

## TTL y tamaño limitado

La presencia es intencionalmente efímera:

-   **TTL:** las entradas más antiguas de 5 minutos se eliminan
-   **Entradas máximas:** 200 (se eliminan primero las más antiguas)

Esto mantiene la lista actualizada y evita un crecimiento de memoria sin límite.

## Advertencia sobre túneles remotos (IPs de loopback)

Cuando un cliente se conecta a través de un túnel SSH / reenvío de puerto local, la Pasarela puede ver la dirección remota como `127.0.0.1`. Para evitar sobrescribir una IP buena reportada por el cliente, se ignoran las direcciones remotas de loopback.

## Consumidores

### Pestaña Instancias de macOS

La app de macOS renderiza la salida de `system-presence` y aplica un pequeño indicador de estado (Activo/Inactivo/Obsoleto) basado en la antigüedad de la última actualización.

## Consejos para depuración

-   Para ver la lista en bruto, llama a `system-presence` contra la Pasarela.
-   Si ves duplicados:
    -   confirma que los clientes envían un `client.instanceId` estable en el apretón de manos
    -   confirma que las balizas periódicas usan el mismo `instanceId`
    -   verifica si a la entrada derivada de la conexión le falta `instanceId` (se esperan duplicados)

[Enrutamiento Multi-Agente](./multi-agent.md)[Mensajes](./messages.md)