

  Comandos CLI

  
# devices

Gestiona solicitudes de emparejamiento de dispositivos y tokens con alcance de dispositivo.

## Comandos

### openclaw devices list

Lista las solicitudes de emparejamiento pendientes y los dispositivos emparejados.

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

Elimina una entrada de dispositivo emparejado.

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

Limpia dispositivos emparejados de forma masiva.

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

Aprueba una solicitud de emparejamiento de dispositivo pendiente. Si se omite `requestId`, OpenClaw aprueba automáticamente la solicitud pendiente más reciente.

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

Rechaza una solicitud de emparejamiento de dispositivo pendiente.

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

Rota un token de dispositivo para un rol específico (opcionalmente actualizando los alcances).

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

Revoca un token de dispositivo para un rol específico.

```bash
openclaw devices revoke --device <deviceId> --role node
```

## Opciones comunes

-   `--url `: URL WebSocket del Gateway (por defecto `gateway.remote.url` cuando está configurado).
-   `--token `: Token del Gateway (si es requerido).
-   `--password `: Contraseña del Gateway (autenticación por contraseña).
-   `--timeout `: Tiempo de espera RPC.
-   `--json`: Salida JSON (recomendado para scripting).

Nota: cuando configuras `--url`, la CLI no recurre a las credenciales de configuración o entorno. Pasa `--token` o `--password` explícitamente. La falta de credenciales explícitas es un error.

## Notas

-   La rotación de token devuelve un nuevo token (sensible). Trátalo como un secreto.
-   Estos comandos requieren el alcance `operator.pairing` (o `operator.admin`).
-   `devices clear` está intencionalmente protegido por `--yes`.
-   Si el alcance de emparejamiento no está disponible en el bucle local (y no se pasa un `--url` explícito), list/approve puede usar un respaldo de emparejamiento local.

[dashboard](./dashboard.md)[directory](./directory.md)

---