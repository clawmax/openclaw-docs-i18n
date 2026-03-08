

  Comandos CLI

  
# update

Actualiza OpenClaw de forma segura y cambia entre los canales estable/beta/dev. Si instalaste mediante **npm/pnpm** (instalación global, sin metadatos git), las actualizaciones se realizan mediante el flujo del gestor de paquetes en [Actualizando](../install/updating.md).

## Uso

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## Opciones

-   `--no-restart`: omite reiniciar el servicio Gateway después de una actualización exitosa.
-   `--channel <stable|beta|dev>`: establece el canal de actualización (git + npm; se persiste en la configuración).
-   `--tag <dist-tag|version>`: anula la etiqueta de distribución de npm o la versión solo para esta actualización.
-   `--dry-run`: previsualiza las acciones de actualización planificadas (flujo de canal/etiqueta/objetivo/reinicio) sin escribir configuración, instalar, sincronizar complementos o reiniciar.
-   `--json`: imprime JSON `UpdateRunResult` legible por máquina.
-   `--timeout `: tiempo de espera por paso (por defecto 1200s).

Nota: las regresiones de versión requieren confirmación porque versiones más antiguas pueden romper la configuración.

## update status

Muestra el canal de actualización activo + etiqueta/rama/SHA de git (para instalaciones desde código fuente), además de la disponibilidad de actualizaciones.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Opciones:

-   `--json`: imprime JSON de estado legible por máquina.
-   `--timeout `: tiempo de espera para las comprobaciones (por defecto 3s).

## update wizard

Flujo interactivo para elegir un canal de actualización y confirmar si reiniciar el Gateway después de actualizar (por defecto es reiniciar). Si seleccionas `dev` sin una copia de trabajo de git, ofrece crear una.

## Qué hace

Cuando cambias de canal explícitamente (`--channel ...`), OpenClaw también mantiene alineado el método de instalación:

-   `dev` → asegura una copia de trabajo de git (por defecto: `~/openclaw`, anular con `OPENCLAW_GIT_DIR`), la actualiza e instala la CLI global desde esa copia.
-   `stable`/`beta` → instala desde npm usando la etiqueta de distribución correspondiente.

El actualizador automático del núcleo de Gateway (cuando está habilitado mediante configuración) reutiliza esta misma ruta de actualización.

## Flujo de copia de trabajo de Git

Canales:

-   `stable`: cambia a la última etiqueta no-beta, luego compila + doctor.
-   `beta`: cambia a la última etiqueta `-beta`, luego compila + doctor.
-   `dev`: cambia a `main`, luego obtiene cambios + reorganiza.

Resumen:

1.  Requiere un árbol de trabajo limpio (sin cambios sin confirmar).
2.  Cambia al canal seleccionado (etiqueta o rama).
3.  Obtiene cambios del repositorio remoto (solo dev).
4.  Solo dev: comprobación previa de lint + compilación TypeScript en un árbol de trabajo temporal; si la punta falla, retrocede hasta 10 commits para encontrar la compilación limpia más nueva.
5.  Reorganiza sobre el commit seleccionado (solo dev).
6.  Instala dependencias (preferible pnpm; fallback npm).
7.  Compila + compila la Interfaz de Control.
8.  Ejecuta `openclaw doctor` como comprobación final de "actualización segura".
9.  Sincroniza complementos con el canal activo (dev usa extensiones empaquetadas; estable/beta usa npm) y actualiza los complementos instalados con npm.

## Abreviatura \--update

`openclaw --update` se reescribe como `openclaw update` (útil para shells y scripts de lanzador).

## Ver también

-   `openclaw doctor` (ofrece ejecutar update primero en copias de trabajo de git)
-   [Canales de desarrollo](../install/development-channels.md)
-   [Actualizando](../install/updating.md)
-   [Referencia CLI](../cli.md)

[uninstall](./uninstall.md)[voicecall](./voicecall.md)