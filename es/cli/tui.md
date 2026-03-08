

  Comandos CLI

  
# tui

Abre la interfaz de usuario de terminal conectada al Gateway. Relacionado:

-   Guía de TUI: [TUI](../web/tui.md)

Notas:

-   `tui` resuelve las referencias SecretRef de autenticación del gateway configuradas para autenticación por token/contraseña cuando es posible (proveedores `env`/`file`/`exec`).

## Ejemplos

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)

---