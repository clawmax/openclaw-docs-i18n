

  Sesiones y memoria

  
# Compresión

Cada modelo tiene una **ventana de contexto** (tokens máximos que puede ver). Los chats de larga duración acumulan mensajes y resultados de herramientas; una vez que la ventana se llena, OpenClaw **comprime** el historial más antiguo para mantenerse dentro de los límites.

## Qué es la compresión

La compresión **resume la conversación anterior** en una entrada de resumen compacta y mantiene intactos los mensajes recientes. El resumen se almacena en el historial de la sesión, por lo que las solicitudes futuras utilizan:

-   El resumen de compresión
-   Los mensajes recientes después del punto de compresión

La compresión **persiste** en el historial JSONL de la sesión.

## Configuración

Utiliza la configuración `agents.defaults.compaction` en tu `openclaw.json` para configurar el comportamiento de la compresión (modo, tokens objetivo, etc.). La compresión por resumen preserva los identificadores opacos por defecto (`identifierPolicy: "strict"`). Puedes anular esto con `identifierPolicy: "off"` o proporcionar texto personalizado con `identifierPolicy: "custom"` e `identifierInstructions`.

## Compresión automática (activada por defecto)

Cuando una sesión se acerca o excede la ventana de contexto del modelo, OpenClaw activa la compresión automática y puede reintentar la solicitud original utilizando el contexto comprimido. Verás:

-   `🧹 Compresión automática completada` en modo detallado
-   `/status` mostrando `🧹 Compresiones: `

Antes de la compresión, OpenClaw puede ejecutar un **turno de vaciado de memoria silencioso** para almacenar notas duraderas en el disco. Consulta [Memoria](./memory.md) para más detalles y configuración.

## Compresión manual

Usa `/compact` (opcionalmente con instrucciones) para forzar un paso de compresión:

```bash
/compact Focus on decisions and open questions
```

## Fuente de la ventana de contexto

La ventana de contexto es específica del modelo. OpenClaw utiliza la definición del modelo del catálogo del proveedor configurado para determinar los límites.

## Compresión vs. poda

-   **Compresión**: resume y **persiste** en JSONL.
-   **Poda de sesión**: recorta solo los **resultados de herramientas** antiguos, **en memoria**, por solicitud.

Consulta [/concepts/session-pruning](./session-pruning.md) para más detalles sobre la poda.

## Compresión del lado del servidor de OpenAI

OpenClaw también admite sugerencias de compresión del lado del servidor de OpenAI Responses para modelos directos de OpenAI compatibles. Esto es independiente de la compresión local de OpenClaw y puede ejecutarse junto a ella.

-   Compresión local: OpenClaw resume y persiste en el JSONL de la sesión.
-   Compresión del lado del servidor: OpenAI comprime el contexto en el lado del proveedor cuando `store` + `context_management` están habilitados.

Consulta [Proveedor OpenAI](../providers/openai.md) para parámetros del modelo y anulaciones.

## Consejos

-   Usa `/compact` cuando las sesiones se sientan obsoletas o el contexto esté inflado.
-   Las salidas grandes de herramientas ya están truncadas; la poda puede reducir aún más la acumulación de resultados de herramientas.
-   Si necesitas un estado limpio, `/new` o `/reset` inicia un nuevo ID de sesión.

[Memoria](./memory.md)[Enrutamiento Multi-Agente](./multi-agent.md)