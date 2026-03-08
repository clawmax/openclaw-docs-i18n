

  Avanzado

  
# Canales de Desarrollo

Última actualización: 2026-01-21 OpenClaw distribuye tres canales de actualización:

-   **estable**: etiqueta de distribución npm `latest`.
-   **beta**: etiqueta de distribución npm `beta` (compilaciones en prueba).
-   **dev**: punta móvil de `main` (git). Etiqueta de distribución npm: `dev` (cuando se publica).

Distribuimos compilaciones a **beta**, las probamos y luego **promovemos una compilación verificada a `latest`** sin cambiar el número de versión — las etiquetas de distribución son la fuente de verdad para las instalaciones npm.

## Cambiando de canal

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

-   `stable`/`beta` extraen la última etiqueta coincidente (a menudo la misma etiqueta).
-   `dev` cambia a `main` y se rebasa sobre el repositorio ascendente.

Instalación global con npm/pnpm:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Esto actualiza mediante la etiqueta de distribución npm correspondiente (`latest`, `beta`, `dev`). Cuando cambias de canal **explícitamente** con `--channel`, OpenClaw también alinea el método de instalación:

-   `dev` asegura un checkout de git (por defecto `~/openclaw`, se puede anular con `OPENCLAW_GIT_DIR`), lo actualiza e instala la CLI global desde ese checkout.
-   `stable`/`beta` instala desde npm usando la etiqueta de distribución coincidente.

Consejo: si quieres estable y dev en paralelo, mantén dos clones y apunta tu puerta de enlace al estable.

## Plugins y canales

Cuando cambias de canal con `openclaw update`, OpenClaw también sincroniza las fuentes de los plugins:

-   `dev` prefiere plugins incluidos desde el checkout de git.
-   `stable` y `beta` restauran paquetes de plugins instalados via npm.

## Mejores prácticas de etiquetado

-   Etiqueta las versiones en las que quieras que aterricen los checkouts de git (`vYYYY.M.D` para estable, `vYYYY.M.D-beta.N` para beta).
-   `vYYYY.M.D.beta.N` también se reconoce por compatibilidad, pero se prefiere `-beta.N`.
-   Las etiquetas heredadas `vYYYY.M.D-` aún se reconocen como estables (no beta).
-   Mantén las etiquetas inmutables: nunca muevas o reutilices una etiqueta.
-   Las etiquetas de distribución npm siguen siendo la fuente de verdad para las instalaciones npm:
    -   `latest` → estable
    -   `beta` → compilación candidata
    -   `dev` → instantánea de main (opcional)

## Disponibilidad de la aplicación para macOS

Las compilaciones beta y dev pueden **no** incluir una versión de la aplicación para macOS. Eso está bien:

-   La etiqueta git y la etiqueta de distribución npm aún se pueden publicar.
-   Indica "no hay compilación para macOS para esta beta" en las notas de la versión o el registro de cambios.

[Desplegar en Northflank](./northflank.md)

---