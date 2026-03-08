

  Comandos CLI

  
# memory

Gestiona la indexación y búsqueda de memoria semántica. Proporcionado por el plugin de memoria activo (por defecto: `memory-core`; establece `plugins.slots.memory = "none"` para deshabilitarlo). Relacionado:

-   Concepto de Memoria: [Memoria](../concepts/memory.md)
-   Plugins: [Plugins](../tools/plugin.md)

## Ejemplos

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Opciones

`memory status` y `memory index`:

-   `--agent `: limita el alcance a un solo agente. Sin esto, estos comandos se ejecutan para cada agente configurado; si no hay lista de agentes configurada, recurren al agente por defecto.
-   `--verbose`: emite registros detallados durante las pruebas y la indexación.

`memory status`:

-   `--deep`: prueba la disponibilidad de vectores + embeddings.
-   `--index`: ejecuta una reindexación si el almacén está sucio (implica `--deep`).
-   `--json`: imprime la salida en JSON.

`memory index`:

-   `--force`: fuerza una reindexación completa.

`memory search`:

-   Entrada de consulta: pasa ya sea el argumento posicional `[query]` o `--query `.
-   Si se proporcionan ambos, `--query` tiene prioridad.
-   Si no se proporciona ninguno, el comando termina con un error.
-   `--agent `: limita el alcance a un solo agente (por defecto: el agente por defecto).
-   `--max-results `: limita el número de resultados devueltos.
-   `--min-score `: filtra las coincidencias con puntuación baja.
-   `--json`: imprime los resultados en JSON.

Notas:

-   `memory index --verbose` imprime detalles por fase (proveedor, modelo, fuentes, actividad por lotes).
-   `memory status` incluye cualquier ruta extra configurada mediante `memorySearch.extraPaths`.
-   Si los campos de clave de API remota de memoria efectivamente activos están configurados como SecretRefs, el comando resuelve esos valores desde la instantánea activa del gateway. Si el gateway no está disponible, el comando falla rápidamente.
-   Nota sobre desfase de versión del gateway: esta ruta de comando requiere un gateway que soporte `secrets.resolve`; los gateways más antiguos devuelven un error de método desconocido.

[logs](./logs.md)[message](./message.md)