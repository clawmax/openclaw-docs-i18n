title: "Guía de configuración de integración de Microsoft Teams para OpenClaw AI"
description: "Aprende a instalar y configurar el complemento de Microsoft Teams para OpenClaw AI. Configura Azure Bot, gestiona el control de acceso y habilita mensajería en MD, grupos y canales."
keywords: ["integración de microsoft teams", "complemento openclaw msteams", "configuración de azure bot", "configuración de chatbot de teams", "canales de openclaw", "webhook de teams", "portal de desarrollador de teams", "tunelización para desarrollo local"]
updated: """""2026-01-21"""""
---

  Plataformas de mensajería

  
# Microsoft Teams

> “Abandonad toda esperanza, vosotros que entráis aquí.”

Actualizado: 2026-01-21 Estado: texto + archivos adjuntos en MD son compatibles; el envío de archivos en canales/grupos requiere `sharePointSiteId` + permisos de Graph (ver [Enviar archivos en chats grupales](#enviar-archivos-en-chats-grupales)). Las encuestas se envían mediante Adaptive Cards.

## Complemento requerido

Microsoft Teams se distribuye como un complemento y no está incluido en la instalación principal. **Cambio importante (2026.1.15):** MS Teams se movió fuera del núcleo. Si lo usas, debes instalar el complemento. Explicación: mantiene las instalaciones principales más ligeras y permite que las dependencias de MS Teams se actualicen de forma independiente. Instala mediante CLI (registro npm):

```bash
openclaw plugins install @openclaw/msteams
```

Checkout local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/msteams
```

Si eliges Teams durante la configuración/onboarding y se detecta un checkout de git, OpenClaw ofrecerá la ruta de instalación local automáticamente. Detalles: [Complementos](../tools/plugin.md)

## Configuración rápida (principiante)

1.  Instala el complemento de Microsoft Teams.
2.  Crea un **Azure Bot** (ID de aplicación + secreto de cliente + ID de inquilino).
3.  Configura OpenClaw con esas credenciales.
4.  Expone `/api/messages` (puerto 3978 por defecto) mediante una URL pública o un túnel.
5.  Instala el paquete de la aplicación de Teams e inicia la pasarela.

Configuración mínima:

```json
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Nota: los chats grupales están bloqueados por defecto (`channels.msteams.groupPolicy: "allowlist"`). Para permitir respuestas en grupo, configura `channels.msteams.groupAllowFrom` (o usa `groupPolicy: "open"` para permitir a cualquier miembro, sujeto a mención).

## Objetivos

-   Hablar con OpenClaw a través de MD, chats grupales o canales de Teams.
-   Mantener el enrutamiento determinista: las respuestas siempre regresan al canal del que provienen.
-   Comportamiento predeterminado seguro en canales (se requieren menciones a menos que se configure lo contrario).

## Escritura de configuración

Por defecto, Microsoft Teams puede escribir actualizaciones de configuración activadas por `/config set|unset` (requiere `commands.config: true`). Deshabilita con:

```json
{
  channels: { msteams: { configWrites: false } },
}
```

## Control de acceso (MD + grupos)

**Acceso en MD**

-   Por defecto: `channels.msteams.dmPolicy = "pairing"`. Los remitentes desconocidos son ignorados hasta ser aprobados.
-   `channels.msteams.allowFrom` debe usar ID de objetos AAD estables.
-   Los UPNs/nombres para mostrar son mutables; la coincidencia directa está deshabilitada por defecto y solo se habilita con `channels.msteams.dangerouslyAllowNameMatching: true`.
-   El asistente puede resolver nombres a ID mediante Microsoft Graph cuando las credenciales lo permiten.

**Acceso en grupo**

-   Por defecto: `channels.msteams.groupPolicy = "allowlist"` (bloqueado a menos que agregues `groupAllowFrom`). Usa `channels.defaults.groupPolicy` para anular el valor predeterminado cuando no esté establecido.
-   `channels.msteams.groupAllowFrom` controla qué remitentes pueden activar el bot en chats grupales/canales (recurre a `channels.msteams.allowFrom`).
-   Configura `groupPolicy: "open"` para permitir a cualquier miembro (aún sujeto a mención por defecto).
-   Para no permitir **ningún canal**, configura `channels.msteams.groupPolicy: "disabled"`.

Ejemplo:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Lista de equipos y canales permitidos**

-   Limita las respuestas en grupo/canal listando equipos y canales bajo `channels.msteams.teams`.
-   Las claves pueden ser ID de equipo o nombres; las claves de canal pueden ser ID de conversación o nombres.
-   Cuando `groupPolicy="allowlist"` y existe una lista de equipos permitidos, solo se aceptan los equipos/canales listados (sujetos a mención).
-   El asistente de configuración acepta entradas `Equipo/Canal` y las almacena por ti.
-   Al iniciar, OpenClaw resuelve los nombres de equipo/canal y de la lista de usuarios permitidos a ID (cuando los permisos de Graph lo permiten) y registra el mapeo; las entradas no resueltas se mantienen como se escribieron.

Ejemplo:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "Mi Equipo": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## Cómo funciona

1.  Instala el complemento de Microsoft Teams.
2.  Crea un **Azure Bot** (ID de aplicación + secreto + ID de inquilino).
3.  Construye un **paquete de aplicación de Teams** que haga referencia al bot e incluya los permisos RSC a continuación.
4.  Sube/instala la aplicación de Teams en un equipo (o en el ámbito personal para MD).
5.  Configura `msteams` en `~/.openclaw/openclaw.json` (o variables de entorno) e inicia la pasarela.
6.  La pasarela escucha el tráfico de webhook del Bot Framework en `/api/messages` por defecto.

## Configuración de Azure Bot (Prerrequisitos)

Antes de configurar OpenClaw, necesitas crear un recurso de Azure Bot.

### Paso 1: Crear Azure Bot

1.  Ve a [Crear Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2.  Completa la pestaña **Básico**:
    
    | Campo | Valor |
    | --- | --- |
    | **Identificador del bot** | Nombre de tu bot, ej., `openclaw-msteams` (debe ser único) |
    | **Suscripción** | Selecciona tu suscripción de Azure |
    | **Grupo de recursos** | Crear nuevo o usar existente |
    | **Plan de tarifa** | **Gratis** para desarrollo/pruebas |
    | **Tipo de aplicación** | **Inquilino único** (recomendado - ver nota abajo) |
    | **Tipo de creación** | **Crear nueva ID de aplicación de Microsoft** |
    

> **Aviso de desuso:** La creación de nuevos bots multiinquilino quedó en desuso después del 2025-07-31. Usa **Inquilino único** para nuevos bots.

3.  Haz clic en **Revisar + crear** → **Crear** (espera ~1-2 minutos)

### Paso 2: Obtener Credenciales

1.  Ve a tu recurso de Azure Bot → **Configuración**
2.  Copia **ID de aplicación de Microsoft** → este es tu `appId`
3.  Haz clic en **Administrar contraseña** → ve al Registro de aplicación
4.  En **Certificados y secretos** → **Nuevo secreto de cliente** → copia el **Valor** → este es tu `appPassword`
5.  Ve a **Información general** → copia **ID de directorio (inquilino)** → este es tu `tenantId`

### Paso 3: Configurar el Punto Final de Mensajería

1.  En Azure Bot → **Configuración**
2.  Establece **Punto final de mensajería** a tu URL de webhook:
    -   Producción: `https://tu-dominio.com/api/messages`
    -   Desarrollo local: Usa un túnel (ver [Desarrollo local con túnel](#desarrollo-local-con-túnel) abajo)

### Paso 4: Habilitar el Canal de Teams

1.  En Azure Bot → **Canales**
2.  Haz clic en **Microsoft Teams** → Configurar → Guardar
3.  Acepta los Términos de servicio

## Desarrollo Local (Tunelización)

Teams no puede alcanzar `localhost`. Usa un túnel para desarrollo local: **Opción A: ngrok**

```bash
ngrok http 3978
# Copia la URL https, ej., https://abc123.ngrok.io
# Establece el punto final de mensajería a: https://abc123.ngrok.io/api/messages
```

**Opción B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Usa tu URL de funnel de Tailscale como punto final de mensajería
```

## Portal de Desarrollador de Teams (Alternativa)

En lugar de crear manualmente un ZIP de manifiesto, puedes usar el [Portal de Desarrollador de Teams](https://dev.teams.microsoft.com/apps):

1.  Haz clic en **\+ Nueva aplicación**
2.  Completa la información básica (nombre, descripción, información del desarrollador)
3.  Ve a **Características de la aplicación** → **Bot**
4.  Selecciona **Ingresar un ID de bot manualmente** y pega tu ID de aplicación de Azure Bot
5.  Marca los ámbitos: **Personal**, **Equipo**, **Chat grupal**
6.  Haz clic en **Distribuir** → **Descargar paquete de aplicación**
7.  En Teams: **Aplicaciones** → **Administrar tus aplicaciones** → **Cargar una aplicación personalizada** → selecciona el ZIP

Esto suele ser más fácil que editar manualmente manifiestos JSON.

## Probar el Bot

**Opción A: Chat web de Azure (verificar webhook primero)**

1.  En el Portal de Azure → tu recurso de Azure Bot → **Probar en Chat web**
2.  Envía un mensaje - deberías ver una respuesta
3.  Esto confirma que tu punto final de webhook funciona antes de configurar Teams

**Opción B: Teams (después de instalar la aplicación)**

1.  Instala la aplicación de Teams (carga lateral o catálogo de la organización)
2.  Encuentra el bot en Teams y envía un MD
3.  Revisa los registros de la pasarela para ver actividad entrante

## Configuración (mínima solo texto)

1.  **Instala el complemento de Microsoft Teams**
    -   Desde npm: `openclaw plugins install @openclaw/msteams`
    -   Desde un checkout local: `openclaw plugins install ./extensions/msteams`
2.  **Registro del bot**
    -   Crea un Azure Bot (ver arriba) y anota:
        -   ID de aplicación
        -   Secreto de cliente (Contraseña de aplicación)
        -   ID de inquilino (inquilino único)
3.  **Manifiesto de la aplicación de Teams**
    -   Incluye una entrada `bot` con `botId = `.
    -   Ámbitos: `personal`, `team`, `groupChat`.
    -   `supportsFiles: true` (requerido para manejo de archivos en ámbito personal).
    -   Agrega permisos RSC (abajo).
    -   Crea iconos: `outline.png` (32x32) y `color.png` (192x192).
    -   Comprime los tres archivos juntos: `manifest.json`, `outline.png`, `color.png`.
4.  **Configura OpenClaw**
    
    Copiar
    
    ```json
    {
      "msteams": {
        "enabled": true,
        "appId": "<APP_ID>",
        "appPassword": "<APP_PASSWORD>",
        "tenantId": "<TENANT_ID>",
        "webhook": { "port": 3978, "path": "/api/messages" }
      }
    }
    ```
    
    También puedes usar variables de entorno en lugar de claves de configuración:
    -   `MSTEAMS_APP_ID`
    -   `MSTEAMS_APP_PASSWORD`
    -   `MSTEAMS_TENANT_ID`
5.  **Punto final del bot**
    -   Establece el Punto Final de Mensajería de Azure Bot a:
        -   `https://:3978/api/messages` (o tu ruta/puerto elegido).
6.  **Ejecuta la pasarela**
    -   El canal de Teams se inicia automáticamente cuando el complemento está instalado y existe la configuración `msteams` con credenciales.

## Contexto del historial

-   `channels.msteams.historyLimit` controla cuántos mensajes recientes de canal/grupo se incluyen en el prompt.
-   Recurre a `messages.groupChat.historyLimit`. Configura `0` para deshabilitar (por defecto 50).
-   El historial de MD se puede limitar con `channels.msteams.dmHistoryLimit` (turnos de usuario). Anulaciones por usuario: `channels.msteams.dms["<user_id>"].historyLimit`.

## Permisos RSC actuales de Teams (Manifiesto)

Estos son los **permisos específicos de recursos existentes** en nuestro manifiesto de aplicación de Teams. Solo se aplican dentro del equipo/chat donde la aplicación está instalada. **Para canales (ámbito de equipo):**

-   `ChannelMessage.Read.Group` (Aplicación) - recibir todos los mensajes del canal sin @mención
-   `ChannelMessage.Send.Group` (Aplicación)
-   `Member.Read.Group` (Aplicación)
-   `Owner.Read.Group` (Aplicación)
-   `ChannelSettings.Read.Group` (Aplicación)
-   `TeamMember.Read.Group` (Aplicación)
-   `TeamSettings.Read.Group` (Aplicación)

**Para chats grupales:**

-   `ChatMessage.Read.Chat` (Aplicación) - recibir todos los mensajes del chat grupal sin @mención

## Ejemplo de Manifiesto de Teams (redactado)

Ejemplo mínimo y válido con los campos requeridos. Reemplaza los ID y URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Tu Organización",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw en Teams", "full": "OpenClaw en Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### Advertencias del manifiesto (campos obligatorios)

-   `bots[].botId` **debe** coincidir con el ID de aplicación de Azure Bot.
-   `webApplicationInfo.id` **debe** coincidir con el ID de aplicación de Azure Bot.
-   `bots[].scopes` debe incluir las superficies que planeas usar (`personal`, `team`, `groupChat`).
-   `bots[].supportsFiles: true` es requerido para el manejo de archivos en ámbito personal.
-   `authorization.permissions.resourceSpecific` debe incluir lectura/envío de canal si quieres tráfico de canal.

### Actualizar una aplicación existente

Para actualizar una aplicación de Teams ya instalada (ej., para agregar permisos RSC):

1.  Actualiza tu `manifest.json` con la nueva configuración
2.  **Incrementa el campo `version`** (ej., `1.0.0` → `1.1.0`)
3.  **Vuelve a comprimir** el manifiesto con los iconos (`manifest.json`, `outline.png`, `color.png`)
4.  Sube el nuevo zip:
    -   **Opción A (Centro de administración de Teams):** Centro de administración de Teams → Aplicaciones de Teams → Administrar aplicaciones → encuentra tu aplicación → Cargar nueva versión
    -   **Opción B (Carga lateral):** En Teams → Aplicaciones → Administrar tus aplicaciones → Cargar una aplicación personalizada
5.  **Para canales de equipo:** Reinstala la aplicación en cada equipo para que los nuevos permisos surtan efecto
6.  **Cierra y reinicia Teams por completo** (no solo cerrar la ventana) para borrar los metadatos de la aplicación en caché

## Capacidades: Solo RSC vs Graph

### Con solo RSC de Teams (aplicación instalada, sin permisos de Graph API)

Funciona:

-   Leer contenido de **texto** de mensajes de canal.
-   Enviar contenido de **texto** de mensajes de canal.
-   Recibir archivos adjuntos en **MD (personal)**.

NO funciona:

-   **Contenido de imágenes o archivos** en canal/grupo (la carga útil solo incluye un fragmento HTML).
-   Descargar archivos adjuntos almacenados en SharePoint/OneDrive.
-   Leer historial de mensajes (más allá del evento de webhook en vivo).

### Con RSC de Teams + permisos de Aplicación de Microsoft Graph

Agrega:

-   Descargar contenidos alojados (imágenes pegadas en mensajes).
-   Descargar archivos adjuntos almacenados en SharePoint/OneDrive.
-   Leer historial de mensajes de canal/chat mediante Graph.

### RSC vs Graph API

| Capacidad | Permisos RSC | Graph API |
| --- | --- | --- |
| **Mensajes en tiempo real** | Sí (vía webhook) | No (solo sondeo) |
| **Mensajes históricos** | No | Sí (puede consultar historial) |
| **Complejidad de configuración** | Solo manifiesto de aplicación | Requiere consentimiento de administrador + flujo de token |
| **Funciona sin conexión** | No (debe estar ejecutándose) | Sí (consultar en cualquier momento) |

**Conclusión:** RSC es para escuchar en tiempo real; Graph API es para acceso histórico. Para ponerse al día con mensajes perdidos mientras está sin conexión, necesitas Graph API con `ChannelMessage.Read.All` (requiere consentimiento de administrador).

## Medios e historial habilitados por Graph (requerido para canales)

Si necesitas imágenes/archivos en **canales** o quieres obtener **historial de mensajes**, debes habilitar permisos de Microsoft Graph y otorgar consentimiento de administrador.

1.  En Entra ID (Azure AD) **Registro de aplicación**, agrega **Permisos de aplicación** de Microsoft Graph:
    -   `ChannelMessage.Read.All` (archivos adjuntos de canal + historial)
    -   `Chat.Read.All` o `ChatMessage.Read.All` (chats grupales)
2.  **Otorga consentimiento de administrador** para el inquilino.
3.  Incrementa la **versión del manifiesto** de la aplicación de Teams, vuelve a subirla y **reinstala la aplicación en Teams**.
4.  **Cierra y reinicia Teams por completo** para borrar los metadatos de la aplicación en caché.

**Permiso adicional para menciones de usuario:** Las menciones @ de usuario funcionan de inmediato para usuarios en la conversación. Sin embargo, si quieres buscar dinámicamente y mencionar usuarios que **no están en la conversación actual**, agrega el permiso `User.Read.All` (Aplicación) y otorga consentimiento de administrador.

## Limitaciones conocidas

### Tiempos de espera del webhook

Teams entrega mensajes a través de webhook HTTP. Si el procesamiento tarda demasiado (ej., respuestas lentas del LLM), puedes ver:

-   Tiempos de espera de la pasarela
-   Teams reintentando el mensaje (causando duplicados)
-   Respuestas perdidas

OpenClaw maneja esto devolviendo una respuesta rápidamente y enviando respuestas de forma proactiva, pero respuestas muy lentas aún pueden causar problemas.

### Formato

El markdown de Teams es más limitado que Slack o Discord:

-   El formato básico funciona: **negrita**, *cursiva*, `código`, enlaces
-   El markdown complejo (tablas, listas anidadas) puede no renderizarse correctamente
-   Las Adaptive Cards son compatibles para encuestas y envíos de tarjetas arbitrarias (ver abajo)

## Configuración

Ajustes clave (ver `/gateway/configuration` para patrones de canal compartidos):

-   `channels.msteams.enabled`: habilitar/deshabilitar el canal.
-   `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: credenciales del bot.
-   `channels.msteams.webhook.port` (por defecto `3978`)
-   `channels.msteams.webhook.path` (por defecto `/api/messages`)
-   `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (por defecto: pairing)
-   `channels.msteams.allowFrom`: lista de MD permitidos (se recomiendan ID de objetos AAD). El asistente resuelve nombres a ID durante la configuración cuando hay acceso a Graph disponible.
-   `channels.msteams.dangerouslyAllowNameMatching`: interruptor de emergencia para re-habilitar la coincidencia de UPN/nombre para mostrar mutable.
-   `channels.msteams.textChunkLimit`: tamaño de fragmento de texto saliente.
-   `channels.msteams.chunkMode`: `length` (por defecto) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de la fragmentación por longitud.
-   `channels.msteams.mediaAllowHosts`: lista de hosts permitidos para archivos adjuntos entrantes (por defecto dominios de Microsoft/Teams).
-   `channels.msteams.mediaAuthAllowHosts`: lista de hosts permitidos para adjuntar encabezados Authorization en reintentos de medios (por defecto hosts de Graph + Bot Framework).
-   `channels.msteams.requireMention`: requerir @mención en canales/grupos (por defecto true).
-   `channels.msteams.replyStyle`: `thread | top-level` (ver [Estilo de respuesta](#estilo-de-respuesta-hilos-vs-publicaciones)).
-   `channels.msteams.teams..replyStyle`: anulación por equipo.
-   `channels.msteams.teams..requireMention`: anulación por equipo.
-   `channels.msteams.teams..tools`: anulaciones de política de herramientas por equipo predeterminadas (`allow`/`deny`/`alsoAllow`) usadas cuando falta una anulación de canal.
-   `channels.msteams.teams..toolsBySender`: anulaciones de política de herramientas por remitente por equipo predeterminadas (se admite el comodín `"*"`).
-   `channels.msteams.teams..channels..replyStyle`: anulación por canal.
-   `channels.msteams.teams..channels..requireMention`: anulación por canal.
-   `channels.msteams.teams..channels..tools`: anulaciones de política de herramientas por canal (`allow`/`deny`/`alsoAllow`).
-   `channels.msteams.teams..channels..toolsBySender`: anulaciones de política de herramientas por remitente por canal (se admite el comodín `"*"`).
-   Las claves de `toolsBySender` deben usar prefijos explícitos: `id:`, `e164:`, `username:`, `name:` (las claves sin prefijo heredadas aún se asignan solo a `id:`).
-   `channels.msteams.sharePointSiteId`: ID del sitio de SharePoint para cargas de archivos en chats grupales/canales (ver [Enviar archivos en chats grupales](#enviar-archivos-en-chats-grupales)).

## Enrutamiento y Sesiones

-   Las claves de sesión siguen el formato estándar de agente (ver [/concepts/session](../concepts/session.md)):
    -   Los mensajes directos comparten la sesión principal (`agent::`).
    -   Los mensajes de canal/grupo usan el id de conversación:
        -   `agent::msteams:channel:`
        -   `agent::msteams:group:`

## Estilo de respuesta: Hilos vs Publicaciones

Teams introdujo recientemente dos estilos de UI de canal sobre el mismo modelo de datos subyacente:

| Estilo | Descripción | `replyStyle` recomendado |
| --- | --- | --- |
| **Publicaciones** (clásico) | Los mensajes aparecen como tarjetas con respuestas en hilo debajo | `thread` (por defecto) |
| **Hilos** (similar a Slack) | Los mensajes fluyen linealmente, más como Slack | `top-level` |

**El problema:** La API de Teams no expone qué estilo de UI usa un canal. Si usas el `replyStyle` incorrecto:

-   `thread` en un canal con estilo Hilos → las respuestas aparecen anidadas de manera incómoda
-   `top-level` en un canal con estilo Publicaciones → las respuestas aparecen como publicaciones de nivel superior separadas en lugar de dentro del hilo

**Solución:** Configura `replyStyle` por canal según cómo esté configurado el canal:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## Archivos adjuntos e Imágenes

**Limitaciones actuales:**

-   **MD:** Las imágenes y archivos adjuntos funcionan mediante las API de archivos del bot de Teams.
-   **Canales/grupos:** Los archivos adjuntos residen en almacenamiento M365 (SharePoint/OneDrive). La carga útil del webhook solo incluye un fragmento HTML, no los bytes reales del archivo. **Se requieren permisos de Graph API** para descargar archivos adjuntos de canal.

Sin permisos de Graph, los mensajes de canal con imágenes se recibirán solo como texto (el contenido de la imagen no es accesible para el bot). Por defecto, OpenClaw solo descarga medios de nombres de host de Microsoft/Teams. Anula con `channels.msteams.mediaAllowHosts` (usa `["*"]` para permitir cualquier host). Los encabezados Authorization solo se adjuntan para hosts en `channels.msteams.mediaAuthAllowHosts` (por defecto hosts de Graph + Bot Framework). Mantén esta lista estricta (evita sufijos multiinquilino).

## Enviar archivos en chats grupales

Los bots pueden enviar archivos en MD usando el flujo FileConsentCard (integrado). Sin embargo, **enviar archivos en chats grupales/canales** requiere configuración adicional:

| Contexto | Cómo se envían los archivos | Configuración necesaria |
| --- | --- | --- |
| **MD** | FileConsentCard → usuario acepta → bot sube | Funciona de inmediato |
| **Chats grupales/canales** | Subir a SharePoint → compartir enlace | Requiere `sharePointSiteId` + permisos de Graph |
| **Imágenes (cualquier contexto)** | Codificado en línea en Base64 | Funciona de inmediato |

### Por qué los chats grupales necesitan SharePoint

Los bots no tienen una unidad personal de OneDrive (el punto final de Graph API `/me/drive` no funciona para identidades de aplicación). Para enviar archivos en chats grupales/canales, el bot sube a un **sitio de SharePoint** y crea un enlace de uso compartido.

### Configuración

1.  **Agrega permisos de Graph API** en Entra ID (Azure AD) → Registro de aplicación:
    -   `Sites.ReadWrite.All` (Aplicación) - subir archivos a SharePoint
    -   `Chat.Read.All` (Aplicación) - opcional, habilita enlaces de uso compartido por usuario
2.  **Otorga consentimiento de administrador** para el inquilino.
3.  **Obtén tu ID de sitio de SharePoint:**
    
    Copiar
    
    ```bash
    # Mediante Graph Explorer o curl con un token válido:
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"
    
    # Ejemplo: para un sitio en "contoso.sharepoint.com/sites/BotFiles"
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"
    
    # La respuesta incluye: "id": "contoso.sharepoint.com,guid1,guid2"
    ```
    
4.  **Configura OpenClaw:**
    
    Copiar
    
    ```json
    {
      channels: {
        msteams: {
          // ... otra configuración ...
          sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
        },
      },
    }
    ```
    

### Comportamiento de uso compartido

| Permiso | Comportamiento de uso compartido |
| --- | --- |
| Solo `Sites.ReadWrite.All` | Enlace de uso compartido para toda la organización (cualquiera en la org puede acceder) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Enlace de uso compartido por usuario (solo los miembros del chat pueden acceder) |

El uso compartido por usuario es más seguro ya que solo los participantes del chat pueden acceder al archivo. Si falta el permiso `Chat.Read.All`, el bot recurre al uso compartido para toda la organización.

### Comportamiento de respaldo

| Escenario | Resultado |
| --- | --- |
| Chat grupal + archivo + `sharePointSiteId` configurado | Subir a SharePoint, enviar enlace de uso compartido |
| Chat grupal + archivo + sin `sharePointSiteId` | Intentar subida a OneDrive (puede fallar), enviar solo texto |
| Chat personal + archivo | Flujo FileConsentCard (funciona sin SharePoint) |
| Cualquier contexto + imagen | Codificado en línea en Base64 (funciona sin SharePoint) |

### Ubicación de almacenamiento de archivos

Los archivos subidos se almacenan en una carpeta `/OpenClawShared/` en la biblioteca de documentos predeterminada del sitio de SharePoint configurado.

## Encuestas (Adaptive Cards)

OpenClaw envía encuestas de Teams como Adaptive Cards (no hay una API nativa