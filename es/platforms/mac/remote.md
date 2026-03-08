

  Aplicación complementaria para macOS

  
# Control Remoto

Este flujo permite que la aplicación de macOS actúe como un control remoto completo para una puerta de enlace OpenClaw que se ejecuta en otro host (escritorio/servidor). Es la función **Remoto sobre SSH** (ejecución remota) de la aplicación. Todas las funciones—comprobaciones de estado, reenvío de Voice Wake y Chat Web—reutilizan la misma configuración SSH remota de *Configuración → General*.

## Modos

-   **Local (esta Mac)**: Todo se ejecuta en el portátil. No se utiliza SSH.
-   **Remoto sobre SSH (predeterminado)**: Los comandos de OpenClaw se ejecutan en el host remoto. La aplicación mac abre una conexión SSH con `-o BatchMode` más tu identidad/clave elegida y un reenvío de puerto local.
-   **Remoto directo (ws/wss)**: Sin túnel SSH. La aplicación mac se conecta directamente a la URL de la puerta de enlace (por ejemplo, a través de Tailscale Serve o un proxy inverso HTTPS público).

## Transportes remotos

El modo remoto admite dos transportes:

-   **Túnel SSH** (predeterminado): Utiliza `ssh -N -L ...` para reenviar el puerto de la puerta de enlace a localhost. La puerta de enlace verá la IP del nodo como `127.0.0.1` porque el túnel es de bucle local.
-   **Directo (ws/wss)**: Se conecta directamente a la URL de la puerta de enlace. La puerta de enlace ve la IP real del cliente.

## Requisitos previos en el host remoto

1.  Instala Node + pnpm y compila/instala la CLI de OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2.  Asegúrate de que `openclaw` esté en el PATH para shells no interactivos (crea un enlace simbólico en `/usr/local/bin` o `/opt/homebrew/bin` si es necesario).
3.  Abre SSH con autenticación por clave. Recomendamos IPs de **Tailscale** para una accesibilidad estable fuera de la LAN.

## Configuración de la aplicación macOS

1.  Abre *Configuración → General*.
2.  En **Ejecuciones de OpenClaw**, elige **Remoto sobre SSH** y configura:
    -   **Transporte**: **Túnel SSH** o **Directo (ws/wss)**.
    -   **Destino SSH**: `usuario@host` (opcional `:puerto`).
        -   Si la puerta de enlace está en la misma LAN y anuncia Bonjour, selecciónala de la lista descubierta para completar este campo automáticamente.
    -   **URL de la puerta de enlace** (solo Directo): `wss://gateway.example.ts.net` (o `ws://...` para local/LAN).
    -   **Archivo de identidad** (avanzado): ruta a tu clave.
    -   **Raíz del proyecto** (avanzado): ruta del repositorio remoto utilizada para los comandos.
    -   **Ruta de la CLI** (avanzado): ruta opcional a un punto de entrada/binario ejecutable `openclaw` (se completa automáticamente cuando se anuncia).
3.  Pulsa **Probar remoto**. El éxito indica que el comando remoto `openclaw status --json` se ejecuta correctamente. Los fallos suelen significar problemas de PATH/CLI; el código de salida 127 significa que no se encuentra la CLI de forma remota.
4.  Las comprobaciones de estado y el Chat Web ahora se ejecutarán a través de este túnel SSH automáticamente.

## Chat Web

-   **Túnel SSH**: Chat Web se conecta a la puerta de enlace a través del puerto de control WebSocket reenviado (predeterminado 18789).
-   **Directo (ws/wss)**: Chat Web se conecta directamente a la URL de la puerta de enlace configurada.
-   Ya no hay un servidor HTTP separado para Chat Web.

## Permisos

-   El host remoto necesita las mismas aprobaciones TCC que en local (Automatización, Accesibilidad, Grabación de pantalla, Micrófono, Reconocimiento de voz, Notificaciones). Ejecuta la incorporación en esa máquina para otorgarlas una vez.
-   Los nodos anuncian su estado de permisos a través de `node.list` / `node.describe` para que los agentes sepan lo que está disponible.

## Notas de seguridad

-   Prefiere enlaces de bucle local en el host remoto y conéctate a través de SSH o Tailscale.
-   El túnel SSH utiliza comprobación estricta de clave de host; confía primero en la clave del host para que exista en `~/.ssh/known_hosts`.
-   Si enlazas la Puerta de enlace a una interfaz que no sea de bucle local, requiere autenticación por token/contraseña.
-   Consulta [Seguridad](../../gateway/security.md) y [Tailscale](../../gateway/tailscale.md).

## Flujo de inicio de sesión de WhatsApp (remoto)

-   Ejecuta `openclaw channels login --verbose` **en el host remoto**. Escanea el código QR con WhatsApp en tu teléfono.
-   Vuelve a ejecutar el inicio de sesión en ese host si la autenticación expira. La comprobación de estado mostrará problemas de enlace.

## Solución de problemas

-   **Código de salida 127 / no encontrado**: `openclaw` no está en el PATH para shells no interactivos. Añádelo a `/etc/paths`, tu archivo rc del shell o crea un enlace simbólico en `/usr/local/bin`/`/opt/homebrew/bin`.
-   **Sonda de estado fallida**: comprueba la accesibilidad SSH, el PATH y que Baileys haya iniciado sesión (`openclaw status --json`).
-   **Chat Web atascado**: confirma que la puerta de enlace se está ejecutando en el host remoto y que el puerto reenviado coincide con el puerto WS de la puerta de enlace; la interfaz de usuario requiere una conexión WS saludable.
-   **La IP del nodo muestra 127.0.0.1**: esperado con el túnel SSH. Cambia el **Transporte** a **Directo (ws/wss)** si quieres que la puerta de enlace vea la IP real del cliente.
-   **Voice Wake**: las frases de activación se reenvían automáticamente en modo remoto; no se necesita un reenviador separado.

## Sonidos de notificación

Elige sonidos por notificación desde scripts con `openclaw` y `node.invoke`, por ejemplo:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Puerta de enlace remota lista" --sound Glass
```

Ya no hay un interruptor global de "sonido predeterminado" en la aplicación; los llamantes eligen un sonido (o ninguno) por solicitud.

[Permisos de macOS](./permissions.md)[Firma de macOS](./signing.md)