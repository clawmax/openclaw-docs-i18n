

  Seguridad

  
# MODELO DE AMENAZAS ATLAS

## Marco MITRE ATLAS

**Versión:** 1.0-borrador **Última actualización:** 2026-02-04 **Metodología:** MITRE ATLAS + Diagramas de Flujo de Datos **Marco:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Atribución del Marco

Este modelo de amenazas se basa en [MITRE ATLAS](https://atlas.mitre.org/), el marco estándar de la industria para documentar amenazas adversarias a sistemas de IA/ML. ATLAS es mantenido por [MITRE](https://www.mitre.org/) en colaboración con la comunidad de seguridad de IA. **Recursos clave de ATLAS:**

-   [Técnicas ATLAS](https://atlas.mitre.org/techniques/)
-   [Tácticas ATLAS](https://atlas.mitre.org/tactics/)
-   [Estudios de Caso ATLAS](https://atlas.mitre.org/studies/)
-   [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
-   [Contribuir a ATLAS](https://atlas.mitre.org/resources/contribute)

### Contribuir a Este Modelo de Amenazas

Este es un documento vivo mantenido por la comunidad de OpenClaw. Consulta [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) para las pautas de contribución:

-   Reportar nuevas amenazas
-   Actualizar amenazas existentes
-   Proponer cadenas de ataque
-   Sugerir mitigaciones

* * *

## 1. Introducción

### 1.1 Propósito

Este modelo de amenazas documenta las amenazas adversarias a la plataforma de agentes de IA OpenClaw y al mercado de habilidades ClawHub, utilizando el marco MITRE ATLAS diseñado específicamente para sistemas de IA/ML.

### 1.2 Alcance

| Componente | Incluido | Notas |
| --- | --- | --- |
| Entorno de Ejecución del Agente OpenClaw | Sí | Ejecución central del agente, llamadas a herramientas, sesiones |
| Gateway | Sí | Autenticación, enrutamiento, integración de canales |
| Integraciones de Canales | Sí | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| Mercado ClawHub | Sí | Publicación de habilidades, moderación, distribución |
| Servidores MCP | Sí | Proveedores de herramientas externas |
| Dispositivos de Usuario | Parcial | Aplicaciones móviles, clientes de escritorio |

### 1.3 Fuera del Alcance

Nada está explícitamente fuera del alcance de este modelo de amenazas.

* * *

## 2. Arquitectura del Sistema

### 2.1 Límites de Confianza

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZONA NO CONFIABLE                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LÍMITE DE CONFIANZA 1: Acceso al Canal           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Emparejamiento de dispositivo (período de gracia 30s)  │   │
│  │  • Validación AllowFrom / AllowList                       │   │
│  │  • Autenticación Token/Contraseña/Tailscale               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LÍMITE DE CONFIANZA 2: Aislamiento de Sesión     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESIONES DEL AGENTE                     │   │
│  │  • Clave de sesión = agente:canal:peer                    │   │
│  │  • Políticas de herramientas por agente                   │   │
│  │  • Registro de transcripciones                            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LÍMITE DE CONFIANZA 3: Ejecución de Herramientas │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  SANDBOX DE EJECUCIÓN                     │   │
│  │  • Sandbox Docker O Host (exec-approvals)                 │   │
│  │  • Ejecución remota Node                                  │   │
│  │  • Protección SSRF (DNS pinning + bloqueo de IP)          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LÍMITE DE CONFIANZA 4: Contenido Externo         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              URLs OBTENIDAS / CORREOS / WEBHOOKS          │   │
│  │  • Envoltura de contenido externo (etiquetas XML)         │   │
│  │  • Inyección de aviso de seguridad                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LÍMITE DE CONFIANZA 5: Cadena de Suministro      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publicación de habilidades (semver, SKILL.md requerido)│   │
│  │  • Banderas de moderación basadas en patrones             │   │
│  │  • Escaneo VirusTotal (próximamente)                      │   │
│  │  • Verificación de antigüedad de cuenta GitHub            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Flujos de Datos

| Flujo | Origen | Destino | Datos | Protección |
| --- | --- | --- | --- | --- |
| F1 | Canal | Gateway | Mensajes de usuario | TLS, AllowFrom |
| F2 | Gateway | Agente | Mensajes enrutados | Aislamiento de sesión |
| F3 | Agente | Herramientas | Invocaciones de herramientas | Aplicación de políticas |
| F4 | Agente | Externo | solicitudes web\_fetch | Bloqueo SSRF |
| F5 | ClawHub | Agente | Código de habilidad | Moderación, escaneo |
| F6 | Agente | Canal | Respuestas | Filtrado de salida |

* * *

## 3. Análisis de Amenazas por Táctica ATLAS

### 3.1 Reconocimiento (AML.TA0002)

#### T-RECON-001: Descubrimiento de Endpoints del Agente

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0006 - Escaneo Activo |
| **Descripción** | Atacante escanea endpoints expuestos del gateway OpenClaw |
| **Vector de Ataque** | Escaneo de red, consultas shodan, enumeración DNS |
| **Componentes Afectados** | Gateway, endpoints API expuestos |
| **Mitigaciones Actuales** | Opción de autenticación Tailscale, vincular a loopback por defecto |
| **Riesgo Residual** | Medio - Gateways públicos descubribles |
| **Recomendaciones** | Documentar despliegue seguro, agregar limitación de tasa en endpoints de descubrimiento |

#### T-RECON-002: Sondeo de Integración de Canales

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0006 - Escaneo Activo |
| **Descripción** | Atacante sondea canales de mensajería para identificar cuentas gestionadas por IA |
| **Vector de Ataque** | Envío de mensajes de prueba, observación de patrones de respuesta |
| **Componentes Afectados** | Todas las integraciones de canales |
| **Mitigaciones Actuales** | Ninguna específica |
| **Riesgo Residual** | Bajo - Valor limitado solo del descubrimiento |
| **Recomendaciones** | Considerar aleatorización del tiempo de respuesta |

* * *

### 3.2 Acceso Inicial (AML.TA0004)

#### T-ACCESS-001: Intercepción de Código de Emparejamiento

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Acceso a API de Inferencia de Modelo de IA |
| **Descripción** | Atacante intercepta el código de emparejamiento durante el período de gracia de 30s |
| **Vector de Ataque** | Mirar por encima del hombro, sniffing de red, ingeniería social |
| **Componentes Afectados** | Sistema de emparejamiento de dispositivos |
| **Mitigaciones Actuales** | Expiración 30s, códigos enviados por canal existente |
| **Riesgo Residual** | Medio - Período de gracia explotable |
| **Recomendaciones** | Reducir período de gracia, agregar paso de confirmación |

#### T-ACCESS-002: Suplantación de AllowFrom

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Acceso a API de Inferencia de Modelo de IA |
| **Descripción** | Atacante suplanta la identidad del remitente permitido en el canal |
| **Vector de Ataque** | Depende del canal - suplantación de número telefónico, suplantación de nombre de usuario |
| **Componentes Afectados** | Validación AllowFrom por canal |
| **Mitigaciones Actuales** | Verificación de identidad específica del canal |
| **Riesgo Residual** | Medio - Algunos canales vulnerables a suplantación |
| **Recomendaciones** | Documentar riesgos específicos del canal, agregar verificación criptográfica donde sea posible |

#### T-ACCESS-003: Robo de Token

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Acceso a API de Inferencia de Modelo de IA |
| **Descripción** | Atacante roba tokens de autenticación de archivos de configuración |
| **Vector de Ataque** | Malware, acceso no autorizado al dispositivo, exposición de copias de seguridad de configuración |
| **Componentes Afectados** | ~/.openclaw/credentials/, almacenamiento de configuración |
| **Mitigaciones Actuales** | Permisos de archivo |
| **Riesgo Residual** | Alto - Tokens almacenados en texto plano |
| **Recomendaciones** | Implementar cifrado de tokens en reposo, agregar rotación de tokens |

* * *

### 3.3 Ejecución (AML.TA0005)

#### T-EXEC-001: Inyección de Prompt Directa

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0051.000 - Inyección de Prompt LLM: Directa |
| **Descripción** | Atacante envía prompts manipulados para alterar el comportamiento del agente |
| **Vector de Ataque** | Mensajes de canal que contienen instrucciones adversarias |
| **Componentes Afectados** | LLM del agente, todas las superficies de entrada |
| **Mitigaciones Actuales** | Detección de patrones, envoltura de contenido externo |
| **Riesgo Residual** | Crítico - Solo detección, sin bloqueo; ataques sofisticados lo evaden |
| **Recomendaciones** | Implementar defensa multicapa, validación de salida, confirmación del usuario para acciones sensibles |

#### T-EXEC-002: Inyección de Prompt Indirecta

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0051.001 - Inyección de Prompt LLM: Indirecta |
| **Descripción** | Atacante incrusta instrucciones maliciosas en contenido obtenido |
| **Vector de Ataque** | URLs maliciosas, correos electrónicos envenenados, webhooks comprometidos |
| **Componentes Afectados** | web\_fetch, ingesta de correo, fuentes de datos externas |
| **Mitigaciones Actuales** | Envoltura de contenido con etiquetas XML y aviso de seguridad |
| **Riesgo Residual** | Alto - El LLM puede ignorar las instrucciones de envoltura |
| **Recomendaciones** | Implementar sanitización de contenido, contextos de ejecución separados |

#### T-EXEC-003: Inyección de Argumentos de Herramienta

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0051.000 - Inyección de Prompt LLM: Directa |
| **Descripción** | Atacante manipula argumentos de herramientas a través de inyección de prompt |
| **Vector de Ataque** | Prompts manipulados que influyen en los valores de los parámetros de la herramienta |
| **Componentes Afectados** | Todas las invocaciones de herramientas |
| **Mitigaciones Actuales** | Aprobaciones de ejecución para comandos peligrosos |
| **Riesgo Residual** | Alto - Depende del juicio del usuario |
| **Recomendaciones** | Implementar validación de argumentos, llamadas a herramientas parametrizadas |

#### T-EXEC-004: Omisión de Aprobación de Ejecución

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Crear Datos Adversarios |
| **Descripción** | Atacante crea comandos que omiten la lista de permitidos de aprobación |
| **Vector de Ataque** | Ofuscación de comandos, explotación de alias, manipulación de rutas |
| **Componentes Afectados** | exec-approvals.ts, lista de permitidos de comandos |
| **Mitigaciones Actuales** | Lista de permitidos + modo de consulta |
| **Riesgo Residual** | Alto - Sin sanitización de comandos |
| **Recomendaciones** | Implementar normalización de comandos, expandir lista de bloqueados |

* * *

### 3.4 Persistencia (AML.TA0006)

#### T-PERSIST-001: Instalación de Habilidad Maliciosa

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0010.001 - Compromiso de Cadena de Suministro: Software de IA |
| **Descripción** | Atacante publica una habilidad maliciosa en ClawHub |
| **Vector de Ataque** | Crear cuenta, publicar habilidad con código malicioso oculto |
| **Componentes Afectados** | ClawHub, carga de habilidades, ejecución del agente |
| **Mitigaciones Actuales** | Verificación de antigüedad de cuenta GitHub, banderas de moderación basadas en patrones |
| **Riesgo Residual** | Crítico - Sin sandboxing, revisión limitada |
| **Recomendaciones** | Integración VirusTotal (en progreso), sandboxing de habilidades, revisión comunitaria |

#### T-PERSIST-002: Envenenamiento de Actualización de Habilidad

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0010.001 - Compromiso de Cadena de Suministro: Software de IA |
| **Descripción** | Atacante compromete una habilidad popular y envía una actualización maliciosa |
| **Vector de Ataque** | Compromiso de cuenta, ingeniería social del propietario de la habilidad |
| **Componentes Afectados** | Control de versiones de ClawHub, flujos de actualización automática |
| **Mitigaciones Actuales** | Huella digital de versión |
| **Riesgo Residual** | Alto - Las actualizaciones automáticas pueden obtener versiones maliciosas |
| **Recomendaciones** | Implementar firma de actualizaciones, capacidad de reversión, fijación de versión |

#### T-PERSIST-003: Manipulación de Configuración del Agente

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0010.002 - Compromiso de Cadena de Suministro: Datos |
| **Descripción** | Atacante modifica la configuración del agente para mantener el acceso |
| **Vector de Ataque** | Modificación de archivo de configuración, inyección de configuraciones |
| **Componentes Afectados** | Configuración del agente, políticas de herramientas |
| **Mitigaciones Actuales** | Permisos de archivo |
| **Riesgo Residual** | Medio - Requiere acceso local |
| **Recomendaciones** | Verificación de integridad de configuración, registro de auditoría para cambios de configuración |

* * *

### 3.5 Evasión de Defensa (AML.TA0007)

#### T-EVADE-001: Omisión de Patrón de Moderación

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Crear Datos Adversarios |
| **Descripción** | Atacante crea contenido de habilidad para evadir patrones de moderación |
| **Vector de Ataque** | Homóglifos Unicode, trucos de codificación, carga dinámica |
| **Componentes Afectados** | ClawHub moderation.ts |
| **Mitigaciones Actuales** | FLAG\_RULES basadas en patrones |
| **Riesgo Residual** | Alto - Expresiones regulares simples fácilmente evitadas |
| **Recomendaciones** | Agregar análisis de comportamiento (VirusTotal Code Insight), detección basada en AST |

#### T-EVADE-002: Escape de Envoltura de Contenido

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Crear Datos Adversarios |
| **Descripción** | Atacante crea contenido que escapa del contexto de envoltura XML |
| **Vector de Ataque** | Manipulación de etiquetas, confusión de contexto, anulación de instrucciones |
| **Componentes Afectados** | Envoltura de contenido externo |
| **Mitigaciones Actuales** | Etiquetas XML + aviso de seguridad |
| **Riesgo Residual** | Medio - Se descubren escapes novedosos regularmente |
| **Recomendaciones** | Múltiples capas de envoltura, validación del lado de salida |

* * *

### 3.6 Descubrimiento (AML.TA0008)

#### T-DISC-001: Enumeración de Herramientas

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Acceso a API de Inferencia de Modelo de IA |
| **Descripción** | Atacante enumera las herramientas disponibles a través de prompts |
| **Vector de Ataque** | Consultas del estilo "¿Qué herramientas tienes?" |
| **Componentes Afectados** | Registro de herramientas del agente |
| **Mitigaciones Actuales** | Ninguna específica |
| **Riesgo Residual** | Bajo - Las herramientas generalmente están documentadas |
| **Recomendaciones** | Considerar controles de visibilidad de herramientas |

#### T-DISC-002: Extracción de Datos de Sesión

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Acceso a API de Inferencia de Modelo de IA |
| **Descripción** | Atacante extrae datos sensibles del contexto de la sesión |
| **Vector de Ataque** | Consultas "¿De qué hablamos?", sondeo de contexto |
| **Componentes Afectados** | Transcripciones de sesión, ventana de contexto |
| **Mitigaciones Actuales** | Aislamiento de sesión por remitente |
| **Riesgo Residual** | Medio - Datos dentro de la sesión accesibles |
| **Recomendaciones** | Implementar redacción de datos sensibles en el contexto |

* * *

### 3.7 Colección y Exfiltración (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Robo de Datos via web\_fetch

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Colección |
| **Descripción** | Atacante exfiltra datos instruyendo al agente para que los envíe a una URL externa |
| **Vector de Ataque** | Inyección de prompt que hace que el agente POSTee datos al servidor del atacante |
| **Componentes Afectados** | Herramienta web\_fetch |
| **Mitigaciones Actuales** | Bloqueo SSRF para redes internas |
| **Riesgo Residual** | Alto - URLs externas permitidas |
| **Recomendaciones** | Implementar listas de permitidos de URL, conciencia de clasificación de datos |

#### T-EXFIL-002: Envío No Autorizado de Mensajes

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Colección |
| **Descripción** | Atacante hace que el agente envíe mensajes que contienen datos sensibles |
| **Vector de Ataque** | Inyección de prompt que hace que el agente envíe mensajes al atacante |
| **Componentes Afectados** | Herramienta de mensajes, integraciones de canales |
| **Mitigaciones Actuales** | Control de mensajes salientes |
| **Riesgo Residual** | Medio - El control puede ser omitido |
| **Recomendaciones** | Requerir confirmación explícita para nuevos destinatarios |

#### T-EXFIL-003: Cosecha de Credenciales

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Colección |
| **Descripción** | Habilidad maliciosa cosecha credenciales del contexto del agente |
| **Vector de Ataque** | Código de habilidad lee variables de entorno, archivos de configuración |
| **Componentes Afectados** | Entorno de ejecución de habilidades |
| **Mitigaciones Actuales** | Ninguna específica para habilidades |
| **Riesgo Residual** | Crítico - Las habilidades se ejecutan con privilegios del agente |
| **Recomendaciones** | Sandboxing de habilidades, aislamiento de credenciales |

* * *

### 3.8 Impacto (AML.TA0011)

#### T-IMPACT-001: Ejecución No Autorizada de Comandos

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Erosionar la Integridad del Modelo de IA |
| **Descripción** | Atacante ejecuta comandos arbitrarios en el sistema del usuario |
| **Vector de Ataque** | Inyección de prompt combinada con omisión de aprobación de ejecución |
| **Componentes Afectados** | Herramienta Bash, ejecución de comandos |
| **Mitigaciones Actuales** | Aprobaciones de ejecución, opción de sandbox Docker |
| **Riesgo Residual** | Crítico - Ejecución en host sin sandbox |
| **Recomendaciones** | Predeterminar sandbox, mejorar UX de aprobación |

#### T-IMPACT-002: Agotamiento de Recursos (DoS)

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Erosionar la Integridad del Modelo de IA |
| **Descripción** | Atacante agota créditos de API o recursos de cómputo |
| **Vector de Ataque** | Inundación automatizada de mensajes, llamadas a herramientas costosas |
| **Componentes Afectados** | Gateway, sesiones del agente, proveedor de API |
| **Mitigaciones Actuales** | Ninguna |
| **Riesgo Residual** | Alto - Sin limitación de tasa |
| **Recomendaciones** | Implementar límites de tasa por remitente, presupuestos de costo |

#### T-IMPACT-003: Daño a la Reputación

| Atributo | Valor |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Erosionar la Integridad del Modelo de IA |
| **Descripción** | Atacante hace que el agente envíe contenido dañino/ofensivo |
| **Vector de Ataque** | Inyección de prompt que causa respuestas inapropiadas |
| **Componentes Afectados** | Generación de salida, mensajería de canal |
| **Mitigaciones Actuales** | Políticas de contenido del proveedor LLM |
| **Riesgo Residual** | Medio - Filtros del proveedor imperfectos |
| **Recomendaciones** | Capa de filtrado de salida, controles de usuario |

* * *

## 4. Análisis de la Cadena de Suministro de ClawHub

### 4.1 Controles de Seguridad Actuales

| Control | Implementación | Efectividad |
| --- | --- | --- |
| Antigüedad de Cuenta GitHub | `requireGitHubAccountAge()` | Medio - Eleva la barrera para nuevos atacantes |
| Sanitización de Ruta | `sanitizePath()` | Alto - Previene traversales de ruta |
| Validación de Tipo de Archivo | `isTextFile()` | Medio - Solo archivos de texto, pero aún pueden ser maliciosos |
| Límites de Tamaño | 50MB paquete total | Alto - Previene agotamiento de recursos |
| SKILL.md Requerido | Readme obligatorio | Bajo valor de seguridad - Solo informativo |
| Moderación por Patrones | FLAG\_RULES en moderation.ts | Bajo - Fácilmente evadido |
| Estado de Moderación | Campo `moderationStatus` | Medio - Revisión manual posible |

### 4.2 Patrones de Banderas de Moderación

Patrones actuales en `moderation.ts`:

```
// Identificadores conocidos como malos
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Palabras clave sospechosas
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+\|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**Limitaciones:**

-   Solo verifica slug, displayName, summary, frontmatter, metadata, rutas de archivo
-   No analiza el contenido real del código de la habilidad
-   Expresiones regulares simples fácilmente evadidas con ofuscación
-   Sin análisis de comportamiento

### 4.3 Mejoras Planeadas

| Mejora | Estado | Impacto |
| --- | --- | --- |
| Integración VirusTotal | En Progreso | Alto - Análisis de comportamiento Code Insight |
| Reporte Comunitario | Parcial (existe tabla `skillReports`) | Medio |
| Registro de Auditoría | Parcial (existe tabla `auditLogs`) | Medio |
| Sistema de Insignias | Implementado | Medio - `highlighted`, `official`, `deprecated`, `redactionApproved` |

* * *

## 5. Matriz de Riesgo

### 5.1 Probabilidad vs Impacto

| ID de Amenaza | Probabilidad | Impacto | Nivel de Riesgo | Prioridad |
| --- | --- | --- | --- | --- |
| T-EXEC-001 | Alto | Crítico | **Crítico** | P0 |
| T-PERSIST-001 | Alto | Crítico | **Crítico** | P0 |
| T-EXFIL-003 | Medio | Crítico | **Crítico** | P0 |
| T-IMPACT-001 | Medio | Crítico | **Alto** | P1 |
| T-EXEC-002 | Alto | Alto | **Alto** | P1 |
| T-EXEC-004 | Medio | Alto | **Alto** | P1 |
| T-ACCESS-003 | Medio | Alto | **Alto** | P1 |
| T-EXFIL-001 | Medio | Alto | **Alto** | P1 |
| T-IMPACT-002 | Alto | Medio | **Alto** | P1 |
| T-EVADE-001 | Alto | Medio | **Medio** | P2 |
| T-ACCESS-001 | Bajo | Alto | **Medio** | P2 |
| T-ACCESS-002 | Bajo | Alto | **Medio** | P2 |
| T-PERSIST-002 | Bajo | Alto | **Medio** | P2 |

### 5.2 Cadenas de Ataque de Ruta Crítica

**Cadena de Ataque 1: Robo de Datos Basado en Habilidad**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publicar habilidad maliciosa) → (Evadir moderación) → (Cosechar credenciales)
```

**Cadena de Ataque 2: Inyección de Prompt a RCE**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Injectar prompt) → (Omitir aprobación de ejecución) → (Ejecutar comandos)
```

**Cadena de Ataque 3: Inyección Indirecta via Contenido Obtenido**

```
T-EXEC-002 → T-EXFIL-001 → Exfiltración externa
(Envenenar contenido de URL) → (Agente obtiene y sigue instrucciones) → (Datos enviados al atacante)
```

* * *

## 6. Resumen de Recomendaciones

### 6.1 Inmediatas (P0)

| ID | Recomendación | Aborda |
| --- | --- | --- |
| R-001 | Completar integración VirusTotal | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implementar sandboxing de habilidades | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Agregar validación de salida para acciones sensibles | T-EXEC-001, T-EXEC-002 |

### 6.2 Corto Plazo (P1)

| ID | Recomendación | Aborda |
| --- | --- | --- |
| R-004 | Implementar limitación de tasa | T-IMPACT-002 |
| R-005 | Agregar cifrado de tokens en reposo | T-ACCESS-003 |
| R-006 | Mejorar UX y validación de aprobación de ejecución | T-EXEC-004 |
| R-007 | Implementar listas de permitidos de URL para web\_fetch | T-EXFIL-001 |

### 6.3 Mediano Plazo (P2)

| ID | Recomendación | Aborda |
| --- | --- | --- |
| R-008 | Agregar verificación criptográfica de canal donde sea posible | T-ACCESS-002 |
| R-009 | Implementar verificación de integridad de configuración | T-PERSIST-003 |
| R-010 | Agregar firma de actualizaciones y fijación de versión | T-PERSIST-002 |

* * *

## 7. Apéndices

### 7.1 Mapeo de Técnicas ATLAS

| ID ATLAS | Nombre de Técnica | Amenazas OpenClaw |
| --- | --- | --- |
| AML.T0006 | Escaneo Activo | T-RECON-001, T-RECON-002 |
| AML.T0009 | Colección | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003 |
| AML.T0010.001 | Cadena de Suministro: Software de IA | T-PERSIST-001, T-PERSIST-002 |
| AML.T0010.002 | Cadena de Suministro: Datos | T-PERSIST-003 |
| AML.T0031 | Erosionar la Integridad del Modelo de IA | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003 |
| AML.T0040 | Acceso a API de Inferencia de Modelo de IA | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043 | Crear Datos Adversarios | T-EXEC-004, T-EVADE-001, T-EVADE-002 |
| AML.T0051.000 | Inyección de Prompt LLM: Directa | T-EXEC-001, T-EXEC-003 |
| AML.T0051.001 | Inyección de Prompt LLM: Indirecta | T-EXEC-002 |

### 7.2 Archivos de Seguridad Clave

| Ruta | Propósito | Nivel de Riesgo |
| --- | ---