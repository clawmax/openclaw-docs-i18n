

  Comandos CLI

  
# logs

Sigue (tail) los logs de archivos del Gateway a través de RPC (funciona en modo remoto). Relacionado:

-   Descripción general del registro: [Registro (Logging)](../logging.md)

## Ejemplos

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Usa `--local-time` para mostrar las marcas de tiempo en tu zona horaria local.

[hooks](./hooks.md)[memory](./memory.md)

---