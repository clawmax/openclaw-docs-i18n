

  Comandos CLI

  
# completion

Genera scripts de completado de shell y, opcionalmente, instálalos en tu perfil de shell.

## Uso

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## Opciones

-   `-s, --shell `: shell objetivo (`zsh`, `bash`, `powershell`, `fish`; predeterminado: `zsh`)
-   `-i, --install`: instala el completado agregando una línea de origen a tu perfil de shell
-   `--write-state`: escribe el/los script(s) de completado en `$OPENCLAW_STATE_DIR/completions` sin imprimir en stdout
-   `-y, --yes`: omite las confirmaciones de instalación

## Notas

-   `--install` escribe un pequeño bloque "OpenClaw Completion" en tu perfil de shell y lo apunta al script en caché.
-   Sin `--install` o `--write-state`, el comando imprime el script en stdout.
-   La generación de completado carga de forma ansiosa los árboles de comandos, por lo que se incluyen los subcomandos anidados.

[clawbot](./clawbot.md)[config](./config.md)

---