# Automatización IA para Experiencia de Clientes

Ecosistema autónomo de inteligencia artificial para procesar encuestas de satisfacción de clientes de servicio técnico automotriz, clasificar automáticamente cada experiencia y ejecutar acciones operacionales según su nivel de criticidad.

La solución integra Microsoft Forms, Excel/OneDrive, Make, Google Gemini, Notion, Gmail y Slack, incorporando automatización end-to-end, Human-in-the-Loop, trazabilidad, manejo de errores, reintentos y monitoreo operacional.

---

## Objetivo

Automatizar el procesamiento de encuestas de satisfacción posteriores a un servicio técnico, reduciendo la revisión manual y permitiendo que los casos críticos sean detectados y escalados rápidamente.

La automatización:

- recibe respuestas desde Microsoft Forms;
- procesa los datos mediante Make;
- analiza cada encuesta mediante Google Gemini;
- genera score, clasificación, sentimiento, categoría y criticidad;
- actualiza la información en Notion;
- ejecuta distintas acciones según la clasificación;
- requiere aprobación humana antes de contactar al cliente en casos críticos;
- registra éxitos, errores y recuperaciones.

---

## Arquitectura

![Arquitectura de la solución](arquitectura/arquitectura_solucion.png)

### Flujo general

Microsoft Forms → Excel / OneDrive → Make → Notion → Gemini → Router

El Router distribuye los casos en tres rutas:

**Ruta Verde**
- Score entre 80 y 100.
- Comunicación automática mediante Gmail.
- Caso cerrado o enviado a seguimiento según solicitud de contacto.

**Ruta Amarilla**
- Score entre 50 y 79.
- Se crea una acción de seguimiento.
- Se envía comunicación al cliente.
- El caso queda en seguimiento.

**Ruta Roja**
- Score inferior a 50 o `critical_flag = true`.
- Se crea un escalamiento crítico.
- Se envía alerta a Slack.
- Se requiere aprobación humana antes de contactar al cliente.

---

## Human-in-the-Loop

Los casos críticos utilizan Slack como mecanismo de aprobación humana.

Comandos disponibles:

```text
APROBAR ENC-ID
RECHAZAR ENC-ID
