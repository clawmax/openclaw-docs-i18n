

  Extensiones

  
# Plugin Zalo Personal

Soporte para Zalo Personal en OpenClaw a través de un plugin, utilizando `zca-js` nativo para automatizar una cuenta de usuario normal de Zalo.

> **Advertencia:** La automatización no oficial puede llevar a la suspensión/bloqueo de la cuenta. Úsalo bajo tu propio riesgo.

## Nomenclatura

El id del canal es `zalouser` para dejar claro que esto automatiza una **cuenta de usuario personal de Zalo** (no oficial). Mantenemos `zalo` reservado para una posible futura integración con la API oficial de Zalo.

## Dónde se ejecuta

Este plugin se ejecuta **dentro del proceso del Gateway**. Si usas un Gateway remoto, instálalo/configúralo en la **máquina que ejecuta el Gateway**, luego reinicia el Gateway. No se requiere ningún binario CLI externo `zca`/`openzca`.

## Instalación

### Opción A: instalar desde npm

```bash
openclaw plugins install @openclaw/zalouser
```

Reinicia el Gateway después.

### Opción B: instalar desde una carpeta local (desarrollo)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Reinicia el Gateway después.

## Configuración

La configuración del canal reside en `channels.zalouser` (no en `plugins.entries.*`):

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Herramienta de agente

Nombre de la herramienta: `zalouser` Acciones: `send`, `image`, `link`, `friends`, `groups`, `me`, `status` Las acciones de mensajes del canal también soportan `react` para reacciones a mensajes.

[Plugin de Llamada de Voz](./voice-call.md)[Manifiesto del Plugin](./manifest.md)

---