

  Conceptos internos

  
# Formato Markdown

OpenClaw formatea el Markdown saliente convirtiéndolo en una representación intermedia (IR) compartida antes de renderizar la salida específica del canal. La IR mantiene intacto el texto fuente mientras transporta intervalos de estilo/enlace para que la fragmentación y el renderizado puedan mantenerse consistentes en todos los canales.

## Objetivos

-   **Consistencia:** un paso de análisis, múltiples renderizadores.
-   **Fragmentación segura:** dividir el texto antes de renderizar para que el formato en línea nunca se rompa entre fragmentos.
-   **Adaptación al canal:** mapear la misma IR a Slack mrkdwn, HTML de Telegram y rangos de estilo de Signal sin reanalizar Markdown.

## Proceso

1.  **Analizar Markdown -> IR**
    -   La IR es texto plano más intervalos de estilo (negrita/cursiva/tachado/código/spoiler) e intervalos de enlace.
    -   Los desplazamientos son unidades de código UTF-16 para que los rangos de estilo de Signal se alineen con su API.
    -   Las tablas se analizan solo cuando un canal opta por la conversión de tablas.
2.  **Fragmentar IR (formato primero)**
    -   La fragmentación ocurre en el texto IR antes del renderizado.
    -   El formato en línea no se divide entre fragmentos; los intervalos se dividen por fragmento.
3.  **Renderizar por canal**
    -   **Slack:** tokens mrkdwn (negrita/cursiva/tachado/código), enlaces como `<url|etiqueta>`.
    -   **Telegram:** etiquetas HTML (``, ``, ``, ``, ``, ``).
    -   **Signal:** texto plano + rangos `text-style`; los enlaces se convierten en `etiqueta (url)` cuando la etiqueta difiere.

## Ejemplo de IR

Markdown de entrada:

```bash
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (esquemático):

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## Dónde se utiliza

-   Los adaptadores salientes de Slack, Telegram y Signal renderizan desde la IR.
-   Otros canales (WhatsApp, iMessage, MS Teams, Discord) aún usan texto plano o sus propias reglas de formato, con la conversión de tablas Markdown aplicada antes de la fragmentación cuando está habilitada.

## Manejo de tablas

Las tablas Markdown no son compatibles de manera consistente en todos los clientes de chat. Usa `markdown.tables` para controlar la conversión por canal (y por cuenta).

-   `code`: renderizar tablas como bloques de código (predeterminado para la mayoría de los canales).
-   `bullets`: convertir cada fila en puntos de viñeta (predeterminado para Signal + WhatsApp).
-   `off`: deshabilitar el análisis y conversión de tablas; el texto de tabla crudo pasa a través.

Claves de configuración:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## Reglas de fragmentación

-   Los límites de fragmento provienen de adaptadores/configuración del canal y se aplican al texto IR.
-   Los bloques de código delimitados se conservan como un solo bloque con una nueva línea final para que los canales los rendericen correctamente.
-   Los prefijos de lista y los prefijos de cita en bloque son parte del texto IR, por lo que la fragmentación no divide a la mitad del prefijo.
-   Los estilos en línea (negrita/cursiva/tachado/código-en-línea/spoiler) nunca se dividen entre fragmentos; el renderizador reabre los estilos dentro de cada fragmento.

Si necesitas más sobre el comportamiento de fragmentación en los canales, consulta [Streaming + fragmentación](./streaming.md).

## Política de enlaces

-   **Slack:** `[etiqueta](url)` -> `<url|etiqueta>`; las URLs desnudas permanecen desnudas. El autovínculo está deshabilitado durante el análisis para evitar doble enlace.
-   **Telegram:** `[etiqueta](url)` -> `etiqueta
` (modo de análisis HTML).
-   **Signal:** `[etiqueta](url)` -> `etiqueta (url)` a menos que la etiqueta coincida con la URL.

## Spoilers

Los marcadores de spoiler (`||spoiler||`) se analizan solo para Signal, donde se mapean a rangos de estilo SPOILER. Otros canales los tratan como texto plano.

## Cómo agregar o actualizar un formateador de canal

1.  **Analizar una vez:** usa el ayudante compartido `markdownToIR(...)` con opciones apropiadas para el canal (autovínculo, estilo de encabezado, prefijo de cita en bloque).
2.  **Renderizar:** implementa un renderizador con `renderMarkdownWithMarkers(...)` y un mapa de marcadores de estilo (o rangos de estilo de Signal).
3.  **Fragmentar:** llama a `chunkMarkdownIR(...)` antes de renderizar; renderiza cada fragmento.
4.  **Conectar adaptador:** actualiza el adaptador saliente del canal para usar el nuevo fragmentador y renderizador.
5.  **Probar:** agrega o actualiza pruebas de formato y una prueba de entrega saliente si el canal usa fragmentación.

## Errores comunes

-   Los tokens de corchetes angulares de Slack (`<@U123>`, `<#C123>`, `<https://...>`) deben preservarse; escapa el HTML crudo de forma segura.
-   El HTML de Telegram requiere escapar el texto fuera de las etiquetas para evitar marcado roto.
-   Los rangos de estilo de Signal dependen de desplazamientos UTF-16; no uses desplazamientos de puntos de código.
-   Preserva las nuevas líneas finales para los bloques de código delimitados para que los marcadores de cierre queden en su propia línea.

[TypeBox](./typebox.md)[Indicadores de escritura](./typing-indicators.md)