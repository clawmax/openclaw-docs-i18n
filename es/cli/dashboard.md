

  Comandos CLI

  
# dashboard

Abre la Interfaz de Usuario de Control usando tu autenticación actual.

```bash
openclaw dashboard
openclaw dashboard --no-open
```

Notas:

-   `dashboard` resuelve los SecretRefs configurados en `gateway.auth.token` cuando es posible.
-   Para tokens gestionados por SecretRef (resueltos o no resueltos), `dashboard` imprime/copia/abre una URL sin tokenizar para evitar exponer secretos externos en la salida de la terminal, el historial del portapapeles o los argumentos de lanzamiento del navegador.
-   Si `gateway.auth.token` está gestionado por SecretRef pero no se resuelve en esta ruta de comando, el comando imprime una URL sin tokenizar y una guía de remediación explícita en lugar de incrustar un marcador de posición de token no válido.

[daemon](./daemon.md)[devices](./devices.md)