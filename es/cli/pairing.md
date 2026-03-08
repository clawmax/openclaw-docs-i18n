

  Comandos CLI

  
# pairing

Aprobar o inspeccionar solicitudes de emparejamiento por DM (para canales que admiten emparejamiento). Relacionado:

-   Flujo de emparejamiento: [Emparejamiento](../channels/pairing.md)

## Comandos

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Notas

-   Entrada del canal: pásalo posicionalmente (`pairing list telegram`) o con `--channel `.
-   `pairing list` admite `--account ` para canales multi-cuenta.
-   `pairing approve` admite `--account ` y `--notify`.
-   Si solo hay un canal configurado con capacidad de emparejamiento, se permite `pairing approve `.

[onboard](./onboard.md)[plugins](./plugins.md)

---