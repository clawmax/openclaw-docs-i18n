

  Conceptos de modelos

  
# CLI de Modelos

Consulta [/concepts/model-failover](./model-failover.md) para la rotación de perfiles de autenticación, períodos de enfriamiento y cómo interactúa con los respaldos. Descripción general rápida de proveedores + ejemplos: [/concepts/model-providers](./model-providers.md).

## Cómo funciona la selección de modelos

OpenClaw selecciona modelos en este orden:

1.  Modelo **Primario** (`agents.defaults.model.primary` o `agents.defaults.model`).
2.  **Respaldos** en `agents.defaults.model.fallbacks` (en orden).
3.  La **conmutación por error de autenticación del proveedor** ocurre dentro de un proveedor antes de pasar al siguiente modelo.

Relacionado:

-   `agents.defaults.models` es la lista de permitidos/catálogo de modelos que OpenClaw puede usar (más alias).
-   `agents.defaults.imageModel` se usa **solo cuando** el modelo primario no puede aceptar imágenes.
-   Los valores predeterminados por agente pueden anular `agents.defaults.model` mediante `agents.list[].model` más enlaces (ver [/concepts/multi-agent](./multi-agent.md)).

## Política rápida de modelos

-   Establece tu modelo primario como el modelo de última generación más potente disponible para ti.
-   Usa respaldos para tareas sensibles al costo/la latencia y chats de menor importancia.
-   Para agentes con herramientas o entradas no confiables, evita niveles de modelos más antiguos/débiles.

## Asistente de configuración (recomendado)

Si no quieres editar la configuración manualmente, ejecuta el asistente de incorporación:

```bash
openclaw onboard
```

Puede configurar modelo + autenticación para proveedores comunes, incluida la **suscripción a OpenAI Code (Codex)** (OAuth) y **Anthropic** (clave API o `claude setup-token`).

## Claves de configuración (descripción general)

-   `agents.defaults.model.primary` y `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel.primary` y `agents.defaults.imageModel.fallbacks`
-   `agents.defaults.models` (lista de permitidos + alias + parámetros del proveedor)
-   `models.providers` (proveedores personalizados escritos en `models.json`)

Las referencias de modelo se normalizan a minúsculas. Los alias de proveedor como `z.ai/*` se normalizan a `zai/*`. Los ejemplos de configuración de proveedores (incluido OpenCode Zen) se encuentran en [/gateway/configuration](../gateway/configuration.md#opencode-zen-multi-model-proxy).

## “Modelo no permitido” (y por qué se detienen las respuestas)

Si `agents.defaults.models` está configurado, se convierte en la **lista de permitidos** para `/model` y para las anulaciones de sesión. Cuando un usuario selecciona un modelo que no está en esa lista, OpenClaw devuelve:

```bash
Model "provider/model" is not allowed. Use /model to list available models.
```

Esto sucede **antes** de que se genere una respuesta normal, por lo que el mensaje puede parecer que "no respondió". La solución es:

-   Agregar el modelo a `agents.defaults.models`, o
-   Limpiar la lista de permitidos (eliminar `agents.defaults.models`), o
-   Elegir un modelo de `/model list`.

Ejemplo de configuración de lista de permitidos:

```json
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## Cambiar modelos en el chat (/model)

Puedes cambiar modelos para la sesión actual sin reiniciar:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notas:

-   `/model` (y `/model list`) es un selector compacto y numerado (familia de modelos + proveedores disponibles).
-   En Discord, `/model` y `/models` abren un selector interactivo con menús desplegables de proveedor y modelo más un paso de Enviar.
-   `/model <#>` selecciona de ese selector.
-   `/model status` es la vista detallada (candidatos de autenticación y, cuando está configurado, `baseUrl` del endpoint del proveedor + modo `api`).
-   Las referencias de modelo se analizan dividiendo en la **primera** `/`. Usa `provider/model` al escribir `/model `.
-   Si el ID del modelo contiene `/` (estilo OpenRouter), debes incluir el prefijo del proveedor (ejemplo: `/model openrouter/moonshotai/kimi-k2`).
-   Si omites el proveedor, OpenClaw trata la entrada como un alias o un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).

Comportamiento/configuración completa del comando: [Comandos de barra](../tools/slash-commands.md).

