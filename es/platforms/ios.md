

  Descripción general de plataformas

  
# Aplicación iOS

Disponibilidad: vista previa interna. La aplicación iOS aún no se distribuye públicamente.

## Qué hace

-   Se conecta a un Gateway a través de WebSocket (LAN o tailnet).
-   Expone capacidades del nodo: Canvas, Captura de pantalla, Captura de cámara, Ubicación, Modo de conversación, Voice Wake.
-   Recibe comandos `node.invoke` y reporta eventos de estado del nodo.

## Requisitos

-   Gateway ejecutándose en otro dispositivo (macOS, Linux o Windows vía WSL2).
-   Ruta de red:
    -   Misma LAN vía Bonjour, **o**
    -   Tailnet vía DNS-SD unicast (dominio de ejemplo: `openclaw.internal.`), **o**
    -   Host/puerto manual (respaldo).

## Inicio rápido (emparejar + conectar)

1.  Inicia el Gateway:

```bash
openclaw gateway --port 18789
```

2.  En la aplicación iOS, abre Configuración y elige un gateway descubierto (o habilita Host Manual e ingresa el host/puerto).
3.  Aprueba la solicitud de emparejamiento en el host del gateway:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  Verifica la conexión:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## Rutas de descubrimiento

### Bonjour (LAN)

El Gateway anuncia `_openclaw-gw._tcp` en `local.`. La aplicación iOS los lista automáticamente.

### Tailnet (entre redes)

Si mDNS está bloqueado, usa una zona DNS-SD unicast (elige un dominio; ejemplo: `openclaw.internal.`) y DNS dividido de Tailscale. Consulta [Bonjour](../gateway/bonjour.md) para ver el ejemplo de CoreDNS.

### Host/puerto manual

En Configuración, habilita **Host Manual** e ingresa el host y puerto del gateway (por defecto `18789`).

## Canvas + A2UI

El nodo iOS renderiza un canvas WKWebView. Usa `node.invoke` para controlarlo:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

Notas:

-   El host del canvas del Gateway sirve `/__openclaw__/canvas/` y `/__openclaw__/a2ui/`.
-   Se sirve desde el servidor HTTP del Gateway (mismo puerto que `gateway.port`, por defecto `18789`).
-   El nodo iOS navega automáticamente a A2UI al conectarse cuando se anuncia una URL de host de canvas.
-   Regresa al andamio integrado con `canvas.navigate` y `{"url":""}`.

### Eval de canvas / captura

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## Voice Wake + modo de conversación

-   Voice Wake y el modo de conversación están disponibles en Configuración.
-   iOS puede suspender el audio en segundo plano; trata las funciones de voz como de mejor esfuerzo cuando la aplicación no está activa.

## Errores comunes

-   `NODE_BACKGROUND_UNAVAILABLE`: lleva la aplicación iOS al primer plano (los comandos de canvas/cámara/pantalla lo requieren).
-   `A2UI_HOST_NOT_CONFIGURED`: el Gateway no anunció una URL de host de canvas; verifica `canvasHost` en la [configuración del Gateway](../gateway/configuration.md).
-   El aviso de emparejamiento nunca aparece: ejecuta `openclaw devices list` y aprueba manualmente.
-   La reconexión falla después de reinstalar: el token de emparejamiento del Keychain se borró; vuelve a emparejar el nodo.

## Documentación relacionada

-   [Emparejamiento](../channels/pairing.md)
-   [Descubrimiento](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[Aplicación Android](./android.md)[DigitalOcean](./digitalocean.md)