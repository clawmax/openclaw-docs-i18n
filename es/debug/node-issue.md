

  Entorno y depuración

  
# Fallo de Node + tsx

Esta página trata la resolución de fallos de **Node.js y tsx** al ejecutar la puerta de enlace o la CLI de OpenClaw (p. ej. `node --import tsx` o scripts que usan tsx).

## Causas habituales

- **Versión de Node** : OpenClaw requiere Node 22+. Versiones anteriores pueden provocar fallos en tiempo de ejecución o en módulos nativos.
- **tsx / cargador** : Los fallos al arrancar o al cargar TypeScript pueden deberse a la versión de tsx, al orden de `--import` o a cargadores en conflicto.
- **Memoria / nativo** : Cargas grandes o complementos nativos pueden provocar OOM o segfaults; consulta [Node.js](../install/node.md) para la versión y el entorno.

## Comprobaciones rápidas

1. **Versión de Node** : Ejecutar `node -v` (se espera v22 o superior).
2. **Reproducir sin tsx** : Probar el mismo flujo con `node` solo y JS precompilado para ver si el fallo es específico de tsx.
3. **Diagnóstico** : Usar los [indicadores de diagnóstico](../diagnostics/flags.md) (p. ej. `OPENCLAW_DEBUG_*`) para obtener más registros antes de un fallo.

## Relacionado

- [Depuración](../help/debugging.md) — sobreescrituras en runtime, modo watch de la puerta de enlace, perfil dev
- [Scripts](../help/scripts.md) — scripts auxiliares y convenciones
- [Node.js](../install/node.md) — versión de Node y PATH
- [Indicadores de diagnóstico](../diagnostics/flags.md) — indicadores de depuración y traza
