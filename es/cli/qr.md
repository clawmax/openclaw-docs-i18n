

  Comandos CLI

  
# qr

Genera un código QR de emparejamiento para iOS y un código de configuración a partir de la configuración actual de tu Gateway.

## Uso

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## Opciones

-   `--remote`: usa `gateway.remote.url` más el token/contraseña remota de la configuración
-   `--url `: anula la URL del gateway usada en la carga útil
-   `--public-url `: anula la URL pública usada en la carga útil
-   `--token `: anula el token del gateway para la carga útil
-   `--password `: anula la contraseña del gateway para la carga útil
-   `--setup-code-only`: imprime solo el código de configuración
-   `--no-ascii`: omite la representación ASCII del QR
-   `--json`: emite JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## Notas

-   `--token` y `--password` son mutuamente excluyentes.
-   Con `--remote`, si las credenciales remotas efectivamente activas están configuradas como SecretRefs y no pasas `--token` o `--password`, el comando las resuelve desde la instantánea activa del gateway. Si el gateway no está disponible, el comando falla rápidamente.
-   Sin `--remote`, las SecretRefs de autenticación del gateway local se resuelven cuando no se pasa ninguna anulación de autenticación por CLI:
    -   `gateway.auth.token` se resuelve cuando la autenticación por token puede ganar (`gateway.auth.mode="token"` explícito o modo inferido donde ninguna fuente de contraseña gana).
    -   `gateway.auth.password` se resuelve cuando la autenticación por contraseña puede ganar (`gateway.auth.mode="password"` explícito o modo inferido sin un token ganador de auth/env).
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados (incluyendo SecretRefs) y `gateway.auth.mode` no está establecido, la resolución del código de configuración falla hasta que el modo se establece explícitamente.
-   Nota sobre desfase de versión del gateway: esta ruta de comando requiere un gateway que admita `secrets.resolve`; los gateways más antiguos devuelven un error de método desconocido.
-   Después de escanear, aprueba el emparejamiento del dispositivo con:
    -   `openclaw devices list`
    -   `openclaw devices approve `

[plugins](./plugins.md)[reset](./reset.md)