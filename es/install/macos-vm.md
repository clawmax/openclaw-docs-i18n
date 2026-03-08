

  Alojamiento y despliegue

  
# Máquinas virtuales macOS

## Recomendación por defecto (la mayoría de usuarios)

-   **VPS Linux pequeño** para un Gateway siempre activo y bajo costo. Ver [Alojamiento VPS](../vps.md).
-   **Hardware dedicado** (Mac mini o caja Linux) si quieres control total y una **IP residencial** para automatización del navegador. Muchos sitios bloquean IPs de centros de datos, por lo que la navegación local suele funcionar mejor.
-   **Híbrido:** mantén el Gateway en un VPS barato, y conecta tu Mac como un **nodo** cuando necesites automatización del navegador/interfaz. Ver [Nodos](../nodes.md) y [Gateway remoto](../gateway/remote.md).

Usa una máquina virtual macOS cuando necesites específicamente capacidades exclusivas de macOS (iMessage/BlueBubbles) o quieras un aislamiento estricto de tu Mac diario.

## Opciones de máquina virtual macOS

### VM local en tu Mac Apple Silicon (Lume)

Ejecuta OpenClaw en una máquina virtual macOS aislada en tu Mac Apple Silicon existente usando [Lume](https://cua.ai/docs/lume). Esto te da:

-   Entorno macOS completo en aislamiento (tu anfitrión se mantiene limpio)
-   Soporte de iMessage vía BlueBubbles (imposible en Linux/Windows)
-   Restablecimiento instantáneo clonando VMs
-   Sin costos adicionales de hardware o nube

### Proveedores de Mac alojados (nube)

Si quieres macOS en la nube, los proveedores de Mac alojados también funcionan:

-   [MacStadium](https://www.macstadium.com/) (Macs alojados)
-   Otros vendedores de Mac alojados también funcionan; sigue sus documentos de VM + SSH

Una vez que tengas acceso SSH a una máquina virtual macOS, continúa en el paso 6 a continuación.

* * *

## Ruta rápida (Lume, usuarios experimentados)

1.  Instalar Lume
2.  `lume create openclaw --os macos --ipsw latest`
3.  Completar el Asistente de Configuración, habilitar Inicio de Sesión Remoto (SSH)
4.  `lume run openclaw --no-display`
5.  Conectarse por SSH, instalar OpenClaw, configurar canales
6.  Listo

* * *

## Lo que necesitas (Lume)

-   Mac Apple Silicon (M1/M2/M3/M4)
-   macOS Sequoia o posterior en el anfitrión
-   ~60 GB de espacio libre en disco por VM
-   ~20 minutos

* * *

## 1) Instalar Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Si `~/.local/bin` no está en tu PATH:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Verificar:

```bash
lume --version
```

Documentación: [Instalación de Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) Crear la máquina virtual macOS

```bash
lume create openclaw --os macos --ipsw latest
```

Esto descarga macOS y crea la VM. Una ventana VNC se abre automáticamente. Nota: La descarga puede tardar un rato dependiendo de tu conexión.

* * *

## 3) Completar el Asistente de Configuración

En la ventana VNC:

1.  Seleccionar idioma y región
2.  Omitir ID de Apple (o iniciar sesión si quieres iMessage más tarde)
3.  Crear una cuenta de usuario (recuerda el nombre de usuario y contraseña)
4.  Omitir todas las funciones opcionales

Después de completar la configuración, habilita SSH:

1.  Abrir Configuración del Sistema → General → Compartir
2.  Habilitar "Inicio de Sesión Remoto"

* * *

## 4) Obtener la dirección IP de la VM

```bash
lume get openclaw
```

Busca la dirección IP (normalmente `192.168.64.x`).

* * *

## 5) Conectarse por SSH a la VM

```bash
ssh youruser@192.168.64.X
```

Reemplaza `youruser` con la cuenta que creaste, y la IP con la IP de tu VM.

* * *

## 6) Instalar OpenClaw

Dentro de la VM:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Sigue las indicaciones de incorporación para configurar tu proveedor de modelo (Anthropic, OpenAI, etc.).

* * *

## 7) Configurar canales

Edita el archivo de configuración:

```bash
nano ~/.openclaw/openclaw.json
```

Añade tus canales:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Luego inicia sesión en WhatsApp (escanea el código QR):

```bash
openclaw channels login
```

* * *

## 8) Ejecutar la VM sin interfaz gráfica

Detén la VM y reinicia sin pantalla:

```bash
lume stop openclaw
lume run openclaw --no-display
```

La VM se ejecuta en segundo plano. El daemon de OpenClaw mantiene el gateway en funcionamiento. Para verificar el estado:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## Extra: Integración con iMessage

Esta es la función estrella de ejecutar en macOS. Usa [BlueBubbles](https://bluebubbles.app) para añadir iMessage a OpenClaw. Dentro de la VM:

1.  Descarga BlueBubbles desde bluebubbles.app
2.  Inicia sesión con tu ID de Apple
3.  Habilita la API Web y establece una contraseña
4.  Dirige los webhooks de BlueBubbles a tu gateway (ejemplo: `https://your-gateway-host:3000/bluebubbles-webhook?password=`)

Añade a tu configuración de OpenClaw:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Reinicia el gateway. Ahora tu agente puede enviar y recibir iMessages. Detalles completos de configuración: [Canal BlueBubbles](../channels/bluebubbles.md)

* * *

## Guardar una imagen dorada

Antes de personalizar más, haz una instantánea de tu estado limpio:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Restablece en cualquier momento:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## Ejecución 24/7

Mantén la VM en ejecución:

-   Manteniendo tu Mac enchufado
-   Deshabilitando el suspender en Configuración del Sistema → Ahorro de Energía
-   Usando `caffeinate` si es necesario

Para un verdadero siempre activo, considera un Mac mini dedicado o un VPS pequeño. Ver [Alojamiento VPS](../vps.md).

* * *

## Solución de problemas

| Problema | Solución |
| --- | --- |
| No se puede conectar por SSH a la VM | Verifica que "Inicio de Sesión Remoto" esté habilitado en Configuración del Sistema de la VM |
| IP de la VM no aparece | Espera a que la VM arranque completamente, ejecuta `lume get openclaw` de nuevo |
| Comando Lume no encontrado | Añade `~/.local/bin` a tu PATH |
| Código QR de WhatsApp no escanea | Asegúrate de haber iniciado sesión en la VM (no en el anfitrión) al ejecutar `openclaw channels login` |

* * *

## Documentación relacionada

-   [Alojamiento VPS](../vps.md)
-   [Nodos](../nodes.md)
-   [Gateway remoto](../gateway/remote.md)
-   [Canal BlueBubbles](../channels/bluebubbles.md)
-   [Inicio rápido de Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [Referencia CLI de Lume](https://cua.ai/docs/lume/reference/cli-reference)
-   [Configuración de VM desatendida](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avanzado)
-   [Aislamiento con Docker](./docker.md) (enfoque alternativo de aislamiento)

[GCP](./gcp.md)[exe.dev](./exe-dev.md)