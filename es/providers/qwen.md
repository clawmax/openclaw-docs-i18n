

  Proveedores

  
# Qwen

Qwen proporciona un flujo OAuth de nivel gratuito para los modelos Qwen Coder y Qwen Vision (2,000 solicitudes/día, sujeto a los límites de tasa de Qwen).

## Habilitar el plugin

```bash
openclaw plugins enable qwen-portal-auth
```

Reinicia el Gateway después de habilitarlo.

## Autenticar

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Esto ejecuta el flujo OAuth de código de dispositivo de Qwen y escribe una entrada de proveedor en tu `models.json` (más un alias `qwen` para cambiar rápidamente).

## IDs de Modelo

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

Cambia de modelo con:

```bash
openclaw models set qwen-portal/coder-model
```

## Reutilizar el inicio de sesión de la CLI de Qwen Code

Si ya has iniciado sesión con la CLI de Qwen Code, OpenClaw sincronizará las credenciales desde `~/.qwen/oauth_creds.json` cuando cargue el almacén de autenticación. Aún necesitas una entrada `models.providers.qwen-portal` (usa el comando de inicio de sesión anterior para crear una).

## Notas

-   Los tokens se actualizan automáticamente; vuelve a ejecutar el comando de inicio de sesión si la actualización falla o se revoca el acceso.
-   URL base por defecto: `https://portal.qwen.ai/v1` (anula con `models.providers.qwen-portal.baseUrl` si Qwen proporciona un endpoint diferente).
-   Consulta [Proveedores de modelos](../concepts/model-providers.md) para reglas generales de los proveedores.

[Qianfan](./qianfan.md)[Synthetic](./synthetic.md)

---