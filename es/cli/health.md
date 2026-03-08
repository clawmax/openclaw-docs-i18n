

  Comandos CLI

  
# health

Obtén el estado de salud del Gateway en ejecución.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notas:

-   `--verbose` ejecuta sondeos en vivo e imprime tiempos por cuenta cuando hay múltiples cuentas configuradas.
-   La salida incluye almacenes de sesión por agente cuando hay múltiples agentes configurados.

[gateway](./gateway.md)[hooks](./hooks.md)

---