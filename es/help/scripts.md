

  Entorno y depuración

  
# Scripts

El directorio `scripts/` contiene scripts auxiliares para flujos de trabajo locales y tareas de operaciones. Úsalos cuando una tarea esté claramente vinculada a un script; de lo contrario, prefiere la CLI.

## Convenciones

-   Los scripts son **opcionales** a menos que se haga referencia a ellos en la documentación o listas de verificación de lanzamientos.
-   Prefiere las interfaces de línea de comandos (CLI) cuando existan (ejemplo: el monitoreo de autenticación usa `openclaw models status --check`).
-   Asume que los scripts son específicos del host; léelos antes de ejecutarlos en una máquina nueva.

## Scripts de monitoreo de autenticación

Los scripts de monitoreo de autenticación están documentados aquí: [/automation/auth-monitoring](../automation/auth-monitoring.md)

## Al agregar scripts

-   Mantén los scripts enfocados y documentados.
-   Añade una breve entrada en la documentación relevante (o créala si falta).

[Pruebas](./testing.md)[Fallo de Node + tsx](../debug/node-issue.md)

---