## Comandos CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (sin subcomando) es un acceso directo para `models status`.

### models list

Muestra los modelos configurados por defecto. Banderas útiles:

-   `--all`: catálogo completo
-   `--local`: solo proveedores locales
-   `--provider `: filtrar por proveedor
-   `--plain`: un modelo por línea
-   `--json`: salida legible por máquina

### models status

Muestra el modelo primario resuelto, respaldos, modelo de imagen y una descripción general de autenticación de los proveedores configurados. También muestra el estado de caducidad de OAuth para los perfiles encontrados en el almacén de autenticación (advierte dentro de las 24h por defecto). `--plain` imprime solo el modelo primario resuelto. El estado de OAuth siempre se muestra (y se incluye en la salida `--json`). Si un proveedor configurado no tiene credenciales, `models status` imprime una sección **Falta autenticación**. JSON incluye `auth.oauth` (ventana de advertencia + perfiles) y `auth.providers` (autenticación efectiva por proveedor). Usa `--check` para automatización (salida `1` cuando falta/caduca, `2` cuando está por caducar). La elección de autenticación depende del proveedor/cuenta. Para hosts de puerta de enlace siempre activos, las claves API suelen ser las más predecibles; también se admiten flujos de token de suscripción. Ejemplo (Anthropic setup-token):

```bash
claude setup-token
openclaw models status
```

## Escaneo (modelos gratuitos de OpenRouter)

`openclaw models scan` inspecciona el **catálogo de modelos gratuitos** de OpenRouter y puede opcionalmente sondear modelos para soporte de herramientas e imágenes. Banderas clave:

-   `--no-probe`: omitir sondeos en vivo (solo metadatos)
-   `--min-params `: tamaño mínimo de parámetros (miles de millones)
-   `--max-age-days `: omitir modelos más antiguos
-   `--provider `: filtro de prefijo de proveedor
-   `--max-candidates `: tamaño de la lista de respaldos
-   `--set-default`: establecer `agents.defaults.model.primary` a la primera selección
-   `--set-image`: establecer `agents.defaults.imageModel.primary` a la primera selección de imagen

El sondeo requiere una clave API de OpenRouter (de perfiles de autenticación o `OPENROUTER_API_KEY`). Sin una clave, usa `--no-probe` para listar solo candidatos. Los resultados del escaneo se clasifican por:

1.  Soporte de imagen
2.  Latencia de herramientas
3.  Tamaño de contexto
4.  Cantidad de parámetros

Entrada

-   Lista `/models` de OpenRouter (filtro `:free`)
-   Requiere clave API de OpenRouter de perfiles de autenticación o `OPENROUTER_API_KEY` (ver [/environment](../help/environment.md))
-   Filtros opcionales: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
-   Controles de sondeo: `--timeout`, `--concurrency`

Cuando se ejecuta en una TTY, puedes seleccionar respaldos interactivamente. En modo no interactivo, pasa `--yes` para aceptar los valores predeterminados.

## Registro de modelos (models.json)

Los proveedores personalizados en `models.providers` se escriben en `models.json` bajo el directorio del agente (predeterminado `~/.openclaw/agents//models.json`). Este archivo se fusiona por defecto a menos que `models.mode` esté configurado en `replace`. Precedencia del modo de fusión para IDs de proveedor coincidentes:

-   `baseUrl` no vacío ya presente en el `models.json` del agente gana.
-   `apiKey` no vacío en el `models.json` del agente gana solo cuando ese proveedor no está gestionado por SecretRef en el contexto de configuración/perfil de autenticación actual.
-   Los valores `apiKey` de proveedores gestionados por SecretRef se actualizan desde marcadores de origen (`ENV_VAR_NAME` para referencias de entorno, `secretref-managed` para referencias de archivo/ejecución) en lugar de persistir secretos resueltos.
-   `apiKey`/`baseUrl` vacíos o faltantes en el agente recurren a `models.providers` de la configuración.
-   Otros campos del proveedor se actualizan desde la configuración y datos normalizados del catálogo.

Esta persistencia basada en marcadores se aplica siempre que OpenClaw regenera `models.json`, incluidas rutas impulsadas por comandos como `openclaw agent`.

[Inicio rápido de proveedores de modelos](../providers/models.md)[Proveedores de modelos](./model-providers.md)