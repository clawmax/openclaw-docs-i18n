title: "Guía de Pruebas de OpenClaw AI: Comandos, Puntos de Referencia y Docker"
description: "Aprende a ejecutar pruebas de OpenClaw AI, incluyendo unitarias, de integración y en vivo. Obtén comandos para cobertura, puntos de referencia y pruebas E2E basadas en Docker."
keywords: ["pruebas openclaw", "comandos vitest", "pruebas e2e docker", "cobertura de pruebas", "puntos de referencia cli", "integración de gateway", "pnpm test", "punto de referencia de latencia de modelo"]
---

  Notas de la versión

  
# Pruebas

-   Kit completo de pruebas (suites, en vivo, Docker): [Testing](../help/testing.md)
-   `pnpm test:force`: Termina cualquier proceso de gateway residual que esté ocupando el puerto de control por defecto, luego ejecuta la suite completa de Vitest con un puerto de gateway aislado para que las pruebas del servidor no choquen con una instancia en ejecución. Usa esto cuando una ejecución previa de gateway dejó ocupado el puerto 18789.
-   `pnpm test:coverage`: Ejecuta la suite unitaria con cobertura V8 (vía `vitest.unit.config.ts`). Los umbrales globales son 70% líneas/ramas/funciones/declaraciones. La cobertura excluye puntos de entrada con mucha integración (cableado CLI, puentes gateway/telegram, servidor estático webchat) para mantener el objetivo centrado en la lógica unitaria.
-   `pnpm test` en Node 24+: OpenClaw desactiva automáticamente los `vmForks` de Vitest y usa `forks` para evitar `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. Puedes forzar el comportamiento con `OPENCLAW_TEST_VM_FORKS=0|1`.
-   `pnpm test`: ejecuta por defecto la vía unitaria principal rápida para una retroalimentación local rápida.
-   `pnpm test:channels`: ejecuta las suites con muchos canales.
-   `pnpm test:extensions`: ejecuta las suites de extensiones/plugins.
-   Integración de Gateway: opcional mediante `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` o `pnpm test:gateway`.
-   `pnpm test:e2e`: Ejecuta pruebas de humo end-to-end del gateway (emparejamiento multi-instancia WS/HTTP/node). Por defecto usa `vmForks` + trabajadores adaptativos en `vitest.e2e.config.ts`; ajusta con `OPENCLAW_E2E_WORKERS=` y establece `OPENCLAW_E2E_VERBOSE=1` para registros detallados.
-   `pnpm test:live`: Ejecuta pruebas en vivo de proveedores (minimax/zai). Requiere claves API y `LIVE=1` (o `*_LIVE_TEST=1` específico del proveedor) para no omitirlas.

## Puerta de PR local

Para las comprobaciones de puerta/aterrizaje de PR local, ejecuta:

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

Si `pnpm test` falla intermitentemente en un host cargado, vuelve a ejecutarlo una vez antes de tratarlo como una regresión, luego aísla con `pnpm vitest run <ruta/a/la/prueba>`. Para hosts con memoria limitada, usa:

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## Punto de referencia de latencia de modelo (claves locales)

Script: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) Uso:

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   Variables de entorno opcionales: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   Prompt por defecto: “Responde con una sola palabra: ok. Sin puntuación ni texto extra.”

Última ejecución (2025-12-31, 20 ejecuciones):

-   minimax mediana 1279ms (mín 1114, máx 2431)
-   opus mediana 2454ms (mín 1224, máx 3170)

## Punto de referencia de inicio de CLI

Script: [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) Uso:

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

Este punto de referencia mide estos comandos:

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

La salida incluye promedio, p50, p95, mínimo/máximo y distribución de código de salida/señal para cada comando.

## E2E de incorporación (Docker)

Docker es opcional; esto solo es necesario para pruebas de humo de incorporación en contenedores. Flujo de inicio en frío completo en un contenedor Linux limpio:

```
scripts/e2e/onboard-docker.sh
```

Este script controla el asistente interactivo a través de una pseudo-tty, verifica los archivos de configuración/espacio de trabajo/sesión, luego inicia el gateway y ejecuta `openclaw health`.

## Prueba de humo de importación QR (Docker)

Asegura que `qrcode-terminal` se cargue bajo Node 22+ en Docker:

```bash
pnpm test:docker:qr
```

[Lista de Verificación de Lanzamiento](./RELEASING.md)[Integración del gateway Kilo](../design/kilo-gateway-integration.md)

---