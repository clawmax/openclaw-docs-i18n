

  Configuración

  
# Grupos de Difusión

**Estado:** Experimental  
**Versión:** Añadido en 2026.1.9

## Descripción General

Los Grupos de Difusión permiten que múltiples agentes procesen y respondan al mismo mensaje simultáneamente. Esto te permite crear equipos de agentes especializados que trabajan juntos en un solo grupo o DM de WhatsApp — todo usando un solo número de teléfono. Alcance actual: **Solo WhatsApp** (canal web). Los grupos de difusión se evalúan después de las listas de permitidos del canal y las reglas de activación de grupo. En grupos de WhatsApp, esto significa que las difusiones ocurren cuando OpenClaw normalmente respondería (por ejemplo: al ser mencionado, dependiendo de tu configuración de grupo).

## Casos de Uso

### 1\. Equipos de Agentes Especializados

Despliega múltiples agentes con responsabilidades atómicas y enfocadas:

```
Group: "Development Team"
Agents:
  - CodeReviewer (revisa fragmentos de código)
  - DocumentationBot (genera documentación)
  - SecurityAuditor (busca vulnerabilidades)
  - TestGenerator (sugiere casos de prueba)
```

Cada agente procesa el mismo mensaje y proporciona su perspectiva especializada.

### 2\. Soporte Multi-Idioma

```
Group: "International Support"
Agents:
  - Agent_EN (responde en inglés)
  - Agent_DE (responde en alemán)
  - Agent_ES (responde en español)
```

### 3\. Flujos de Trabajo de Control de Calidad

```
Group: "Customer Support"
Agents:
  - SupportAgent (proporciona la respuesta)
  - QAAgent (revisa la calidad, solo responde si encuentra problemas)
```

### 4\. Automatización de Tareas

```
Group: "Project Management"
Agents:
  - TaskTracker (actualiza la base de datos de tareas)
  - TimeLogger (registra el tiempo empleado)
  - ReportGenerator (crea resúmenes)
```

## Configuración

### Configuración Básica

Añade una sección de nivel superior `broadcast` (junto a `bindings`). Las claves son IDs de pares de WhatsApp:

