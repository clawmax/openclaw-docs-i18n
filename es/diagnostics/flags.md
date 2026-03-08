

  Entorno y depuración

  
# Banderas de Diagnóstico

Las banderas de diagnóstico te permiten habilitar registros de depuración específicos sin activar el registro detallado en todas partes. Las banderas son opcionales y no tienen efecto a menos que un subsistema las verifique.

## Cómo funciona

-   Las banderas son cadenas de texto (sin distinción entre mayúsculas y minúsculas).
-   Puedes habilitar banderas en la configuración o mediante una anulación de variable de entorno.
-   Se admiten comodines:
    -   `telegram.*` coincide con `telegram.http`
    -   `*` habilita todas las banderas

## Habilitar mediante configuración

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Múltiples banderas:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Reinicia el gateway después de cambiar las banderas.

## Anulación por entorno (para casos puntuales)

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Deshabilitar todas las banderas:

```
OPENCLAW_DIAGNOSTICS=0
```

## Dónde van los registros

Las banderas emiten registros en el archivo de registro de diagnóstico estándar. Por defecto:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Si configuras `logging.file`, usa esa ruta en su lugar. Los registros son JSONL (un objeto JSON por línea). La redacción aún se aplica según `logging.redactSensitive`.

## Extraer registros

Selecciona el archivo de registro más reciente:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtrar por diagnósticos de Telegram HTTP:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

O seguir la salida mientras reproduces el problema:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Para gateways remotos, también puedes usar `openclaw logs --follow` (consulta [/cli/logs](../cli/logs.md)).

## Notas

-   Si `logging.level` está configurado más alto que `warn`, estos registros pueden ser suprimidos. El valor por defecto `info` es correcto.
-   Es seguro dejar las banderas habilitadas; solo afectan el volumen de registros para el subsistema específico.
-   Usa [/logging](../logging.md) para cambiar destinos de registro, niveles y redacción.

[Node + tsx Crash](../debug/node-issue.md)[Node.js](../install/node.md)

---