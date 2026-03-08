

  Comandos CLI

  
# system

Ayudantes a nivel del sistema para la puerta de enlace: poner en cola eventos del sistema, controlar latidos y ver la presencia.

## Comandos comunes

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## system event

Pone en cola un evento del sistema en la sesión **principal**. El siguiente latido lo inyectará como una línea `System:` en el prompt. Usa `--mode now` para activar el latido inmediatamente; `next-heartbeat` espera al siguiente ciclo programado. Banderas:

-   `--text `: texto del evento del sistema (requerido).
-   `--mode `: `now` o `next-heartbeat` (predeterminado).
-   `--json`: salida legible por máquina.

## system heartbeat last|enable|disable

Controles de latido:

-   `last`: muestra el último evento de latido.
-   `enable`: reactiva los latidos (usa esto si fueron desactivados).
-   `disable`: pausa los latidos.

Banderas:

-   `--json`: salida legible por máquina.

## system presence

Enumera las entradas de presencia del sistema actuales que la puerta de enlace conoce (nodos, instancias y líneas de estado similares). Banderas:

-   `--json`: salida legible por máquina.

## Notas

-   Requiere una puerta de enlace en ejecución accesible por tu configuración actual (local o remota).
-   Los eventos del sistema son efímeros y no persisten tras reinicios.

[status](./status.md)[tui](./tui.md)

---