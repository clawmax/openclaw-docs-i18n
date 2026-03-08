

  Medios y dispositivos

  
# Comando de Ubicación

## TL;DR

-   `location.get` es un comando de nodo (a través de `node.invoke`).
-   Desactivado por defecto.
-   La configuración usa un selector: Desactivado / Mientras se usa / Siempre.
-   Alternar separado: Ubicación Precisa.

## Por qué un selector (no solo un interruptor)

Los permisos del sistema operativo son multinivel. Podemos exponer un selector en la aplicación, pero el SO aún decide la concesión real.

-   iOS/macOS: el usuario puede elegir **Mientras se usa** o **Siempre** en las solicitudes del sistema/Configuración. La app puede solicitar una actualización, pero el SO puede requerir ir a Configuración.
-   Android: la ubicación en segundo plano es un permiso separado; en Android 10+ a menudo requiere un flujo en Configuración.
-   La ubicación precisa es una concesión separada (iOS 14+ "Precisa", Android "fina" vs "aproximada").

El selector en la interfaz de usuario impulsa nuestro modo solicitado; la concesión real reside en la configuración del SO.

## Modelo de configuración

Por dispositivo de nodo:

-   `location.enabledMode`: `off | whileUsing | always`
-   `location.preciseEnabled`: bool

Comportamiento de la interfaz de usuario:

-   Seleccionar `whileUsing` solicita permiso en primer plano.
-   Seleccionar `always` primero asegura `whileUsing`, luego solicita permiso en segundo plano (o envía al usuario a Configuración si es necesario).
-   Si el SO niega el nivel solicitado, revertir al nivel más alto concedido y mostrar el estado.

## Mapeo de permisos (node.permissions)

Opcional. El nodo macOS reporta `location` a través del mapa de permisos; iOS/Android pueden omitirlo.

## Comando: location.get

Se llama a través de `node.invoke`. Parámetros (sugeridos):

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Respuesta:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Errores (códigos estables):

-   `LOCATION_DISABLED`: el selector está desactivado.
-   `LOCATION_PERMISSION_REQUIRED`: falta permiso para el modo solicitado.
-   `LOCATION_BACKGROUND_UNAVAILABLE`: la aplicación está en segundo plano pero solo se permite Mientras se usa.
-   `LOCATION_TIMEOUT`: no se obtuvo ubicación a tiempo.
-   `LOCATION_UNAVAILABLE`: fallo del sistema / sin proveedores.

## Comportamiento en segundo plano (futuro)

Objetivo: el modelo puede solicitar ubicación incluso cuando el nodo está en segundo plano, pero solo cuando:

-   El usuario seleccionó **Siempre**.
-   El SO concede ubicación en segundo plano.
-   La aplicación puede ejecutarse en segundo plano para ubicación (modo en segundo plano de iOS / servicio en primer plano de Android o autorización especial).

Flujo activado por push (futuro):

1.  La puerta de enlace envía un push al nodo (push silencioso o datos FCM).
2.  El nodo se activa brevemente y solicita ubicación al dispositivo.
3.  El nodo reenvía la respuesta a la puerta de enlace.

Notas:

-   iOS: Se requiere permiso Siempre + modo de ubicación en segundo plano. El push silencioso puede ser limitado; esperar fallos intermitentes.
-   Android: la ubicación en segundo plano puede requerir un servicio en primer plano; de lo contrario, esperar denegación.

## Integración con modelo/herramientas

-   Superficie de herramientas: la herramienta `nodes` agrega la acción `location_get` (se requiere nodo).
-   CLI: `openclaw nodes location get --node `.
-   Pautas para agentes: llamar solo cuando el usuario haya habilitado la ubicación y comprenda el alcance.

## Textos de interfaz de usuario (sugeridos)

-   Desactivado: "El intercambio de ubicación está deshabilitado."
-   Mientras se usa: "Solo cuando OpenClaw está abierto."
-   Siempre: "Permitir ubicación en segundo plano. Requiere permiso del sistema."
-   Precisa: "Usar ubicación GPS precisa. Desactívalo para compartir ubicación aproximada."

[Voice Wake](./voicewake.md)[Text-to-Speech](../tts.md)

---