-   chats de grupo: JID del grupo (ej. `120363403215116621@g.us`)
-   DMs: número de teléfono E.164 (ej. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Resultado:** Cuando OpenClaw respondería en este chat, ejecutará los tres agentes.

### Estrategia de Procesamiento

Controla cómo los agentes procesan los mensajes:

#### Paralelo (Por Defecto)

Todos los agentes procesan simultáneamente:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Secuencial

Los agentes procesan en orden (uno espera a que el anterior termine):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Ejemplo Completo

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Cómo Funciona

### Flujo del Mensaje

1.  **Llega un mensaje entrante** a un grupo de WhatsApp
2.  **Verificación de difusión**: El sistema verifica si el ID del par está en `broadcast`
3.  **Si está en la lista de difusión**:
    -   Todos los agentes listados procesan el mensaje
    -   Cada agente tiene su propia clave de sesión y contexto aislado
    -   Los agentes procesan en paralelo (por defecto) o secuencialmente
4.  **Si NO está en la lista de difusión**:
    -   Se aplica el enrutamiento normal (primer binding coincidente)

Nota: los grupos de difusión no omiten las listas de permitidos del canal ni las reglas de activación de grupo (menciones/comandos/etc). Solo cambian *qué agentes se ejecutan* cuando un mensaje es elegible para procesamiento.

### Aislamiento de Sesión

Cada agente en un grupo de difusión mantiene completamente separados:

-   **Claves de sesión** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
-   **Historial de conversación** (el agente no ve los mensajes de otros agentes)
-   **Espacio de trabajo** (sandboxes separados si están configurados)
-   **Acceso a herramientas** (diferentes listas de permitidos/denegados)
-   **Memoria/contexto** (IDENTITY.md, SOUL.md, etc. separados)
-   **Búfer de contexto del grupo** (mensajes recientes del grupo usados para contexto) se comparte por par, por lo que todos los agentes de difusión ven el mismo contexto cuando se activan

Esto permite que cada agente tenga:

-   Personalidades diferentes
-   Acceso a herramientas diferente (ej., solo lectura vs. lectura-escritura)
-   Modelos diferentes (ej., opus vs. sonnet)
-   Habilidades instaladas diferentes

### Ejemplo: Sesiones Aisladas

En el grupo `120363403215116621@g.us` con agentes `["alfred", "baerbel"]`: **Contexto de Alfred:**

```yaml
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contexto de Bärbel:**

```yaml
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Mejores Prácticas

### 1\. Mantén a los Agentes Enfocados

Diseña cada agente con una sola responsabilidad clara:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Bien:** Cada agente tiene un trabajo  
❌ **Mal:** Un agente genérico "dev-helper"

### 2\. Usa Nombres Descriptivos

Deja claro lo que hace cada agente:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3\. Configura Diferente Acceso a Herramientas

Da a los agentes solo las herramientas que necesitan:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Solo lectura
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Lectura-escritura
    }
  }
}
```

### 4\. Monitorea el Rendimiento

Con muchos agentes, considera:

-   Usar `"strategy": "parallel"` (por defecto) para velocidad
-   Limitar los grupos de difusión a 5-10 agentes
-   Usar modelos más rápidos para agentes más simples

### 5\. Maneja Fallos con Elegancia

Los agentes fallan independientemente. El error de un agente no bloquea a los demás:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Compatibilidad

### Proveedores

Los grupos de difusión funcionan actualmente con:

-   ✅ WhatsApp (implementado)
-   🚧 Telegram (planeado)
-   🚧 Discord (planeado)
-   🚧 Slack (planeado)

### Enrutamiento

Los grupos de difusión funcionan junto con el enrutamiento existente:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

-   `GROUP_A`: Solo alfred responde (enrutamiento normal)
-   `GROUP_B`: agent1 Y agent2 responden (difusión)

**Precedencia:** `broadcast` tiene prioridad sobre `bindings`.

## Solución de Problemas

### Agentes que No Responden

**Verifica:**

1.  Los IDs de agente existen en `agents.list`
2.  El formato del ID del par es correcto (ej., `120363403215116621@g.us`)
3.  Los agentes no están en listas de denegados

**Depuración:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Solo un Agente Responde

**Causa:** El ID del par podría estar en `bindings` pero no en `broadcast`. **Solución:** Añádelo a la configuración de difusión o elimínalo de los bindings.

### Problemas de Rendimiento

**Si es lento con muchos agentes:**

-   Reduce el número de agentes por grupo
-   Usa modelos más ligeros (sonnet en lugar de opus)
-   Verifica el tiempo de inicio del sandbox

## Ejemplos

### Ejemplo 1: Equipo de Revisión de Código

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**El usuario envía:** Fragmento de código  
**Respuestas:**

-   code-formatter: “Indentación corregida y sugerencias de tipo añadidas”
-   security-scanner: “⚠️ Vulnerabilidad de inyección SQL en la línea 12”
-   test-coverage: “La cobertura es del 45%, faltan pruebas para casos de error”
-   docs-checker: “Falta la cadena de documentación para la función `process_data`”

### Ejemplo 2: Soporte Multi-Idioma

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Referencia de la API

### Esquema de Configuración

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Campos

-   `strategy` (opcional): Cómo procesar los agentes
    -   `"parallel"` (por defecto): Todos los agentes procesan simultáneamente
    -   `"sequential"`: Los agentes procesan en el orden del array
-   `[peerId]`: JID de grupo de WhatsApp, número E.164, u otro ID de par
    -   Valor: Array de IDs de agentes que deben procesar mensajes

## Limitaciones

1.  **Máximo de agentes:** No hay un límite estricto, pero 10+ agentes pueden ser lentos
2.  **Contexto compartido:** Los agentes no ven las respuestas de los otros (por diseño)
3.  **Orden de mensajes:** Las respuestas paralelas pueden llegar en cualquier orden
4.  **Límites de tasa:** Todos los agentes cuentan para los límites de tasa de WhatsApp

## Mejoras Futuras

Características planeadas:

-   [ ]  Modo de contexto compartido (los agentes ven las respuestas de los otros)
-   [ ]  Coordinación de agentes (los agentes pueden señalarse entre sí)
-   [ ]  Selección dinámica de agentes (elegir agentes basándose en el contenido del mensaje)
-   [ ]  Prioridades de agentes (algunos agentes responden antes que otros)

## Ver También

-   [Configuración Multi-Agente](../tools/multi-agent-sandbox-tools.md)
-   [Configuración de Enrutamiento](./channel-routing.md)
-   [Gestión de Sesiones](../concepts/session.md)

[Grupos](./groups.md)[Enrutamiento de Canales](./channel-routing.md)

---