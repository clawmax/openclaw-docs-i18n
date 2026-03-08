

  Seguridad

  
# CONTRIBUIR AL MODELO DE AMENAZAS

Gracias por ayudar a hacer OpenClaw más seguro. Este modelo de amenazas es un documento vivo y damos la bienvenida a contribuciones de cualquier persona; no necesitas ser un experto en seguridad.

## Formas de Contribuir

### Añadir una Amenaza

¿Has detectado un vector de ataque o riesgo que no hemos cubierto? Abre un issue en [openclaw/trust](https://github.com/openclaw/trust/issues) y descríbelo con tus propias palabras. No necesitas conocer ningún framework ni rellenar todos los campos; solo describe el escenario. **Es útil incluir (pero no obligatorio):**

-   El escenario de ataque y cómo podría explotarse
-   Qué partes de OpenClaw se ven afectadas (CLI, gateway, canales, ClawHub, servidores MCP, etc.)
-   Qué tan grave crees que es (baja / media / alta / crítica)
-   Cualquier enlace a investigaciones relacionadas, CVEs o ejemplos del mundo real

Nosotros nos encargaremos del mapeo ATLAS, los IDs de amenaza y la evaluación de riesgo durante la revisión. Si quieres incluir esos detalles, genial, pero no es obligatorio.

> **Esto es para añadir al modelo de amenazas, no para reportar vulnerabilidades activas.** Si has encontrado una vulnerabilidad explotable, consulta nuestra [página de Confianza](https://trust.openclaw.ai) para las instrucciones de divulgación responsable.

### Sugerir una Mitigación

¿Tienes una idea de cómo abordar una amenaza existente? Abre un issue o un PR haciendo referencia a la amenaza. Las mitigaciones útiles son específicas y accionables; por ejemplo, "limitación de tasa por remitente de 10 mensajes/minuto en el gateway" es mejor que "implementar limitación de tasa".

### Proponer una Cadena de Ataque

Las cadenas de ataque muestran cómo múltiples amenazas se combinan en un escenario de ataque realista. Si ves una combinación peligrosa, describe los pasos y cómo un atacante los encadenaría. Una narrativa breve de cómo se desarrolla el ataque en la práctica es más valiosa que una plantilla formal.

### Corregir o Mejorar Contenido Existente

Errores tipográficos, aclaraciones, información desactualizada, mejores ejemplos: los PR son bienvenidos, no se necesita un issue.

## Lo que Utilizamos

### MITRE ATLAS

Este modelo de amenazas se basa en [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), un framework diseñado específicamente para amenazas de IA/ML como inyección de prompts, mal uso de herramientas y explotación de agentes. No necesitas conocer ATLAS para contribuir; mapeamos las propuestas al framework durante la revisión.

### IDs de Amenaza

Cada amenaza recibe un ID como `T-EXEC-003`. Las categorías son:

| Código | Categoría |
| --- | --- |
| RECON | Reconocimiento - recopilación de información |
| ACCESS | Acceso inicial - obtener entrada |
| EXEC | Ejecución - ejecutar acciones maliciosas |
| PERSIST | Persistencia - mantener el acceso |
| EVADE | Evasión de defensas - evitar la detección |
| DISC | Descubrimiento - aprender sobre el entorno |
| EXFIL | Exfiltración - robar datos |
| IMPACT | Impacto - daño o interrupción |

Los IDs son asignados por los mantenedores durante la revisión. No necesitas elegir uno.

### Niveles de Riesgo

| Nivel | Significado |
| --- | --- |
| **Crítico** | Compromiso total del sistema, o alta probabilidad + impacto crítico |
| **Alto** | Daño significativo probable, o probabilidad media + impacto crítico |
| **Medio** | Riesgo moderado, o baja probabilidad + impacto alto |
| **Bajo** | Poco probable y con impacto limitado |

Si no estás seguro del nivel de riesgo, solo describe el impacto y nosotros lo evaluaremos.

## Proceso de Revisión

1.  **Clasificación** - Revisamos las nuevas propuestas en un plazo de 48 horas
2.  **Evaluación** - Verificamos la viabilidad, asignamos el mapeo ATLAS y el ID de amenaza, validamos el nivel de riesgo
3.  **Documentación** - Nos aseguramos de que todo esté formateado y completo
4.  **Fusión** - Se añade al modelo de amenazas y a la visualización

## Recursos

-   [Sitio web de ATLAS](https://atlas.mitre.org/)
-   [Técnicas de ATLAS](https://atlas.mitre.org/techniques/)
-   [Estudios de Caso de ATLAS](https://atlas.mitre.org/studies/)
-   [Modelo de Amenazas de OpenClaw](./THREAT-MODEL-ATLAS.md)

## Contacto

-   **Vulnerabilidades de seguridad:** Consulta nuestra [página de Confianza](https://trust.openclaw.ai) para instrucciones de reporte
-   **Preguntas sobre el modelo de amenazas:** Abre un issue en [openclaw/trust](https://github.com/openclaw/trust/issues)
-   **Chat general:** Canal #security en Discord

## Reconocimiento

Los contribuyentes al modelo de amenazas son reconocidos en los agradecimientos del modelo, las notas de la versión y el salón de la fama de seguridad de OpenClaw por contribuciones significativas.

[MODELO DE AMENAZAS ATLAS](./THREAT-MODEL-ATLAS.md)[Web](../web.md)