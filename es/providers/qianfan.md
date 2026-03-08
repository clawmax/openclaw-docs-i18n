

  Proveedores

  
# Qianfan

Qianfan es la plataforma MaaS de Baidu, proporciona una **API unificada** que enruta las solicitudes a muchos modelos detrás de un único endpoint y clave API. Es compatible con OpenAI, por lo que la mayoría de los SDKs de OpenAI funcionan cambiando la URL base.

## Prerrequisitos

1.  Una cuenta de Baidu Cloud con acceso a la API de Qianfan
2.  Una clave API desde la consola de Qianfan
3.  OpenClaw instalado en tu sistema

## Obteniendo Tu Clave API

1.  Visita la [Consola de Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  Crea una nueva aplicación o selecciona una existente
3.  Genera una clave API (formato: `bce-v3/ALTAK-...`)
4.  Copia la clave API para usarla con OpenClaw

## Configuración de la CLI

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## Documentación Relacionada

-   [Configuración de OpenClaw](../gateway/configuration.md)
-   [Proveedores de Modelos](../concepts/model-providers.md)
-   [Configuración de Agentes](../concepts/agent.md)
-   [Documentación de la API de Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---