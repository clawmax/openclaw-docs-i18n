

  Proveedores

  
# Amazon Bedrock

OpenClaw puede usar modelos de **Amazon Bedrock** a través del proveedor de streaming **Bedrock Converse** de pi‑ai. La autenticación de Bedrock utiliza la **cadena de credenciales predeterminada del SDK de AWS**, no una clave API.

## Lo que pi‑ai soporta

-   Proveedor: `amazon-bedrock`
-   API: `bedrock-converse-stream`
-   Autenticación: Credenciales de AWS (variables de entorno, configuración compartida o rol de instancia)
-   Región: `AWS_REGION` o `AWS_DEFAULT_REGION` (predeterminada: `us-east-1`)

## Descubrimiento automático de modelos

Si se detectan credenciales de AWS, OpenClaw puede descubrir automáticamente los modelos de Bedrock que admiten **streaming** y **salida de texto**. El descubrimiento usa `bedrock:ListFoundationModels` y se almacena en caché (predeterminado: 1 hora). Las opciones de configuración se encuentran en `models.bedrockDiscovery`:

```json
{
  models: {
    bedrockDiscovery: {
      enabled: true,
      region: "us-east-1",
      providerFilter: ["anthropic", "amazon"],
      refreshInterval: 3600,
      defaultContextWindow: 32000,
      defaultMaxTokens: 4096,
    },
  },
}
```

Notas:

-   `enabled` es `true` por defecto cuando hay credenciales de AWS presentes.
-   `region` usa por defecto `AWS_REGION` o `AWS_DEFAULT_REGION`, luego `us-east-1`.
-   `providerFilter` coincide con los nombres de los proveedores de Bedrock (por ejemplo `anthropic`).
-   `refreshInterval` está en segundos; configúralo en `0` para deshabilitar la caché.
-   `defaultContextWindow` (predeterminado: `32000`) y `defaultMaxTokens` (predeterminado: `4096`) se usan para los modelos descubiertos (anúlalos si conoces los límites de tu modelo).

## Incorporación

1.  Asegúrate de que las credenciales de AWS estén disponibles en el **host de la puerta de enlace**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Opcional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Opcional (clave API/token de portador de Bedrock):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2.  Agrega un proveedor y un modelo de Bedrock a tu configuración (no se requiere `apiKey`):

```json
{
  models: {
    providers: {
      "amazon-bedrock": {
        baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
        api: "bedrock-converse-stream",
        auth: "aws-sdk",
        models: [
          {
            id: "us.anthropic.claude-opus-4-6-v1:0",
            name: "Claude Opus 4.6 (Bedrock)",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "amazon-bedrock/us.anthropic.claude-opus-4-6-v1:0" },
    },
  },
}
```

## Roles de instancia EC2

Cuando OpenClaw se ejecuta en una instancia EC2 con un rol de IAM adjunto, el SDK de AWS usará automáticamente el servicio de metadatos de instancia (IMDS) para la autenticación. Sin embargo, la detección de credenciales de OpenClaw actualmente solo verifica las variables de entorno, no las credenciales de IMDS. **Solución alternativa:** Establece `AWS_PROFILE=default` para indicar que hay credenciales de AWS disponibles. La autenticación real sigue usando el rol de instancia a través de IMDS.

```bash
# Agrega a ~/.bashrc o tu perfil de shell
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Permisos de IAM requeridos** para el rol de instancia EC2:

-   `bedrock:InvokeModel`
-   `bedrock:InvokeModelWithResponseStream`
-   `bedrock:ListFoundationModels` (para el descubrimiento automático)

O adjunta la política administrada `AmazonBedrockFullAccess`.

## Configuración rápida (ruta de AWS)

```bash
# 1. Crear rol de IAM y perfil de instancia
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. Adjuntar a tu instancia EC2
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. En la instancia EC2, habilita el descubrimiento
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. Establece las variables de entorno de la solución alternativa
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verifica que los modelos sean descubiertos
openclaw models list
```

## Notas

-   Bedrock requiere que el **acceso al modelo** esté habilitado en tu cuenta/región de AWS.
-   El descubrimiento automático necesita el permiso `bedrock:ListFoundationModels`.
-   Si usas perfiles, establece `AWS_PROFILE` en el host de la puerta de enlace.
-   OpenClaw muestra la fuente de la credencial en este orden: `AWS_BEARER_TOKEN_BEDROCK`, luego `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, luego `AWS_PROFILE`, luego la cadena predeterminada del SDK de AWS.
-   El soporte de razonamiento depende del modelo; consulta la ficha del modelo de Bedrock para conocer las capacidades actuales.
-   Si prefieres un flujo de clave administrada, también puedes colocar un proxy compatible con OpenAI delante de Bedrock y configurarlo como un proveedor de OpenAI.

[Anthropic](./anthropic.md)[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)